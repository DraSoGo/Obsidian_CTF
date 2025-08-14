#cryptography
# What
- ไว้แก้ hash
# Command
format
```bash
hashcat -m [<hash_type>](https://hashcat.net/wiki/doku.php?id=example_hashes) -a <attack_mode> hashfile wordlist
```
Example command
```bash
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```