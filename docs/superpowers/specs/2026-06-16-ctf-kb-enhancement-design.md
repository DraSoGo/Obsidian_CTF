# CTF Knowledge Base Enhancement - Design Specification

**Date:** 2026-06-16  
**Project:** Obsidian_CTF  
**Goal:** Comprehensive enhancement of CTF knowledge base with standardized formatting, expanded techniques, improved cross-references, and enhanced tool documentation

---

## 1. Overview

This specification outlines the comprehensive enhancement of the Obsidian_CTF knowledge base. The enhancement merges multiple improvement vectors: expanding technique coverage, standardizing formatting with Obsidian-specific features, enhancing tool documentation, and building a robust cross-reference system between techniques, tools, and categories.

### Current State
- 8 main categories (OSINT, Linux, Web App, Privilege Escalation, Cryptography, Network, Forensics, Reverse Engineering)
- 45 tool documentation files
- 8 technique files (sparse coverage, inconsistent depth)
- 4 resource references
- Mixed Thai/English content (maintain this style)
- Limited use of Obsidian features (callouts, WikiLinks)
- Inconsistent variable naming in examples

### Target State
- All existing techniques enhanced with comprehensive coverage
- 15+ new web exploitation technique files added
- All 45 tool files enhanced with additional examples and CTF use cases
- Consistent formatting across all notes using Obsidian callouts
- Extensive WikiLink cross-referencing
- Standardized variable naming (`<target_ip>` style)
- Production-ready command examples with proper comments

---

## 2. Content Structure Standards

### 2.1 Technique Note Template

Every technique note follows this structure:

```markdown
#web #cryptography #forensics (appropriate category tags)

# ❓ What
- คำอธิบายสั้น ๆ ว่าเทคนิคนี้คืออะไร (Brief explanation of the technique)
- ใช้ในสถานการณ์ไหน (When/where it's used)
- ทำไมต้องรู้ในบริบท CTF (Why it matters for CTF)

# 🔍 Landmark (Detection)
- สัญญาณที่บอกว่าเป้าหมายมีช่องโหว่นี้ (Signs the target is vulnerable)
- จุดที่ต้องมองหา (What to look for)
- เครื่องมือที่ใช้ตรวจสอบ (Tools for detection)

> [!TIP] CTF Trick
> เทคนิคเร็วสำหรับ CTF (Quick trick for CTF scenarios)

# 🟦 Types/Categories
แยกประเภทย่อย พร้อมอธิบายแต่ละแบบ (Subcategories with explanations)

## Type 1
คำอธิบาย

## Type 2
คำอธิบาย

# 🔨 Exploitation
## Basic Payloads
```bash
# Comment explaining what this does
command --option <target>
```

## Advanced Techniques
เทคนิคขั้นสูง with code examples

> [!WARNING] Common Pitfall
> สิ่งที่ต้องระวัง (Things to watch out for)

# 🚪 Bypass Techniques
วิธีหลบ filter, WAF, protection mechanisms

## Filter Bypass
Examples and techniques

## WAF Evasion
Examples and techniques

# 🛠️ Tool
- [[Tool Name]] : คำอธิบายสั้น ๆ และการใช้งานกับเทคนิคนี้
- [[Another Tool]] : คำอธิบาย

# 💡 Related
- [[Related Technique]] : ความเกี่ยวข้อง
- [[Related Category]] : ลิงก์ไปยังหมวดหมู่

# 📚 Example
ตัวอย่างจาก CTF จริง พร้อม step-by-step solution
```

### 2.2 Tool Documentation Enhancement Template

Existing tool files will be enhanced with:

```markdown
#tool

# ❓ What
- คำอธิบาย tool
- ใช้สำหรับอะไร

# 🎯 Use Cases
## CTF Scenario 1
สถานการณ์และวิธีใช้

## CTF Scenario 2
สถานการณ์และวิธีใช้

# 🔧 Installation
```bash
# Installation commands
```

# 📝 Basic Commands
```bash
# Common command with comments
tool --flag <target>
```

# 🚀 Advanced Usage
```bash
# Advanced scenarios
tool --advanced-flag <options>
```

> [!TIP] CTF Quick Win
> เทคนิคที่ช่วยประหยัดเวลาใน CTF

# 🔗 Related Techniques
- [[Technique Name]] : เกี่ยวข้องอย่างไร

# 💡 Troubleshooting
Common issues and solutions

# 📚 Examples
Real CTF examples with full walkthrough
```

### 2.3 Obsidian Callout Standards

