# توثيق شامل لصفحة هوتسبوت شبكة تونس نت اللاسلكية (MikroTik Hotspot Portal)

هذا التوثيق الشامل يوضح كافة بيانات راوتر مايكروتك (MikroTik RouterOS)، والملفات القالبية، والأكواد المصدرية، ومعاملات الربط، والعمليات التفاعلية (تسجيل الدخول، تغيير السرعات، إيقاف التحديثات، مراقبة الاستهلاك، قطع الاتصال، والحظر) وآلية عمل كل دالة وسكربت بالتفصيل لسهولة الرجوع إليها وإدارتها.

---

## 1. الهيكلية العامة وملفات نظام مايكروتك (Template Files)

عند رفع مجلد الهوتسبوت إلى راوتر مايكروتك (في مسار `hotspot` أو `Files`), يعتمد نظام RouterOS على الملفات التالية لمعالجة الجلسات والبيانات:

| اسم الملف | الوظيفة في مايكروتك | نوع المحتوى |
| :--- | :--- | :--- |
| **`index.html`** | الصفحة الرئيسية للعميل (بوابة تسجيل الدخول، البنر، باقات الأسعار، نقاط البيع، وشاشة الاستهلاك). | واجهة مستخدم HTML5 + CSS3 + JS |
| **`login.html`** | ملف وسيط يستجيب لطلبات مايكروتك عند بدء الجلسة أو حدوث خطأ. | JSON / قالب مايكروتك |
| **`alogin.html`** | ملف وسيط يستجيب فور نجاح تسجيل دخول الكرت بنجاح (Post-Auth). | JSON / قالب مايكروتك |
| **`status.html`** | ملف وسيط يستجيب لاستعلامات المايكروتك الدورية لجلب الرصيد والوقت والسرعة. | JSON / قالب مايكروتك |
| **`logout.html`** | ملف وسيط يستجيب لطلب تسجيل الخروج وإنهاء الجلسة ومسح الكوكيز. | JSON / قالب مايكروتك |
| **`redirect.html`** | إعادة توجيه المتصفح تلقائياً عند طلب رابط محجوب قبل تسجيل الدخول. | HTML Refresh |
| **`mobasher.html`** | صفحة تشغيل البث المباشر وقنوات المباريات بدون استهلاك الرصيد. | HTML Sandbox Player |
| **`config/config.js`** | ملف الإعدادات المخصص (اسم الشبكة، باقات الأسعار، نقاط البيع، شروط الحظر). | إعدادات JavaScript Config |

---

## 2. الأكواد المصدرية لملفات مايكروتك الخاصة بالراوتر

### 1) ملف تسجيل الدخول والبدء: `login.html`
يقوم المايكروتك بإرجاع كائن JSON يحتوي على معلومات الجلسة أو تفاصيل الخطأ إذا كان الكرت غير صالح:
```html
$(if var == "")<script>window.location.href="index.html"</script>$(else) $(if error == "") { "logged_in": "$(logged-in)", "link_login_only": "$(link-login-only)", "link_logout": "$(link-logout)", "link_status": "$(link-status)", "nas_id": "$(identity)", "ip": "$(ip)", "mac": "$(mac-esc)", "trial": "$(trial)", "username": "$(username2)", "action": "onLoginStart" } $(else) { "logged_in" : "$(logged-in)", "link_login_only" : "$(link-login-only)", "error_orig" : "$(error-orig)", "error" : "$(error)", "username" : "", "mac": "$(mac-esc)", "trial": "$(trial)", "action" : "onLoginError" } $(endif) $(endif)
```
- **المتغيرات المستخرجة من مايكروتك:**
  - `$(logged-in)`: حالة الاتصال (`yes` أو `no`).
  - `$(link-login-only)`: رابط إرسال طلب تسجيل الدخول للراوتر.
  - `$(link-logout)`: رابط تسجيل الخروج.
  - `$(link-status)`: رابط جلب بيانات الاستهلاك.
  - `$(identity)`: اسم راوتر المايكروتك (Identity).
  - `$(ip)` & `$(mac-esc)`: عنوان IP والماك أدريس لجهاز العميل.
  - `$(error)`: نص الخطأ الصادر من مايكروتك (مثل: invalid username or password).

