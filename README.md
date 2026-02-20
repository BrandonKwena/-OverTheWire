# OverTheWire: Bandit — Levels 1 to 12

> **Bandit** is a beginner-friendly wargame from [OverTheWire](https://overthewire.org/wargames/bandit/) designed to teach the fundamentals of Linux, the command line, and cybersecurity concepts.

---

## How to Connect

```bash
ssh bandit<level>@bandit.labs.overthewire.org -p 2220
```

Replace `<level>` with the level number (e.g., `bandit0`, `bandit1`, etc.)

---

## Level 0 → Level 1

**Password:** `ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If`

**Connection:**
```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

---

## Level 1 → Level 2

**Goal:** The password is stored in a file called `-` in the home directory.

**Problem:** A filename starting with `-` is interpreted as a flag by most commands.

**Solution:** Reference the file using its full path `./` prefix.

```bash
cat ./-
```

**Password:** `263JGJPfgU6LtdEvgfWU1XP5yac29mFx`

**Concept learned:** Files with special characters like `-` can't be opened normally. Prefixing with `./` tells the shell it's a filename, not a flag.

---

## Level 2 → Level 3

**Goal:** The password is stored in a file with spaces in its filename.

**Solution:** Quote the filename or escape the spaces.

```bash
cat "-- spaces in this filename--"
# or
cat --\ spaces\ in\ this\ filename--
```

**Password:** `MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx`

---

## Level 3 → Level 4

**Goal:** The password is stored in a hidden file inside the `inhere` directory.

**Solution:** Use `ls -a` to reveal hidden files (those starting with `.`), then `cat` the file.

```bash
cd inhere
ls -a
cat ...Hiding-From-You
```

**Password:** `2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ`

**Concept learned:** Files prefixed with `.` are hidden from a standard `ls`. The `-a` flag shows all files including hidden ones.

---

## Level 4 → Level 5

**Goal:** The password is in the only human-readable file inside `inhere`, among 10 files named `-file00` through `-file09`.

**Solution:** Use the `file` command to check all file types at once, then read the ASCII text one.

```bash
cd inhere
file ./-file*
cat -- -file07
```

**Output of `file` command:**
```
./-file07: ASCII text   ← this is the one
```

**Password:** `4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw`

**Concept learned:** The `file` command identifies what type of data a file contains. This is useful when extensions are missing or misleading.

---

## Level 5 → Level 6

**Goal:** Find a file somewhere under `inhere` that is:
- Human-readable
- 1033 bytes in size
- Not executable

**Solution:** Use `find` with multiple filters.

```bash
find . -type f -size 1033c -not -executable -exec file {} + | grep ASCII
cat ./maybehere07/.file2
```

**Password:** `HWasnPhtq9AVKe0dmk45nxy20cvUa6EG`

**Concept learned:** `find` is a powerful command for locating files by properties like size (`-size`), type (`-type`), and permissions (`-not -executable`).

---

## Level 6 → Level 7

**Goal:** The password is stored somewhere on the server and is:
- Owned by user `bandit7`
- Owned by group `bandit6`
- 33 bytes in size

**Solution:** Search the entire filesystem with `find`, and suppress permission errors using `2>/dev/null`.

```bash
find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```

**Password:** `morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj`

**Concept learned:**
- `2>/dev/null` redirects **stderr** (error messages) to `/dev/null`, a special Linux "black hole" that discards everything sent to it. This keeps your output clean by hiding permission-denied errors.

---

## Level 7 → Level 8

**Goal:** The password is next to the word `millionth` in `data.txt`.

**Solution:** Use `grep` to search for the keyword.

```bash
cat data.txt | grep millionth
```

**Password:** `dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc`

**Concept learned:** `grep` is used to search for patterns inside files. Piping (`|`) lets you pass output from one command into another.

---

## Level 8 → Level 9

**Goal:** The password is the only line that appears exactly once in `data.txt`.

**Solution:** Sort the file first (so duplicates are adjacent), then use `uniq -u` to print only unique lines.

```bash
sort data.txt | uniq -u
```

**Password:** `4CKMh1JI91bUIZZPXDqGanal4xvAg0JM`

**Concept learned:** `uniq -u` only works correctly on sorted input — duplicates must be adjacent for it to detect them. Always `sort` first.

---

## Level 9 → Level 10

**Goal:** The password is one of the few human-readable strings in `data.txt`, preceded by several `=` characters.

**Solution:** Use `strings` to extract readable text from a binary file, then filter with `grep`.

```bash
strings data.txt | grep "==="
```

**Password:** `FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey`

**Concept learned:** `strings` extracts printable ASCII sequences from binary files — useful when a file contains mostly non-readable data.

---

## Level 10 → Level 11

**Goal:** `data.txt` contains Base64-encoded data.

**Solution:** Decode it using the `base64` command with the `-d` flag.

```bash
cat data.txt | base64 -d
```

**Password:** `dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr`

**Concept learned:** Base64 is a common encoding scheme. Encoded strings often end with `=` padding and only use characters `A-Z`, `a-z`, `0-9`, `+`, `/`, `=`.

---

## Level 11 → Level 12

**Goal:** All letters in `data.txt` have been rotated by 13 positions (ROT13).

**Solution:** Use `tr` (translate) to reverse the ROT13 cipher.

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**Password:** `7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4`

**Concept learned:** ROT13 is a simple substitution cipher — each letter is shifted 13 positions in the alphabet. Since the alphabet has 26 letters, applying ROT13 twice returns the original text. `tr` maps one character set to another.

---

## Level 12 → Level 13

**Goal:** `data.txt` is a hex dump of a file that has been repeatedly compressed.

**Approach:**
1. Create a working directory in `/tmp`
2. Copy the file there
3. Reverse the hex dump using `xxd -r`
4. Repeatedly decompress (using `file` to identify the compression type each time)

```bash
mktemp -d          # create a temp directory
cp data.txt /tmp/<your_dir>/
cd /tmp/<your_dir>/
xxd -r data.txt > data.bin
file data.bin      # check compression type, then decompress accordingly
```

Compression tools you may need: `gzip`, `bzip2`, `tar`

> *(Password to be added upon completion)*

---

## Key Commands Summary

| Command | Purpose |
|---|---|
| `cat ./-` | Read a file named `-` |
| `ls -a` | Show hidden files |
| `file <filename>` | Identify file type |
| `find` | Search files by properties |
| `2>/dev/null` | Discard error messages |
| `grep` | Search for patterns |
| `sort \| uniq -u` | Find unique lines |
| `strings` | Extract readable text from binaries |
| `base64 -d` | Decode Base64 |
| `tr 'A-Za-z' 'N-ZA-Mn-za-m'` | Decode ROT13 |
| `xxd -r` | Reverse a hex dump |

---

*Writeups by [Your Name] — OverTheWire Bandit Wargame*
