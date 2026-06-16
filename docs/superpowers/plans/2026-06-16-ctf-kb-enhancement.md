# CTF Knowledge Base Enhancement Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Comprehensive enhancement of Obsidian CTF knowledge base with standardized formatting, 15 new techniques, enhanced tool docs, and extensive cross-references.

**Architecture:** Content enhancement follows spec at `docs/superpowers/specs/2026-06-16-ctf-kb-enhancement-design.md`. Each task produces complete markdown files following template structure with Thai/English mix, Obsidian callouts, WikiLinks, and standardized variables (`<target_ip>` style).

**Tech Stack:** Markdown, Obsidian WikiLinks, Git

---

## Implementation Notes

**Template Reference:** See spec section 2.1 for complete technique template structure
**Quality Standards:** Each technique file must have 150+ lines, 3+ code examples, 2+ callouts, 5+ WikiLinks, 1+ real CTF example
**Variable Standard:** Use `<target_ip>`, `<target_url>`, `<database>`, `<table>`, `<payload>` instead of real IPs or bracket placeholders
**Language:** Thai for explanations, English for technical terms, Thai comments in code

---

## Phase 1: Existing Technique Enhancements

### Task 1: Enhance SQL Injection.md

**Files:**
- Modify: `Technic/SQL Injection.md` (51 lines → 250+ lines target)

- [ ] **Step 1: Expand SQL Injection with all types**

Add sections: Union-Based, Error-Based, Boolean-Based Blind, Time-Based Blind, Stacked Queries, Second-Order, NoSQL Injection
Include exploitation examples for each type with Thai comments
Add database-specific techniques (MySQL, PostgreSQL, MSSQL, Oracle)

- [ ] **Step 2: Add WAF bypass techniques**

Add Filter Bypass section with encoding techniques, comment injection, case variation
Add example bypasses for common WAF rules

- [ ] **Step 3: Add Obsidian callouts and WikiLinks**

Add `> [!TIP]` for quick detection tricks
Add `> [!WARNING]` for common mistakes
Add WikiLinks to [[SQLMap]], [[Burp Suite]], [[Authentication]], [[🌐 Web application]]

- [ ] **Step 4: Standardize variables**

Replace all real IPs with `<target_ip>`, `<target_url>`, `<database>`, `<table>`
Add Thai comments to all command examples

- [ ] **Step 5: Add real CTF example**

Add step-by-step CTF walkthrough in Example section

- [ ] **Step 6: Verify and commit**

```bash
cd /home/drasogo/Documents/Obsidian_CTF
git add "Technic/SQL Injection.md"
git commit -m "Enhance SQL Injection technique with all types, WAF bypass, and examples"
```

---

### Task 2: Enhance Authentication.md

**Files:**
- Modify: `Technic/Authentication.md`

- [ ] **Step 1: Expand Authentication techniques**

Add: Multi-factor bypass, Session fixation, OAuth vulnerabilities, Password reset flaws, Brute force optimization

- [ ] **Step 2: Add callouts and WikiLinks**

Add [[Hydra]], [[Burp Suite]], [[🌐 Web application]]

- [ ] **Step 3: Commit**

```bash
git add "Technic/Authentication.md"
git commit -m "Enhance Authentication with advanced bypass techniques"
```

### Task 3: Enhance SUID.md

**Files:**
- Modify: `Technic/SUID.md`

- [ ] **Step 1: Expand SUID exploitation**

Add more exploitation examples, GTFOBins integration, custom binary exploitation

- [ ] **Step 2: Add WikiLinks**

Add [[GTFOBins]], [[⬆️ Privilege Escalation Linux]]

- [ ] **Step 3: Commit**

```bash
git add "Technic/SUID.md"
git commit -m "Enhance SUID with GTFOBins integration and examples"
```

---

### Task 4: Enhance Sudo.md

**Files:**
- Modify: `Technic/Sudo.md`

- [ ] **Step 1: Expand Sudo techniques**

Add CVE examples (CVE-2021-3156, CVE-2019-14287), misconfiguration exploitation

- [ ] **Step 2: Add WikiLinks**

Add [[GTFOBins]], [[⬆️ Privilege Escalation Linux]]

- [ ] **Step 3: Commit**

```bash
git add "Technic/Sudo.md"
git commit -m "Enhance Sudo with CVE examples and misconfigurations"
```

### Task 5: Enhance Remaining Existing Techniques