---

### 2) ملف استجابة نجاح تسجيل الدخول: `alogin.html`
يستجيب المايكروتك بهذا الملف فور قبول الكرت، لتحديث الواجهة بالسرعة والوقت المتاح:
```html
$(if var == "")<script>window.location.href="index.html"</script>$(else) { "logged_in" : "$(logged-in)", "username": "null", "mac": "$(mac-esc)", "link_login_only" : "$(link-login-only)","sspeed" : "$(domain)_","update" : "$(domain)_" , "ip": "$(ip)", "bytes_in": "$(bytes-in)", "bytes_out": "$(bytes-out)", "remain_bytes_total": "$(remain-bytes-total)" , "sspeed" : "$(domain)_" , "session_time_left":	"$(session-time-left)", "uptime": "$(uptime)", "session_time_left_secs":	"$(session-time-left-secs)", "uptime_secs": "$(uptime-secs)", "trial": "$(trial)", "login_by": "$(login-by)", "action" : "onLoggedIn" } $(endif)
```

---

### 3) ملف استعلام بيانات الكرت والاستهلاك: `status.html`
يتم استدعاؤه دورياً كل ثانية أو ثانيتين لتحديث عدادات التنزيل والرفع والوقت المتبقي:
```html
$(if var == "")<script>window.location.href="index.html"</script>
$(else) 
{ "logged_in": "$(logged-in)", "mac": "$(mac-esc)","sspeed" : "$(domain)_" ,"update" : "$(domain)_", "ip": "$(ip)", "bytes_in": "$(bytes-in)", "bytes_out": "$(bytes-out)", "remain_bytes_total": "$(remain-bytes-total)", "session_time_left":	"$(session-time-left)", "uptime": "$(uptime)", "bytesm": "m056fd9fdfdsffsdffdfd1697455$(username)dsfd6571fgfgfgfgdf53sdfdsfgsd14", "trial": "$(trial)", "action": "onStatusQuery" } $(endif)
```
- **المتغيرات:**
  - `$(bytes-in)`: إجمالي البايتات المرفوعة (Upload).
  - `$(bytes-out)`: إجمالي البايتات المحملة (Download).
  - `$(remain-bytes-total)`: الرصيد المتبقي للباقة بالبايت.
  - `$(uptime)`: المدة الزمنية المستهلكة في الجلسة.
  - `$(session-time-left)`: الوقت المتبقي للكرت.
  - `$(domain)_`: القيمة الحالية للسرعة وإيقاف التحديثات المعينة من قبل العميل.

---

### 4) ملف تسجيل الخروج: `logout.html`
يستجيب به المايكروتك عند النقر على "تسجيل الخروج" أو "قطع الاتصال":
```html
$(if var == "")<script>window.location.href="index.html"</script>$(else) { "logged_in": "$(logged-in)", "link_login_only": "$(link-login-only)", "link_logout": "$(link-logout)","sspeed" : "$(domain)_" ,"update" : "$(domain)_" , "username": "null", "mac": "$(mac-esc)", "ip": "$(ip)", "bytes_in": "$(bytes-in)", "bytes_in_nice": "$(bytes-in-nice)", "bytes_out": "$(bytes-out)", "bytes_out_nice": "$(bytes-out-nice)", "remain_bytes_in": "$(remain-bytes-in)", "remain_bytes_out": "$(remain-bytes-out)", "packets_out": "$(packets-out)", "packets_in": "$(packets-in)", "session_time_left":	"$(session-time-left)", "uptime": "$(uptime)", "trial": "$(trial)", "action": "onLoggedOut" } $(endif)
```

---

## 3. ملف الإعدادات الكامل: `config/config.js`

