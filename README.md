# CTF-writeups

I'm studying CTF, and I write my learning path in this repository.

## OverTheWire Bandit: Levels 0-5

First, I learned how to log in to the Bandit server with `ssh username@bandit.labs.overthewire.org -p 2220`. Then I used `./-` when the file name starts with `-`.

Second, I used the Tab key. When I use it, the file name is completed automatically.

Third, I found out how to look for hidden files. The directory `inhere` had hidden files, so I used `cd inhere` and then `ls -a`. Then I could see all the files in `inhere`.

Next, I learned `file ./*`. With this command, I could find the type of each file and pick out the right file.

Finally, I learned `find`. If I want to look for a file that is a regular file, 1033 bytes, and not executable, I can use this command: `find . -type f -size 1033c ! -executable`. Therefore, I could find the correct file among many files.


## OverTheWire Bandit: Levels 6-13

### Level 6
Goal: Find a file owned by user `bandit7`, group `bandit6`, and 33 bytes in size.

```
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

Why: I searched from `/` because the file is somewhere on the server. `2>/dev/null` hides the "Permission denied" errors.

### Level 7
Goal: Find the password next to a word in a very large file.

```
grep millionth data.txt
```

Why: `cat` shows too much text. `grep` shows only the lines that have the word.

### Level 8
Goal: Find the only line that appears one time.

```
sort data.txt | uniq -u
```

Why: `uniq` only checks lines next to each other, so I use `sort` first. `-u` shows only the lines that appear once.

### Level 9
Goal: Find a readable string that starts with several `=` characters.

```
strings data.txt | grep "=="
```

Why: `grep` said "Binary file matches" because the file has unreadable data. `strings` takes out only the readable text.

### Level 10
Goal: Read a Base64 file.

```
base64 -d data.txt
```

Why: The whole file is Base64. `-d` means decode.

### Level 11
Goal: Read a file encoded with ROT13.

```
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

Why: ROT13 moves each letter 13 places. `tr` replaces the letters, so the text becomes readable.

### Level 12
Goal: Decode a hexdump file that was compressed many times.

```
mktemp -d
cp ~/data.txt .
xxd -r data.txt > data
file data
```

Then I repeated these steps:
1. Check the newest file with `ls -lt`.
2. Check its type with `file`.
3. Decompress it: `gzip -d` (after renaming to `.gz`), `bzip2 -d` (after renaming to `.bz2`), or `tar xf`.

Why: I made a working directory in `/tmp` because I can't create files in my home directory. `xxd -r` changes hex back to binary.

Mistake: `tar xf` doesn't delete the original tar file. It makes a new file, so I checked the newest file every time.

### Level 13
Goal: Log in as bandit14 with a private SSH key.

scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private .
chmod 600 sshkey.private
ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220

Why: I downloaded the key with `scp`. SSH ignores a private key that other users can read, so I used `chmod 600`.
The server checks the key, so I didn't need a password.