**Files:**
- Modify: `Technic/Crontab Exploit.md`, `Technic/Race Condition.md`, `Technic/Web Reconnaissance.md`, `Technic/Passwd.md`

- [ ] **Step 1: Enhance Crontab Exploit.md**

Add more exploitation vectors, path hijacking techniques, WikiLinks

- [ ] **Step 2: Enhance Race Condition.md**

Add TOCTOU examples, exploitation scripts, WikiLinks

- [ ] **Step 3: Enhance Web Reconnaissance.md**

Expand methodology, add subdomain enumeration, technology fingerprinting, WikiLinks

- [ ] **Step 4: Enhance Passwd.md**

Add hash cracking strategies, John/Hashcat usage, WikiLinks

- [ ] **Step 5: Commit all**

```bash
git add Technic/Crontab\ Exploit.md Technic/Race\ Condition.md Technic/Web\ Reconnaissance.md Technic/Passwd.md
git commit -m "Enhance Crontab, Race Condition, Web Recon, and Passwd techniques"
```

---

## Phase 2: New Technique Creation

### Task 6: Create XSS (Cross-Site Scripting).md

**Files:**
- Create: `Technic/XSS (Cross-Site Scripting).md`

- [ ] **Step 1: Create XSS technique file**

Follow template with sections: What, Landmark, Types (Reflected/Stored/DOM-based), Exploitation (basic/advanced payloads), Bypass Techniques (filter/CSP), Tools, Related, Example

- [ ] **Step 2: Add WikiLinks**

Add [[Burp Suite]], [[🌐 Web application]]

- [ ] **Step 3: Commit**

```bash
git add "Technic/XSS (Cross-Site Scripting).md"
git commit -m "Add XSS technique with all types and bypass methods"
```

---

### Task 7: Create CSRF (Cross-Site Request Forgery).md

**Files:**
- Create: `Technic/CSRF (Cross-Site Request Forgery).md`

- [ ] **Step 1: Create CSRF technique file**

Follow template: token-based protection, SameSite cookies, bypass techniques, POC generation

- [ ] **Step 2: Add WikiLinks**

Add [[Burp Suite]], [[🌐 Web application]]

- [ ] **Step 3: Commit**

```bash
git add "Technic/CSRF (Cross-Site Request Forgery).md"
git commit -m "Add CSRF technique with bypass and POC generation"
```

### Task 8: Create XXE, SSRF, Command Injection

**Files:**
- Create: `Technic/XXE (XML External Entity).md`
- Create: `Technic/SSRF (Server-Side Request Forgery).md`
- Create: `Technic/Command Injection.md`

- [ ] **Step 1: Create XXE technique file**

Follow template: Classic/Blind/Error-based, file disclosure, SSRF via XXE, OOB exploitation

- [ ] **Step 2: Create SSRF technique file**

Follow template: internal network access, cloud metadata (AWS/Azure/GCP), bypass techniques, blind SSRF

- [ ] **Step 3: Create Command Injection file**

Follow template: OS command injection, filter bypass (spaces/quotes/special chars), reverse shells

- [ ] **Step 4: Commit batch**

```bash
git add "Technic/XXE (XML External Entity).md" "Technic/SSRF (Server-Side Request Forgery).md" "Technic/Command Injection.md"
git commit -m "Add XXE, SSRF, and Command Injection techniques"
```

---

### Task 9: Create LFI, RFI, Path Traversal

**Files:**
- Create: `Technic/LFI (Local File Inclusion).md`
- Create: `Technic/RFI (Remote File Inclusion).md`
- Create: `Technic/Path Traversal.md`

- [ ] **Step 1: Create LFI technique file**