هذا الملف يحتوي على كائن `Config({...})` الذي يتحكم في كافة تفاصيل الشبكة وباقاتها:
```javascript
Config({
  "network-name": "  تونس نت  ",
  "service-number": "772131004",
  "popup-adv-time": 5,
  "popup-adv-img-type": true,
  "popup-adv-repeat-after": 120,
  "login-type": "User",
  "login-chap": 0,
  "news-line": "        {بسم الله الرحمن الرحيم}( قل للمؤمنين يغضوا من أبصرهم ويحفظوا فروجهم ذلك ازكى لهم إن الله خبير بما يصنعون )صدق الله العضيم   ",
  "input-type": "tel",
  "input-autocomplete": "on",
  "speed-trychange": 3,           // أقصى عدد لمرات تغيير السرعة المسموحة
  "speed-trychange-timeout": 3,   // مدة حظر التغيير بالدقائق عند تجاوز المحاولات
  "input-rm-white-spaces": 0,
  "input-to-lower": 0,
  "input-to-upper": 0,
  "input-to-arabic-numbers": 1,
  "input-only-numbers": 0,
  "input-no-numbers": 0,
  "input-only-alphanumeric": 0,
  "input-to-tel-type-when": 0,
  "enable-hot-cookie": 1,         // تفعيل الكوكيز لحفظ كرت المستخدم
  "enable-hot-blocker": 1,        // تفعيل نظام حظر التخمين والمحاولات الخاطئة
  "clear-router-cookie": 1,
  "clear-hot-cookie": 1,
  "block-time": 1,                // مدة الحظر بالدقائق عند إدخال كروت خاطئة متكررة
  "try-count": 20,                // عدد المحاولات الخاطئة المسموح بها قبل الحظر
  "warn-when": 10,                // بدء التحذير عند الوصول لهذا العدد من الأخطاء
  "warn-message": "تحذير !! عدد محاولاتك الخاطئة اصبح {{tryCounter}} محاولات, عدد المحاولات المسموح بها هي {{tryCount}} محاولات فقط, عدد محاولاتك المتبقية {{restTryCount}} محاولات, سيتم حظرك لمدة {{blockTime}} دقائق اذا تجاوزت العدد المسموح للمحاولات",
  "price-button": true,
  "sell-point-button": true,
  "app-store-status-button": false,
  "show-date-field": false,
  "loan-button": true,
  "login-speeds-mode": true,      // تفعيل ميزة اختيار سرعة النت قبل الدخول
  "redirect-to-esterahah": "",
  "redirect-to-mobasher": "",
  "app-store-base-url": "",
  "app-store-base-url-ext": "",
  profiles: [
    { price: "100 ريال", time: "0 ساعات" },
    { price: "150 ريال", time: "0 ساعات" },
    { price: "200 ريال", time: "0 ساعات" },
    { price: "250 ريال", time: "0 ساعات" },
    { price: "300 ريال", time: "0 ساعات" },
    { price: "500 ريال", time: "0 ساعات" },
    { price: "150 ريال", time: "مفتوح " },
  ],
  "sell-points": [
    { name: "البقالات الموجودة في اماكن التغطية   " },
    { name: "  772131004 " },
  ],
});
```

---

## 4. العمليات التفاعلية مع راوتر مايكروتك (MikroTik Operations)

### 1) عملية تسجيل الدخول (Card Authentication)
- **عنصر النموذج**:
  ```html
  <form class="login-form" name="login">
      <input name="username" class="login-input" placeholder="ادخل رقم الكرت هنا" username-field improve-input autocomplete="off">
      <input type="hidden" name="password" class="login-input" placeholder="ادخل كلمة السر هنا" improve-input>
      <select id="speed" name="domain" speed-field style="display: none;">
          <option value="128k/512k">سرعة أقتصادية</option>
          <option selected value="256k/700k">سرعة عادية</option>
          <option value="256k/1M">سرعة متوسطة</option>
          <option value="512M/3M">سرعة عالية</option>
      </select>
      <input type="checkbox" value="_Uon" id="chupdate">
      <button class="button login-submit" parent-id="status" enable-hot-cookie>
          <span class="button-text">تسجيل الدخول</span>
      </button>
  </form>
  ```
- **آلية العمل**:
  1. يقوم المستخدم بإدخال الكرت واختيار السرعة وإيقاف التحديثات.
  2. يتم دمج السرعة وخيار التحديثات في حقل `domain` (مثلاً: `256k/700k_Uoff`).
  3. يتم إرسال طلب `POST` أو `GET` عبر دالة `hotSubmit` لعنوان `/login`.
  4. يتولى راوتر مايكروتك مطابقة الكرت في قاعدة بيانات اليوزرمان أو الهوتسبوت (`/ip hotspot user`).