**Callout Types:**
- `> [!TIP]` - CTF tricks, shortcuts, quick wins, time-savers
- `> [!WARNING]` - Common mistakes, pitfalls, things that break
- `> [!NOTE]` - Important concepts, reminders, key information
- `> [!EXAMPLE]` - Real CTF scenarios, walkthroughs

**Usage Guidelines:**
- Use liberally but purposefully
- Place CTF-specific tips prominently
- Warning callouts for common beginner mistakes
- Example callouts for complex multi-step scenarios

### 2.4 Variable Naming Standards

**Replace:**
- ❌ Real IPs: `10.10.212.122`
- ❌ Bracket placeholders: `[TARGET_IP]`
- ❌ Generic: `example.com`

**With:**
- ✅ Angle brackets: `<target_ip>`, `<domain>`, `<port>`
- ✅ CTF-style: `target.ctf`, `challenge.htb`
- ✅ Descriptive: `<database_name>`, `<table_name>`

**Code Comment Standards:**
- Every command must have a comment explaining what it does
- Multi-line commands need inline comments
- Python/Bash scripts must be production-ready with proper error handling

---

## 3. Content Expansion Plan

### 3.1 New Web Exploitation Techniques to Add

1. **XSS (Cross-Site Scripting).md**
   - Types: Reflected, Stored, DOM-based
   - Detection methods
   - Basic to advanced payloads
   - Filter bypass techniques (encoding, obfuscation)
   - CSP bypass
   - Tools: [[Burp Suite]], [[XSStrike]]
   - Real CTF examples

2. **CSRF (Cross-Site Request Forgery).md**
   - What is CSRF
   - Token-based protection
   - SameSite cookies
   - Bypass techniques
   - POC generation
   - Tools: [[Burp Suite]], [[CSRF POC Generator]]

3. **XXE (XML External Entity).md**
   - XML parsing vulnerabilities
   - Types: Classic, Blind, Error-based
   - File disclosure
   - SSRF via XXE
   - OOB (Out-of-Band) exploitation
   - Modern mitigations
   - Tools: [[Burp Suite]], [[XXEinjector]]

4. **SSRF (Server-Side Request Forgery).md**
   - Internal network access
   - Cloud metadata exploitation (AWS, Azure, GCP)
   - Bypass techniques (IP encoding, DNS rebinding)
   - Blind SSRF
   - Tools: [[SSRFmap]], [[Burp Suite]]

5. **Command Injection.md**
   - OS command injection
   - Detection landmarks
   - Basic payloads
   - Filter bypass (spaces, quotes, special chars)
   - Reverse shells
   - Tools: [[netcat]], [[Burp Suite]]

6. **LFI (Local File Inclusion).md**
   - Path traversal basics
   - Common targets (`/etc/passwd`, logs, config files)
   - Bypass techniques (null byte, encoding, wrappers)
   - PHP wrappers (`php://filter`, `data://`, `expect://`)
   - Log poisoning
   - LFI to RCE
   - Tools: [[Burp Suite]], [[LFISuite]]

7. **RFI (Remote File Inclusion).md**
   - Remote code execution via inclusion
   - Bypass allow_url_include restrictions
   - Payload hosting
   - Tools and techniques

8. **Path Traversal.md**
   - Directory traversal
   - Bypass filters (`../`, `..%2f`, etc.)
   - OS-specific paths
   - Tools: [[DotDotPwn]]

9. **File Upload Vulnerabilities.md**
   - Bypass file type validation
   - Magic byte manipulation
   - Double extension tricks
   - Content-Type bypass
   - Webshell upload
   - Tools: [[Burp Suite]]

10. **Insecure Deserialization.md**
    - Language-specific: PHP, Python, Java, Node.js
    - Detection
    - Exploitation techniques
    - Tools: [[ysoserial]]

11. **IDOR (Insecure Direct Object Reference).md**
    - Finding IDOR vulnerabilities
    - Parameter manipulation
    - Enumeration techniques
    - Tools: [[Burp Suite]], [[Autorize]]

12. **Open Redirect.md**
    - Detection methods
    - Bypass techniques
    - Chaining with other vulnerabilities

13. **CORS Misconfiguration.md**
    - What is CORS
    - Exploitation
    - POC creation

14. **JWT Vulnerabilities.md**
    - JWT structure
    - Algorithm confusion (alg: none)
    - Key confusion attacks
    - Weak secrets
    - Tools: [[jwt_tool]]

15. **API Security.md**
    - REST API testing
    - Authentication flaws
    - Rate limiting bypass
    - Tools: [[Postman]], [[Burp Suite]]

### 3.2 Existing Technique Enhancements

