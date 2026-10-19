# CTF-writeups

I'm studying CTF, and I write my learning path in this repository.

## OverTheWire Bandit: Levels 0-5

First, I learned how to log in to the Bandit server with `ssh username@bandit.labs.overthewire.org -p 2220`. Then I used `./-` when the file name starts with `-`.

Second, I used the Tab key. When I use it, the file name is completed automatically.

Third, I found out how to look for hidden files. The directory `inhere` had hidden files, so I used `cd inhere` and then `ls -a`. Then I could see all the files in `inhere`.

Next, I learned `file ./*`. With this command, I could find the type of each file and pick out the right file.

Finally, I learned `find`. If I want to look for a file that is a regular file, 1033 bytes, and not executable, I can use this command: `find . -type f -size 1033c ! -executable`. Therefore, I could find the correct file among many files.