---

### 2) عملية تبديل وتغيير السرعة أثناء الاتصال (Live Speed Switching)

#### جدول السرعات وقيمها في المايكروتك (Rate-Limit Table):
| مسمى السرعة في الصفحة | كود القيمة في HTML/JS (`value`) | سرعة الرفع (Upload) | سرعة التنزيل (Download) | صيغة المايكروتك (Rate Limit / Max Limit) |
| :--- | :--- | :--- | :--- | :--- |
| **سرعة اقتصادية** | `128k/512k` | 128 كيلوبت/ثانية | 512 كيلوبت/ثانية | `128k/512k` |
| **سرعة عادية (افتراضية)** | `256k/700k` | 256 كيلوبت/ثانية | 700 كيلوبت/ثانية | `256k/700k` |
| **سرعة متوسطة** | `256k/1M` | 256 كيلوبت/ثانية | 1 ميجابت/ثانية | `256k/1024k` |
| **سرعة عالية** | `512M/3M` (أو `512k/3M`) | 512 كيلوبت/ثانية | 3 ميجابت/ثانية | `512k/3072k` |
| **إلغاء التغيير** | `cancel` | - | - | الإبقاء على السرعة الحالية دون تعديل |

---

- **الكود البرمجي في صفحة تسجيل الدخول (`index.html`)**:
  ```html
  <!-- قائمة اختيار السرعة قبل الدخول -->
  <select id="speed" name="domain" speed-field style="display: none;">
      <option value="128k/512k">سرعة أقتصادية (128k/512k)</option>
      <option selected value="256k/700k">سرعة عادية (256k/700k)</option>
      <option value="256k/1M">سرعة متوسطة (256k/1M)</option>
      <option value="512M/3M">سرعة عالية (512k/3M)</option>
  </select>
  ```
- **الكود البرمجي في شاشة الحالة (`#status`) لتغيير السرعة أثناء الاتصال**:
  ```html
  <!-- عرض السرعة الحالية القادمة من المايكروتك -->
  <span id="sspeed"></span>

  <!-- قائمة اختيار سرعة جديدة أثناء الاتصال الفعال -->
  <select id="speedchange" class="tcs">
      <option value="128k/512k">سرعة أقتصادية</option>
      <option value="256k/700k">سرعة عادية</option>
      <option value="256k/1M">سرعة متوسطة</option>
      <option value="512M/3M">سرعة عالية</option>
      <option value="cancel" selected>إلغاء التغيير</option>
  </select>
  ```
- **كيف يفهم راوتر مايكروتك (MikroTik RouterOS) أمر تغيير السرعة؟**
  1. يقوم المتصفح بإرسال قيمة السرعة كمتغير نطاق `domain` ضمن طلب الهوتسبوت.
  2. في راوتر مايكروتك يتم تفعيل سكربت `On Login` في صفحة الهوتسبوت (`/ip hotspot user profile`) لقراءة قيمة `domain`:
  ```routeros
  # سكربت المايكروتك لتطبيق سرعة الكرت (اقتصادية/عادية/متوسطة/عالية) ديناميكياً:
  :local userDomain [/ip hotspot active get [find user="$user"] domain];
  :if ([:len $userDomain] > 0) do={
      :local targetSpeed [:pick $userDomain 0 [:find $userDomain "_"]];
      :if ([:len $targetSpeed] = 0) do={ :set targetSpeed $userDomain; }
      /queue simple set [find name="<hotspot-$user>"] max-limit=$targetSpeed;
  }
  ```
  3. عند تغيير السرعة من شاشة الحالة، يرسل المتصفح استعلام `ajax` إلى رابط المايكروتك فيقوم المايكروتك بتحديث الـ Simple Queue فوراً وتعديل السرعة المخصصة للكرت لحظياً دون فصل الاتصال.

---

### 3) إيقاف التحديثات والمتجر (Block Updates & Store)
- **العناصر في الشاشة**:
  - في تسجيل الدخول: `<input type="checkbox" value="_Uon" id="chupdate">`
  - في شاشة الحالة: `<select id="updatechange" class="tcs">`