1. **SQL Injection.md** (current: sparse)
   - Add: Error-based, Time-based, Boolean-based blind SQLi
   - Add: Second-order SQLi
   - Add: NoSQL injection
   - Add: WAF bypass techniques
   - Expand examples with detailed walkthroughs
   - Add database-specific techniques (MySQL, PostgreSQL, MSSQL, Oracle)

2. **Authentication.md** (current: basic)
   - Add: Multi-factor bypass techniques
   - Add: Session fixation
   - Add: OAuth vulnerabilities
   - Add: Password reset flaws
   - Add: Brute force optimization

3. **SUID.md** (current: minimal)
   - Expand with more exploitation examples
   - Add GTFOBins integration
   - Add custom binary exploitation

4. **Sudo.md** (current: minimal)
   - Expand with CVE examples
   - Add misconfiguration exploitation
   - GTFOBins cross-reference

5. **Crontab Exploit.md**
   - Add more exploitation vectors
   - Path hijacking techniques

6. **Race Condition.md**
   - Expand with TOCTOU examples
   - Add exploitation scripts

7. **Web Reconnaissance.md**
   - Expand methodology
   - Add more tools and techniques
   - Subdomain enumeration
   - Technology fingerprinting

8. **Passwd.md**
   - Expand exploitation techniques
   - Add hash cracking strategies

### 3.3 Tool Documentation Enhancements

All 45 existing tool files will receive:
- Additional "CTF Use Cases" section with 2-3 scenarios
- More command examples with comments
- "Quick Reference" section for common CTF commands
- Troubleshooting tips
- Cross-references to related techniques
- Real CTF examples where applicable

**Priority tool enhancements:**
1. [[Burp Suite]] - Add more advanced features (Intruder, Repeater workflows)
2. [[SQLMap]] - Already good, add more advanced flags
3. [[Nmap]] - Add CTF-specific scanning strategies
4. [[Metasploit]] - Add module selection guidance
5. [[Hydra]] - Add more protocol examples
6. [[Gobuster]] - Add wordlist recommendations
7. [[John The Ripper]] - Add hash type identification
8. [[Wireshark]] - Add CTF-specific filters
9. [[Radare2]] - Add more RE workflows
10. [[Binwalk]] - Add firmware extraction examples

---

## 4. Cross-Reference Architecture

### 4.1 WikiLink Strategy

**Three-layer cross-referencing:**

1. **Technique ↔ Tool**
   - Every technique lists relevant tools
   - Every tool lists techniques it's used for

2. **Technique ↔ Category**
   - Techniques link back to parent category
   - Categories maintain comprehensive technique lists

3. **Technique ↔ Technique**
   - Related techniques cross-reference
   - Attack chain connections (e.g., LFI → RCE)

**Example WikiLink density:**
- Each technique note: 8-15 WikiLinks
- Each tool note: 5-10 WikiLinks
- Each category note: 20-40 WikiLinks

### 4.2 Category Page Updates

All 8 category pages will be updated to include:
- Complete technique listings with brief descriptions
- Complete tool listings with use cases
- Cross-category references (e.g., Web App ↔ Network)
- CTF workflow recommendations

---

## 5. Implementation Strategy

### 5.1 Phased Approach

**Phase 1: Existing Content Enhancement (Days 1-2)**
- Enhance SQL Injection.md to full template
- Enhance 8 existing technique files
- Update variable naming across all files
- Add Obsidian callouts where needed

**Phase 2: New Technique Creation (Days 3-5)**
- Create 15 new web exploitation technique files
- Follow template structure
- Include WikiLinks as created

**Phase 3: Tool Enhancement (Days 6-7)**
- Enhance priority tools (top 10)
- Add CTF use cases to all 45 tools
- Add cross-references

**Phase 4: Category Integration (Day 8)**
- Update all 8 category pages
- Ensure all WikiLinks are bidirectional
- Add cross-category references

**Phase 5: Quality Assurance (Day 9)**
- Verify all WikiLinks resolve
- Check consistency
- Add missing cross-references
- Fix formatting issues

### 5.2 File Organization

**No structural changes:**
- `/Category/` - 8 category files (update)
- `/Technic/` - 8 existing + 15 new = 23 technique files
- `/Tool/` - 45 files (enhance)
- `/Resource/` - 4 files (maintain)

---

## 6. Quality Standards

### 6.1 Completeness Criteria

Each technique file must have:
- ✅ Minimum 150 lines of content
- ✅ All template sections filled
- ✅ At least 3 code examples with comments
- ✅ At least 2 Obsidian callouts
- ✅ At least 5 WikiLinks
- ✅ At least 1 real CTF example

