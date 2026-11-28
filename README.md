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

### Level 14
Goal: Send the current level's password to a service listening on port 30000 on localhost.

```bash
cat /etc/bandit_pass/bandit14
nc localhost 30000
```

Why: `nc` connects to a TCP service. I entered the current password and received the next password. Normal TCP communication is not encrypted.

### Level 15
Goal: Send the current password to a service using an encrypted TLS connection.

```bash
openssl s_client -connect localhost:30001 -quiet
```

Why: `openssl s_client` establishes a TLS connection. Unlike normal `nc` communication, TLS encrypts the data. I entered the password and received the next password.

### Level 16
Goal: Find a TLS service listening on a port between 31000 and 32000 that returns the credentials for the next level.

```bash
nmap -p 31000-32000 localhost
openssl s_client -connect localhost:<PORT> -quiet
```

Why: I used `nmap` to find open ports and tested them one by one using OpenSSL. I found a service that returned an SSH private key.

I saved the SSH private key on my Mac:

```bash
mkdir -p ~/.ssh/bandit
chmod 700 ~/.ssh/bandit
nano ~/.ssh/bandit/bandit17_key
chmod 600 ~/.ssh/bandit/bandit17_key
ssh -i ~/.ssh/bandit/bandit17_key -p 2220 bandit17@bandit.labs.overthewire.org
```

Why: I needed the SSH private key to log in as bandit17. I used `chmod 600` to protect the key. I connected from my Mac because SSH connections from localhost were blocked on the Bandit server.

**What I Learned:** I learned how to use Netcat for TCP communication, OpenSSL for TLS communication, and Nmap to find open ports. I also practiced SSH private key authentication and file permissions.

## OverTheWire Bandit — Levels 17–23

### Level 17
Goal: Find the password that is different between two files.

```bash
diff passwords.old passwords.new
```

Why: `diff` compares two files line by line and shows the differences. I found the new password in `passwords.new`.

### Level 18
Goal: Read the password from `readme`, even though the SSH connection closes immediately after login.

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```

Why: I couldn't stay logged in because the server disconnected me immediately. By adding `"cat readme"` to the SSH command, I could execute the command remotely without opening an interactive shell.

### Level 19
Goal: Use a SetUID executable to read the next level's password.

```bash
ls -l
./bandit20-do cat /etc/bandit_pass/bandit20
```

Why: I found an executable called `bandit20-do`. The `s` in the file permissions indicates SetUID. This allows the program to run with the file owner's privileges, so I could read the password for bandit20.

### Level 20
Goal: Send the current password to a SetUID program using a local TCP connection.

Terminal 1:

```bash
nc -l 12345
```

Terminal 2:

```bash
./suconnect 12345
```

Then I entered the current password in Terminal 1.

Why: `nc -l` listens for incoming connections on a port. The `suconnect` program connects to that port and checks the password. When the password matches, it sends back the next password.

I learned how two programs communicate using TCP and a port number.

### Level 21
Goal: Find the password generated by a scheduled Cron Job.

```bash
ls /etc/cron.d
cat /etc/cron.d/cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
```

Then I checked the file path written in the script:

```bash
cat /tmp/<generated_file>
```

Why: `cron` runs commands automatically at scheduled times. I checked the cron configuration and found the script that writes the next password into a temporary file.

### Level 22
Goal: Find the password file generated by a Cron Job using an MD5 hash.

```bash
cat /etc/cron.d/cronjob_bandit23
cat /usr/bin/cronjob_bandit23.sh
```

I found that the script generates a filename using an MD5 hash.

```bash
myname=bandit23
mytarget=$(echo "I am user $myname" | md5sum | cut -d ' ' -f 1)
cat /tmp/$mytarget
```

Why: `md5sum` calculates a hash value, and `cut` extracts the hash from the command output. I reproduced the filename used by the Cron Job and read the password.

I learned how to read shell scripts and understand how variables and command substitution work.

### Level 23
Goal: Create a shell script that will be executed automatically by a Cron Job running as bandit24.

First, I checked the Cron Job:

```bash
cat /etc/cron.d/cronjob_bandit24
cat /usr/bin/cronjob_bandit24.sh
```

Then I created a temporary directory:

```bash
workdir=$(mktemp -d)
chmod 755 "$workdir"
nano "$workdir/get_password.sh"
```

I wrote the following script, using my actual temporary directory path:

```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/<my_directory>/password.txt
chmod 644 /tmp/<my_directory>/password.txt
```

Then I made the script executable and copied it to the directory used by the Cron Job:

```bash
chmod 755 "$workdir/get_password.sh"
cp "$workdir/get_password.sh" /var/spool/bandit24/foo/
```

After the Cron Job executed the script, I read the output:

```bash
cat "$workdir/password.txt"
```

Why: The Cron Job automatically executes scripts from a specific directory as bandit24. I created a shell script that reads the password and writes it to a temporary file.

I also needed to set appropriate permissions so the Cron Job could access my script and I could read the output.

I learned how Cron Jobs execute shell scripts automatically and why file permissions are important.

---

### What I Learned

In Levels 17–23, I practiced several important Linux and cybersecurity concepts:

- `diff`: Comparing files
- `ssh`: Executing remote commands
- SetUID: Running programs with the file owner's privileges
- `nc`: Listening for TCP connections
- `cron`: Running scheduled tasks
- `md5sum`: Generating hashes
- Shell scripting: Automating commands
- `chmod`: Managing file permissions

The most interesting part was learning how Linux permissions and automated scripts work together.

### Level 24

Goal: Find the next password by trying all 4-digit PINs from 0000 to 9999.

First, I created a temporary directory and used `nano` to make `solve.sh`.

```bash
workdir=$(mktemp -d)
cd "$workdir"
nano solve.sh
```

Then, I wrote this script:

```bash
#!/bin/bash

password=$(cat /etc/bandit_pass/bandit24)

for i in {0..9999}; do
    printf "%s %04d\n" "$password" "$i"
done
```

After that, I ran the script and sent the output to port 30002 using `nc`. I saved the results to `result.txt` because there were too many messages in my terminal.

```bash
bash solve.sh | nc localhost 30002 > result.txt
```

Finally, I used `grep` to find the next password.

```bash
grep -v "Wrong" result.txt
```

Why: The `for` loop generates all 10,000 PINs, and `printf "%04d"` makes each PIN four digits. `nc` sends them to the server, and `>` saves the responses to a file. `grep -v` removes lines containing "Wrong", so I could find the correct password.