- **آلية العمل**:
  - يرسل السكربت الرمز `_Uoff` للمايكروتك.
  - في جدار حماية المايكروتك (Firewall Mangle / Filter), يتم توجيه حزم التحديثات الخاصة بجوجل بلاي وآبل إلى قائمة حظر لتوفير رصيد كرت المشترك.

---

### 4) شاشة عرض الاستهلاك والعدادات (Live Status Dashboard)
يتم تحديث العناصر التالية تلقائياً من ردود `status.html`:
- `bytes_out`: التحميل (Download).
- `bytes_in`: الرفع (Upload).
- `remain_bytes_total`: الرصيد المتبقي للباقة.
- `uptime`: مدة الجلسة الحالية.
- `session_time_left`: الوقت المتبقي في الكرت.

---

### 5) تسجيل الخروج وقطع الاتصال والدخول بكرت سابق (مقارنة تفصيلية وتطابق مع نظام 2024)

بناءً على الفحص الشامل لشفرة نظام 2024 (`2024/index.html` و `2024/logout.html` و `2024/js/main.min.js` و `2024/js/hotCookie.min.js`)، فإن العمليات تعمل الآن بنفس الآلية الدقيقة والمطابقة:

#### أ) زر قطع الاتصال (Disconnect):
- **الوسم في القالب**:
  ```html
  <button class="login-submit app-logout">قطع الاتصال</button>
  ```
- **الطلب المرسل للراوتر**: يرسل طلب `GET /logout?var=callBack` (بدون السمة `erase-cookie`).
- **السلوك في المايكروتك**: يُغلق جلسة الهوتسبوت للعميل فوراً ويتوقف استهلاك الرصيد والوقت.
- **السلوك على الواجهة**:
  - يتوقف استعلام الحالة الدوري فوراً.
  - يتم التوجيه والعودة التلقائية إلى **صفحة تسجيل الدخول (`#login`)** داخل نفس التطبيق.
  - **يظل رقم الكرت وكلمة المرور في الحقل** لتمكين المستخدم من العودة والاتصال فوراً بضغطة زر واحدة دون إعادة كتابة الكرت.

#### ب) زر تسجيل الخروج (Logout):
- **الوسم في القالب**:
  ```html
  <button class="login-submit app-logout" erase-cookie clear-hot-cookie>تسجيل الخروج</button>
  ```
- **الطلب المرسل للراوتر**: يرسل طلب `GET /logout?erase-cookie=yes&var=callBack` (مع السمة `erase-cookie`).
- **السلوك في المايكروتك**: ينهي الجلسة ويلغي كوكيز المصادقة النشطة للهوتسبوت (`HTTP-Cookie`).
- **السلوك على الواجهة**:
  - يتم مسح كوكيز الجلسة النشطة الحالية فقط عبر دالة `clearCookies()` (`username`, `password`, `speed`).
  - يتم تفريغ حقل إدخال الكرت بالكامل لمنع الدخول التلقائي.
  - العودة إلى **صفحة تسجيل الدخول (`#login`)**.
  - **يبقى الكرت محفوظاً في سجل الكروت السابقة `storedValues` في المتصفح**.

#### ج) ميزة "دخول بكرت سابق" (Previous Card History):
- يتم حفظ كل كرت يتم إدخاله وتسجيل الدخول به بنجاح في مصفوفة الكروت السابقة `storedValues` (حتى 6 كروت).
- عند النقر على زر "دخول بكرت سابق"، تقوم دالة `getLoginCardCookie()` باستدعاء آخر كرت من المصفوفة وتعبئته مباشرة في الحقل، بحيث يعمل استرجاع الكرت فوراً حتى بعد تسجيل الخروج ومسح جلسة الراوتر النشطة، تماماً كما كان مصمماً في نظام 2024.

---