Each enhanced tool file must have:
- ✅ At least 2 CTF use case scenarios
- ✅ At least 5 command examples
- ✅ At least 3 WikiLinks to techniques
- ✅ Quick reference section

### 6.2 Language Standards

- Thai for explanations and concepts
- English for technical terms (maintained as-is, not translated)
- Command examples with Thai comments
- Consistent tone matching existing content

### 6.3 Code Standards

**Bash/Command examples:**
```bash
# คำอธิบายว่าคำสั่งนี้ทำอะไร (Thai comment)
command --flag <target>
```

**Python scripts:**
- Must be production-ready (imports, error handling, proper structure)
- Thai comments for clarity
- No placeholders, use argparse or clear variable naming

### 6.4 Variable Naming

**Standardized variables:**
- `<target_ip>` - Target IP address
- `<target_url>` - Target URL
- `<port>` - Port number
- `<username>` - Username
- `<password>` - Password
- `<payload>` - Injection payload
- `<database>` - Database name
- `<table>` - Table name
- `<column>` - Column name
- `<file>` - File path
- `<domain>` - Domain name

---

## 7. Examples

### 7.1 Before/After: SQL Injection.md

**Before (current state):**
- 51 lines total
- Basic SQLMap examples
- Union-based SQLi only
- Real IPs in examples
- No callouts
- Minimal cross-references

**After (enhanced):**
- 250+ lines
- All SQLi types covered (Error, Boolean, Time, Union, Stacked)
- NoSQL injection added
- WAF bypass techniques
- Blind SQLi methodologies
- Database-specific techniques
- Obsidian callouts for tips/warnings
- Extensive WikiLinks to tools and related techniques
- Real CTF walkthroughs
- Standardized variables

### 7.2 Before/After: Tool Enhancement Example (Nmap)

**Before:**
```markdown
# What
Network scanner

# Command
```bash
nmap -sV 10.10.10.5
```
```

**After:**
```markdown
#tool #network

# ❓ What
- Network scanner สำหรับค้นหา host, port, และ service
- เครื่องมือพื้นฐานสำหรับ reconnaissance ใน CTF

# 🎯 CTF Use Cases
## Scenario 1: Quick Port Scan
หา port ที่เปิดอยู่อย่างรวดเร็ว
```bash
# Scan top 1000 ports
nmap -T4 -F <target_ip>
```

## Scenario 2: Service Detection
ตรวจสอบ version ของ service
```bash
# Service version detection
nmap -sV -p- <target_ip>
```

# 📝 Basic Commands
```bash
# TCP SYN scan (ไม่ทำ full connection)
nmap -sS <target_ip>

# UDP scan
nmap -sU <target_ip>

# OS detection
nmap -O <target_ip>
```

> [!TIP] CTF Quick Win
> ใช้ -T4 หรือ -T5 เพื่อเพิ่มความเร็วในการ scan (ระวังทำให้ตรวจจับได้ง่าย)

# 🔗 Related Techniques
- [[Web Reconnaissance]] : ขั้นตอนแรกของการ recon
- [[Network]] : หมวดหมู่หลัก

# 📚 Examples
### Example: Finding Hidden SSH on High Port
```bash
# Scan all ports to find SSH on non-standard port
nmap -p- --min-rate=1000 <target_ip>
```
```

---

## 8. Success Metrics

**Quantitative:**
- 23 total technique files (8 enhanced + 15 new)
- 45 enhanced tool files
- 8 updated category files
- 150+ new WikiLinks created
- 50+ Obsidian callouts added

**Qualitative:**
- Consistent formatting across all notes
- Production-ready code examples
- Comprehensive coverage of web exploitation
- Strong cross-reference network
- CTF-ready reference material

---

## 9. Maintenance Plan

**Post-implementation:**
- Add new techniques as encountered in CTFs
- Update tools when new features are discovered
- Add real CTF examples as solved
- Keep cross-references updated
- Maintain formatting consistency

**Regular reviews:**
- Monthly: Check for broken WikiLinks
- Quarterly: Review and update tool documentation
- After major CTFs: Add new examples and techniques

---

## 10. Timeline

**Total estimated time:** 9 days

- **Days 1-2:** Existing content enhancement
- **Days 3-5:** New technique creation
- **Days 6-7:** Tool enhancement
- **Day 8:** Category integration
- **Day 9:** Quality assurance

---

## 11. Notes

- Maintain mixed Thai/English style throughout
- All code must be production-ready
- Focus on CTF-specific tips and tricks
- Real-world examples prioritized
- No theoretical content without practical application
- Keep scannability high with clear headers and callouts