Follow template: path traversal, PHP wrappers (php://filter, data://, expect://), log poisoning, LFI to RCE

- [ ] **Step 2: Create RFI technique file**

Follow template: remote code execution via inclusion, bypass restrictions, payload hosting

- [ ] **Step 3: Create Path Traversal file**

Follow template: directory traversal, bypass filters (../, ..%2f), OS-specific paths

- [ ] **Step 4: Commit batch**

```bash
git add "Technic/LFI (Local File Inclusion).md" "Technic/RFI (Remote File Inclusion).md" "Technic/Path Traversal.md"
git commit -m "Add LFI, RFI, and Path Traversal techniques"
```

### Task 10: Create File Upload, Deserialization, IDOR

**Files:**
- Create: `Technic/File Upload Vulnerabilities.md`
- Create: `Technic/Insecure Deserialization.md`
- Create: `Technic/IDOR (Insecure Direct Object Reference).md`

- [ ] **Step 1: Create File Upload Vulnerabilities file**

Follow template: bypass file type validation, magic byte manipulation, double extension tricks, webshell upload

- [ ] **Step 2: Create Insecure Deserialization file**

Follow template: language-specific (PHP/Python/Java/Node.js), detection, exploitation

- [ ] **Step 3: Create IDOR file**

Follow template: finding IDOR, parameter manipulation, enumeration techniques

- [ ] **Step 4: Commit batch**

```bash
git add "Technic/File Upload Vulnerabilities.md" "Technic/Insecure Deserialization.md" "Technic/IDOR (Insecure Direct Object Reference).md"
git commit -m "Add File Upload, Deserialization, and IDOR techniques"
```

---

### Task 11: Create Open Redirect, CORS, JWT, API Security

**Files:**
- Create: `Technic/Open Redirect.md`
- Create: `Technic/CORS Misconfiguration.md`
- Create: `Technic/JWT Vulnerabilities.md`
- Create: `Technic/API Security.md`

- [ ] **Step 1: Create Open Redirect file**

Follow template: detection methods, bypass techniques, chaining with other vulns

- [ ] **Step 2: Create CORS Misconfiguration file**

Follow template: CORS basics, exploitation, POC creation

- [ ] **Step 3: Create JWT Vulnerabilities file**

Follow template: JWT structure, algorithm confusion (alg: none), key confusion, weak secrets

- [ ] **Step 4: Create API Security file**

Follow template: REST API testing, authentication flaws, rate limiting bypass

- [ ] **Step 5: Commit batch**

```bash
git add "Technic/Open Redirect.md" "Technic/CORS Misconfiguration.md" "Technic/JWT Vulnerabilities.md" "Technic/API Security.md"
git commit -m "Add Open Redirect, CORS, JWT, and API Security techniques"
```

---

## Phase 3: Tool Enhancement

### Task 12: Enhance Priority Tools (Burp Suite, SQLMap, Nmap)

**Files:**
- Modify: `Tool/Burp Suite.md` (if exists, else create)
- Modify: `Tool/SQLMap.md`
- Modify: `Tool/Nmap.md` (if exists, else create)

- [ ] **Step 1: Enhance SQLMap.md**

Add: CTF Use Cases section, advanced flags (-r, --tamper, --batch), troubleshooting, WikiLinks to techniques

- [ ] **Step 2: Create/enhance Burp Suite.md**

Add: Intruder workflows, Repeater usage, CTF scenarios, WikiLinks

- [ ] **Step 3: Create/enhance Nmap.md**

Add: CTF-specific scanning strategies, service detection, WikiLinks

- [ ] **Step 4: Commit batch**

```bash
git add Tool/SQLMap.md Tool/Burp\ Suite.md Tool/Nmap.md
git commit -m "Enhance priority tools with CTF use cases"
```

---

### Task 13: Enhance Additional Priority Tools

**Files:**
- Modify: `Tool/Metasploit.md`, `Tool/Hydra.md`, `Tool/Gobuster.md`, `Tool/John The Ripper.md`, `Tool/Wireshark.md`

- [ ] **Step 1: Enhance each tool file**

For each tool: add CTF Use Cases (2-3 scenarios), more command examples with Thai comments, WikiLinks to related techniques

- [ ] **Step 2: Commit batch**

```bash
git add Tool/Metasploit.md Tool/Hydra.md Tool/Gobuster.md Tool/John\ The\ Ripper.md Tool/Wireshark.md
git commit -m "Enhance Metasploit, Hydra, Gobuster, JTR, Wireshark tools"
```

### Task 14: Enhance Remaining Tools (Batch Processing)

**Files:**
- Modify: All remaining 35+ tool files in `Tool/` directory

- [ ] **Step 1: List all tool files**

```bash
cd /home/drasogo/Documents/Obsidian_CTF
find Tool -name "*.md" -type f | sort > /tmp/tool_list.txt
```

- [ ] **Step 2: Batch enhance tool files**

For each remaining tool file:
- Add CTF Use Cases section with 2 scenarios
- Add more command examples (minimum 3) with Thai comments
- Add WikiLinks to related techniques (minimum 2)
- Add Quick Reference section if applicable
- Standardize variable naming

Process in batches of 5-10 files per commit

- [ ] **Step 3: Commit in batches**

```bash
# Example for batch 1
git add Tool/Git.md Tool/Radare2.md Tool/Binwalk.md Tool/Ciphey.md Tool/Steghide.md
git commit -m "Enhance Git, Radare2, Binwalk, Ciphey, Steghide tools"

# Continue for remaining tools
```

---

## Phase 4: Category Integration

### Task 15: Update All Category Pages

**Files:**
- Modify: `Category/🌐 Web application.md`
- Modify: `Category/🔐 Cryptography.md`
- Modify: `Category/🧿 Forensics.md`
- Modify: `Category/🔃 Reverse Engineering.md`
- Modify: `Category/🕸️ Network.md`
- Modify: `Category/🐧 Linux.md`
- Modify: `Category/⬆️ Privilege Escalation Linux.md`
- Modify: `Category/🔍 OSINT.md`

- [ ] **Step 1: Update Web Application category**

Add all new web techniques to the Technic section with WikiLinks and brief descriptions

- [ ] **Step 2: Update other categories**

Ensure each category lists all relevant techniques and tools with WikiLinks

- [ ] **Step 3: Add cross-category references**

Add Related Categories section linking related areas (e.g., Web App ↔ Network, Linux ↔ Privilege Escalation)

- [ ] **Step 4: Commit all categories**

```bash
git add Category/*.md
git commit -m "Update all categories with complete technique/tool listings and cross-references"
```

---

## Phase 5: Quality Assurance

### Task 16: Verify WikiLinks and Cross-References

**Files:**
- All files in `Technic/`, `Tool/`, `Category/`

- [ ] **Step 1: Check for broken WikiLinks**

```bash
cd /home/drasogo/Documents/Obsidian_CTF
# List all WikiLinks used in technique files
grep -rh '\[\[.*\]\]' Technic/ | grep -o '\[\[.*\]\]' | sort -u > /tmp/wikilinks.txt
```

- [ ] **Step 2: Verify WikiLink targets exist**

Manually verify that each WikiLink has a corresponding file or is a valid cross-reference

- [ ] **Step 3: Add missing WikiLinks**

If any cross-references are missing, add them to relevant files

- [ ] **Step 4: Commit fixes**

```bash
git add -A
git commit -m "Fix broken WikiLinks and add missing cross-references"
```

---

### Task 17: Verify Quality Standards

**Files:**
- All technique and tool files

- [ ] **Step 1: Check technique files meet standards**

For each technique file, verify:
- Minimum 150 lines of content
- At least 3 code examples with Thai comments
- At least 2 Obsidian callouts
- At least 5 WikiLinks
- At least 1 real CTF example
- All variables use `<variable>` format

- [ ] **Step 2: Check tool files meet standards**

For each tool file, verify:
- At least 2 CTF use case scenarios
- At least 5 command examples
- At least 3 WikiLinks to techniques
- Quick reference or troubleshooting section

- [ ] **Step 3: Fix any quality issues**

Update files that don't meet standards

- [ ] **Step 4: Commit final fixes**

```bash
git add -A
git commit -m "Final quality assurance: ensure all files meet standards"
```

---

### Task 18: Final Review and Documentation

**Files:**
- All modified/created files

- [ ] **Step 1: Count deliverables**

```bash
cd /home/drasogo/Documents/Obsidian_CTF
echo "Technique files:" && ls Technic/*.md | wc -l
echo "Tool files:" && ls Tool/*.md | wc -l
echo "Category files:" && ls Category/*.md | wc -l
```

- [ ] **Step 2: Create completion summary**

Verify success metrics:
- 23 total technique files (8 enhanced + 15 new)
- 45 enhanced tool files
- 8 updated category files
- 150+ new WikiLinks created
- 50+ Obsidian callouts added

- [ ] **Step 3: Final commit**

```bash
git add -A
git commit -m "Complete CTF knowledge base enhancement

- Enhanced 8 existing techniques with comprehensive coverage
- Created 15 new web exploitation techniques
- Enhanced all 45 tool files with CTF use cases
- Updated 8 category files with complete cross-references
- Standardized formatting with Obsidian callouts and WikiLinks
- Applied consistent variable naming throughout"
```

---

## Plan Complete

**Total Tasks:** 18 tasks covering 5 phases
**Estimated Time:** 9 days (per spec timeline)
**Success Criteria:** All technique files 150+ lines, all tools enhanced, all WikiLinks bidirectional, consistent formatting throughout