### جدول المقارنة بين الإجراءين:
| الميزة / الإجراء | زر قطع الاتصال (Disconnect) | زر تسجيل الخروج (Logout) |
| :--- | :--- | :--- |
| **الطلب للمايكروتك** | `/logout?var=callBack` | `/logout?erase-cookie=yes&var=callBack` |
| **الوجهة بعد النقر** | صفحة تسجيل الدخول (`#login`) | صفحة تسجيل الدخول (`#login`) |
| **حالة حقل الكرت** | يبقى معبأً برقم الكرت السابق | يُفرغ الحقل بالكامل |
| **كوكيز الجلسة الحالية** | تبقى محفوظة للاتصال السريع | تُمحى لمنع الدخول التلقائي |
| **سجل الكروت السابقة** | محفوظ في `storedValues` | محفوظ في `storedValues` لاسترجاعه بزر "دخول بكرت سابق" |

---

### 6) نظام الحظر ومكافحة التخمين (HotBlocker & Anti-Bruteforce)
- عند تكرار إدخال أرقام كروت خاطئة، يقوم سكربت `js/hotBlocker.min.js` بحساب عدد الأخطاء محلياً وربطه مع الراوتر.
- عند الوصول إلى `warn-when: 10`، يظهر تحذير للمستخدم بعدد المحاولات المتبقية.
- عند الوصول إلى `try-count: 20`، يتم قفل الشاشة والانتقال لشاشة الحظر `#block` مع بدء عداد تنازلي مدته `block-time: 1` دقيقة.

---

## 5. سكربتات الجافاسكريبت ودور كل ملف

1. **`js/init.min.js`**:
   - قراءة إعدادات `Config` من `config/config.js`.
   - استبدال النصوص الديناميكية في الصفحة مثل `{{network-name}}` و `{{service-number}}` و `{{news-line}}`.
   - فحص استجابة المايكروتك الأولى عبر الرابط `/login?var=callBack`.

2. **`js/hotCookie.min.js`**:
   - حفظ أرقام الكروت المستخدمة في الـ LocalStorage والكوكيز لتسهيل ميزة "دخول بكرت سابق".

3. **`js/hotOptions.min.js`**:
   - تهيئة خيارات السرعات والتحكم في إعدادات الباندويث وربطها مع المايكروتك.

4. **`js/hotBlocker.min.js`**:
   - إدارة سجل محاولات الدخول الخاطئة وإطلاق شاشة الحظر `#block`.

5. **`js/hotInImprover.min.js`**:
   - تحويل الأرقام العربية إلى إنجليزية، وتنظيف المسافات الزائدة في أرقام الكروت لمنع أخطاء الإدخال.

6. **`js/templates.min.js`**:
   - إنشاء جداول باقات الأسعار ونقاط البيع تلقائياً من المصفوفات في ملف `config.js`.

7. **`js/main.min.js`**:
   - المحرك الأساسي لإدارة التنقل بين الشاشات (`#login`, `#status`, `#price`, `#sell-point`, `#loan`, `#block`).
   - استقبال وتوزيع أحداث مايكروتك (`onLoginStart`, `onLoggedIn`, `onStatusQuery`, `onLoggedOut`, `onLoginError`).

8. **`js/jsqr.min.js`**:
   - مكتبة مسح وقراءة أكواد QR للكروت عبر كاميرا الهاتف مباشرة دون الحاجة للاتصال بالإنترنت.

---

## 6. خطوات الرفع والتشغيل على راوتر مايكروتك

1. افتح برنامج **WinBox** وسجل الدخول لراوتر المايكروتك.
2. توجه إلى القائمة الجانبية: **Files**.
3. قم بسحب وإفلات مجلد القالب (الذي يحتوي على `index.html`, `login.html`, `alogin.html`, `status.html`, `logout.html`, مجلد `js`, `css`, `img`, `fonts`, `config`) إلى داخل مجلد `hotspot` في الراوتر.
4. توجه إلى: **IP** -> **Hotspot** -> **Server Profiles**.
5. انقر مرتين على بروفايل الهوتسبوت الخاص بك، وتأكد من تعيين **HTML Directory** إلى اسم المجلد المرفوع (افتراضياً: `hotspot`).
6. ستعمل بوابة تسجيل الدخول بكافة وظائفها البصرية والتفاعلية فور اتصال أي هاتف أو كمبيوتر بالشبكة.
