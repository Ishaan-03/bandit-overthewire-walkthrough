# OverTheWire Bandit — Complete Walkthrough (Levels 0–33)

This repository documents my personal playthrough of the [OverTheWire Bandit](https://overthewire.org/wargames/bandit/) wargame, a beginner-friendly series of Linux command-line challenges widely used to build fundamental skills for cybersecurity, system administration, and CTF (Capture The Flag) work.

Each level requires finding a password hidden somewhere on the server (in a file, inside a process, behind an encoding scheme, etc.) using nothing but the Linux command line. That password is then used to log in as the next level's user account.

> **Note on passwords:** The passwords below are the ones I obtained while completing each level. OverTheWire periodically resets/rotates these values for security reasons, so if you play the game yourself your passwords will very likely be different. Use this repo to learn the *technique* for each level, not to skip the game by reusing these exact values.

## How to connect

```bash
ssh -p 2220 bandit0@bandit.labs.overthewire.org
# password: bandit0 (the only publicly known starting password)
```

Each subsequent level is reached with:

```bash
ssh -p 2220 banditN@bandit.labs.overthewire.org
```

---

## Level 0 → 1
**Goal:** Log in for the first time and find the password hidden in a file called `readme` in the home directory.

**Commands used:**
```bash
ssh -p 2220 bandit0@bandit.labs.overthewire.org
ls
cat readme
```

**What it teaches:** Basic SSH login and the two most fundamental Linux commands — `ls` (list files) and `cat` (print file contents).

**Password for bandit1:** `<redacted>`

---

## Level 1 → 2
**Goal:** Read a file literally named `-` (a single dash).

**Commands used:**
```bash
cat ./-
```

**What it teaches:** A bare `-` is normally interpreted by many commands as "read from stdin", so `cat -` alone would hang. Prefixing the filename with `./` forces the shell to treat it as a relative path instead.

**Password for bandit2:** `<redacted>`

---

## Level 2 → 3
**Goal:** Read a file named `spaces in this filename`.

**Commands used:**
```bash
cat "spaces in this filename"
```

**What it teaches:** Filenames containing spaces must be quoted or have each space escaped with a backslash, otherwise the shell treats them as separate arguments.

**Password for bandit3:** `<redacted>`

---

## Level 3 → 4
**Goal:** Find a hidden file inside the `inhere` directory.

**Commands used:**
```bash
cd inhere
ls -a
cat .hidden
```

**What it teaches:** Files/directories starting with a dot are "hidden" from a plain `ls` and require the `-a` (all) flag to be shown.

**Password for bandit4:** `<redacted>`

---

## Level 4 → 5
**Goal:** Among several files with misleading names in `inhere`, find the one human-readable file.

**Commands used:**
```bash
cd inhere
file ./*
cat ./-file07
```

**What it teaches:** The `file` command inspects actual file content/type rather than trusting the filename, which is critical since filenames can be misleading.

**Password for bandit5:** `<redacted>`

---

## Level 5 → 6
**Goal:** In a maze of nested directories, find a file that is human-readable, exactly 1033 bytes, and not executable.

**Commands used:**
```bash
find . -type f -size 1033c ! -executable
```

**What it teaches:** `find` can filter by size (`-size`), permissions (`-executable`/`-perm`), and type, which is far more efficient than manually browsing a large directory tree.

**Password for bandit6:** `<redacted>`

---

## Level 6 → 7
**Goal:** Find a file anywhere on the entire filesystem owned by user `bandit7`, group `bandit6`, and exactly 33 bytes in size.

**Commands used:**
```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

**What it teaches:** `find` can search from the filesystem root, and redirecting stderr (`2>/dev/null`) hides "Permission denied" noise from directories you can't access.

**Password for bandit7:** `<redacted>`

---

## Level 7 → 8
**Goal:** The password sits in `data.txt` next to the word "millionth".

**Commands used:**
```bash
grep millionth data.txt
```

**What it teaches:** `grep` searches text for a pattern — an essential tool for filtering large files down to relevant lines.

**Password for bandit8:** `<redacted>`

---

## Level 8 → 9
**Goal:** `data.txt` contains many duplicate lines; the password is the one line that occurs exactly once.

**Commands used:**
```bash
sort data.txt | uniq -u
```

**What it teaches:** `uniq` only detects duplicates in *adjacent* lines, so the input must be `sort`-ed first; `-u` then prints only unique (non-repeated) lines.

**Password for bandit9:** `<redacted>`

---

## Level 9 → 10
**Goal:** `data.txt` is mostly binary garbage; the password is a human-readable string preceded by several `=` characters.

**Commands used:**
```bash
strings data.txt | grep '='
```

**What it teaches:** `strings` extracts printable text from binary data, and piping it into `grep` narrows down the noise to the exact pattern you're looking for.

**Password for bandit10:** `<redacted>`

---

## Level 10 → 11
**Goal:** `data.txt` holds the password encoded in Base64.

**Commands used:**
```bash
base64 -d data.txt
```

**What it teaches:** Base64 is a common, easily reversible text-encoding scheme, not encryption — decoding it requires no secret key.

**Password for bandit11:** `<redacted>`

---

## Level 11 → 12
**Goal:** `data.txt` holds the password with every letter rotated 13 places (ROT13).

**Commands used:**
```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**What it teaches:** `tr` translates characters one-for-one, letting you implement simple substitution ciphers like ROT13 directly on the command line.

**Password for bandit12:** `<redacted>`

---

## Level 12 → 13
**Goal:** `data.txt` is a hexdump of a file that has been compressed multiple times in a row with different tools.

**Commands used:**
```bash
mkdir /tmp/work12 && cp data.txt /tmp/work12 && cd /tmp/work12
xxd -r data.txt > data.bin
file data.bin        # identify compression type at each step
# repeat: rename with correct extension, then one of:
gzip -d file.gz
bzip2 -d file.bz2
tar xf file.tar
# ...until "file" reports plain ASCII text
```

**What it teaches:** `xxd -r` reverses a hex dump back into raw binary; `file` identifies the actual compression format so you can pick the right decompression tool; some files are wrapped in several layers, requiring the process to be repeated.

**Password for bandit13:** `<redacted>`

---

## Level 13 → 14
**Goal:** Use a provided private SSH key (`sshkey.private`) to log in directly as `bandit14`.

**Commands used:**
```bash
chmod 600 sshkey.private
ssh -i sshkey.private -p 2220 bandit14@localhost
```

**What it teaches:** SSH key-based authentication as an alternative to passwords, and why private keys must have restrictive permissions (`600`) before SSH will accept them.

**Result:** Logged in directly with the private key (no plaintext password needed for this hop).

---

## Level 14 → 15
**Goal:** The current level's password (readable only by that user, in `/etc/bandit_pass/`) must be submitted to a listening service on port 30000 on localhost to receive the next password.

**Commands used:**
```bash
cat /etc/bandit_pass/bandit14
nc localhost 30000
# paste the password, press enter
```

**What it teaches:** Interacting with a raw TCP service using `nc` (netcat), and the idea of a simple password-based challenge/response service.

**Password for bandit15:** `<redacted>`

---

## Level 15 → 16
**Goal:** Same idea as level 14, but the listening service (port 30001) requires an SSL/TLS-encrypted connection.

**Commands used:**
```bash
openssl s_client -connect localhost:30001 -quiet
# paste the password, press enter
```

**What it teaches:** Not all network services are plaintext; `openssl s_client` lets you manually speak TLS to a server the same way `nc` speaks plain TCP.

**Password for bandit16:** `<redacted>`
**Additional value obtained:** `<redacted>`

---

## Level 16 → 17
**Goal:** Scan a range of ports (31000–32000) on localhost to find the one SSL service that accepts the current password and, in return, hands back a private RSA key for the next level.

**Commands used:**
```bash
nmap -p31000-32000 localhost
openssl s_client -connect localhost:<port> -quiet
# paste password, receive private key in response
```

**What it teaches:** Port scanning with `nmap` to enumerate open services, then probing each candidate manually — a small taste of real-world reconnaissance.

**Result for bandit17:** A private SSH key (rather than a plaintext password).

---

## Level 17 → 18
**Goal:** Two files, `passwords.old` and `passwords.new`, are almost identical except for one line — the new password.

**Commands used:**
```bash
diff passwords.old passwords.new
```

**What it teaches:** `diff` quickly highlights differences between two versions of a file, which is invaluable for spotting a single changed line among thousands.

**Password for bandit18:** `<redacted>`

---

## Level 18 → 19
**Goal:** This account's `.bashrc` logs you out immediately on interactive login, so a normal `ssh` session never gives you a shell prompt.

**Commands used:**
```bash
ssh -p 2220 bandit18@bandit.labs.overthewire.org "cat readme"
```

**What it teaches:** SSH can execute a single remote command non-interactively, which bypasses login scripts that only run for interactive shells.

**Password for bandit19:** `<redacted>`

---

## Level 19 → 20
**Goal:** A setuid binary (`bandit20-do`) in the home directory lets you run one command as `bandit20`.

**Commands used:**
```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

**What it teaches:** setuid binaries run with the permissions of their *owner* rather than the user executing them — a common privilege-escalation mechanism (and a common misconfiguration risk) in Linux.

**Password for bandit20:** `<redacted>`

---

## Level 20 → 21
**Goal:** A setuid binary connects back to a port of your choosing and expects the current password to be sent to it in order to release the next one.

**Commands used:**
```bash
nc -l -p <local_port> &
./suconnect <local_port>
# type the current password into the netcat session
```

**What it teaches:** Setting up your own listener with `nc -l` and having a privileged process connect to it, combining networking with local privilege boundaries.

**Password for bandit21:** `<redacted>`

---

## Level 21 → 22
**Goal:** A cron job configured in `/etc/cron.d/` runs a script periodically as `bandit22` that writes the next password to a file.

**Commands used:**
```bash
cat /etc/cron.d/cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
cat /tmp/<output_file_referenced_in_script>
```

**What it teaches:** Reading scheduled tasks (`cron`) to understand what automated jobs run as which users — a common way privileged information leaks on real systems.

**Password for bandit22:** `<redacted>`

---

## Level 22 → 23
**Goal:** Same idea, but the script computes its output filename dynamically (e.g. via a hash of the username and date), so you must read the script to derive the path.

**Commands used:**
```bash
cat /etc/cron.d/cronjob_bandit23
cat /usr/bin/cronjob_bandit23.sh
# manually compute the same command the script runs, e.g.:
echo "I am user bandit23" | md5sum | cut -d ' ' -f 1
cat /tmp/<computed_filename>
```

**What it teaches:** Understanding a shell script well enough to reproduce its logic by hand, rather than just running it.

**Password for bandit23:** `<redacted>`

---

## Level 23 → 24
**Goal:** A cron job executes *any* script placed in a specific spool directory, running it as `bandit24`.

**Commands used:**
```bash
cat > /var/spool/bandit24/mytask.sh << 'EOF'
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/myoutput_$(whoami)
chmod 777 /tmp/myoutput_$(whoami)
EOF
chmod 777 /var/spool/bandit24/mytask.sh
# wait for cron to run it, then:
cat /tmp/myoutput_bandit23
```

**What it teaches:** A world-writable cron spool directory is a serious misconfiguration — dropping your own script there lets you run arbitrary code as another user.

**Password for bandit24:** `<redacted>`

---

## Level 24 → 25
**Goal:** A daemon listens on a port and expects the current password plus a 4-digit numeric PIN; it must be brute-forced.

**Commands used:**
```bash
for i in $(seq -w 0 9999); do
  echo "<current_password> $i"
done > /tmp/pins.txt
cat /tmp/pins.txt | nc localhost <port>
```

**What it teaches:** Simple brute-force enumeration when the search space is small (10,000 combinations), and piping generated input directly into `nc`.

**Password for bandit25:** `<redacted>`

---

## Level 25 → 26 → 27
**Goal:** This account's login shell is a restricted program rather than bash, only permitting a limited pager-like interface. A private SSH key is provided for `bandit26`. The pager can be tricked into spawning a real shell.

**Commands used / technique:**
```bash
ssh -i bandit26.sshkey -p 2220 bandit26@bandit.labs.overthewire.org
# resize the terminal to very few visible lines so the restricted shell auto-invokes "more"
# inside more, press "v" to open the vi/vim editor
# inside vi, run:
:set shell=/bin/bash
:shell
```

**What it teaches:** Restricted shells only limit what you type directly — if they invoke a full-featured program like a pager or editor internally, that program's own escape hatches (like vi's `:shell`) can break out of the restriction entirely. This is a classic "shell escape" technique still relevant in real-world restricted environments.

**Password for bandit27:** `<redacted>`

---

## Level 27 → 28
**Goal:** Clone a Git repository hosted locally on the server to find the next password.

**Commands used:**
```bash
git clone ssh://bandit27-git@localhost/home/bandit27-git/repo
cat repo/README
```

**What it teaches:** Basic Git usage (`git clone`) and the idea that sensitive data can live in version-controlled repositories, not just plain files.

**Password for bandit28:** `<redacted>`

---

## Level 28 → 29
**Goal:** The current README no longer shows the password — it was committed once and then removed in a later commit. The value still lives in the Git history.

**Commands used:**
```bash
git log -p
```

**What it teaches:** Git history preserves everything that was ever committed; deleting content in a new commit does *not* erase it from the repository's history — an important lesson about accidentally committing secrets.

**Password for bandit29:** `<redacted>`

---

## Level 29 → 30
**Goal:** The password lives on a different Git branch than the default one.

**Commands used:**
```bash
git branch -a
git checkout <branch-name>
cat README
```

**What it teaches:** Branches can hold entirely different file contents, and sensitive data can hide outside of the default/main branch.

**Password for bandit30:** `<redacted>`

---

## Level 30 → 31
**Goal:** The password is referenced by a Git tag rather than being in any tracked file directly.

**Commands used:**
```bash
git tag
git show <tag-name>
```

**What it teaches:** Tags are another Git object type that can carry hidden data (e.g. in annotated tag messages), separate from branches and commits.

**Password for bandit31:** `<redacted>`

---

## Level 31 → 32
**Goal:** The repository is set up so that pushing a specific file (`key.txt`) with specific content unlocks the next password. A `.gitignore` blocks the file from being added normally.

**Commands used:**
```bash
echo "May I come in?" > key.txt
git add -f key.txt
git commit -m "add key"
git push
```

**What it teaches:** `.gitignore` only prevents *accidental* additions — `git add -f` (force) overrides it, showing that ignore rules are not a security boundary.

**Password for bandit32:** `<redacted>`

---

## Level 32 → 33
**Goal:** The login shell is an "UPPERSHELL" that immediately uppercases anything you type before executing it, so ordinary lowercase Linux commands are rejected.

**Commands used / technique:**
```bash
$0
```

**What it teaches:** `$0` is a shell variable holding the path to the current shell binary. Because it's expanded by the shell *before* the uppercasing logic sees it as literal typed text, invoking `$0` spawns a normal, case-sensitive shell — a clever demonstration of how input filtering can be bypassed via shell expansion.

**Result:** Shell access as bandit33 — the final level of the game.

---

## Key concepts and skills learned

Working through Bandit end-to-end covers a broad foundation for Linux and cybersecurity fundamentals: core file navigation and permissions (`ls`, `cat`, `find`, `chmod`, hidden files), text processing and filtering (`grep`, `sort`, `uniq`, `strings`, `tr`, `diff`), encodings and simple ciphers (Base64, ROT13, hexdumps, nested compression formats), networking basics (`nc`, `openssl s_client`, `nmap`, port scanning), SSH key-based authentication, privilege escalation concepts (setuid binaries, cron misconfigurations, world-writable directories), restricted shell escapes, and Git fundamentals including history, branches, tags, and `.gitignore` behavior. Together these mirror many of the building-block skills used in real penetration testing, CTF competitions, and general Linux system administration.

## Disclaimer

This repository is intended purely for educational purposes, to document one person's learning journey through a well-known, intentionally vulnerable training platform designed for this exact purpose. Please play the game yourself at [overthewire.org](https://overthewire.org/wargames/bandit/) to get your own hands-on experience — the value of Bandit is in the problem-solving process, not just the final passwords.
