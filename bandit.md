# OverTheWire: Bandit

Playthrough of the Bandit wargame hosted by OverTheWire

<https://overthewire.org/wargames/bandit>

## Game Notes

- From remote machine:
>  This machine might hold several wargames.
>  If you are playing "somegame", then:
>
>    * USERNAMES are somegame0, somegame1, ...
>    * Most LEVELS are stored in /somegame/.
>    * PASSWORDS for each level are stored in /etc/somegame\_pass/.
>
>  Write-access to homedirectories is disabled. It is advised to create a
>  working directory with a hard-to-guess name in /tmp/.  You can use the
>  command "mktemp -d" in order to generate a random and hard to guess
>  directory in /tmp/.  Read-access to both /tmp/ is disabled and to /proc
>  restricted so that users cannot snoop on eachother. Files and directories
>  with easily guessable or short names will be periodically deleted! The /tmp
>  directory is regularly wiped.

- Tools installed by default:
  - GEF (GDB Enhanced Features): <https://github.com/hugsy/gef>
  - pwndbg (GDB extension): <https://github.com/pwndbg/pwndbg>
  - gdbinit: <https://github.com/gdbinit/Gdbinit>
  - pwntools (CTF framework and exploit development library): <https://github.com/Gallopsled/pwntools>
  - radare (Reversing toolkit): <https://www.radare.org/n/>

## Level 0

- Open terminal and SSH into game server with username `bandit0`: `ssh bandit0@bandit.labs.overthewire.org -p 2220`
  - Enter password `bandit0`

## Level 0 --> 1

- `cat readme` reveals `bandit1` user password: `ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If`

## Level 1 --> 2

- Use password obtained from last level to SSH into remote as `bandit1`: `ssh bandit1@bandit.labs.overthewire.org -p 2220`
- There is a file called "-" which contains the next level's password in plaintext
  - `ls` -> `-`
  - I tried using several variations of `cat` to read the file, but they all failed:
    - `cat -`; `cat -- -`; `cat "-"`; `cat '-'`; `cat '\-'`: They all appear to stall
  - I try reading the man page, `man cat`, to no avail
  - I then begin skimming the recommended reading...
    - <https://linux.die.net/abs-guide/special-chars.html>
  - Solution: I then get the idea to try `cat ./-`, which works
    - The password for the next level is `263JGJPfgU6LtdEvgfWU1XP5yac29mFx`

## Level 2 --> 3

- `ssh bandit2@bandit.labs.overthewire.org -p 2220` with password obtained from previous level
- `ls` reveals a single file named `spaces in this filename`
- `cat "space in this filename"` -> `MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx`

## Level 3 --> 4

- Given: *The password for the next level is stored in a hidden file in the inhere directory.*
- `ssh bandit3@bandit.labs.overthewire.org -p 2220` with password obtained from previous level
- `ls` -> `inhere`
- `ls inhere` -> empty
- `ls -a inhere` reveals `...Hiding-From-You`
- `cat inhere/...Hiding-From-You` -> `2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ`

## Level 4 --> 5

- Given: *The password for the next level is stored in the only human-readable file in the inhere directory. Tip: if your terminal is messed up, try the “reset” command.*
- `ssh bandit4@bandit.labs.overthewire.org -p 2220` with password obtained from previous level
- `ls -l inhere` reveals several files with identical header information (from `-l`
- Reading `help for` and testing what the `file` command does, I worked out this loop: `for filename in $(find inhere); do file $filename; done`, which reveals `inhere/-file07: ASCII text` as opposed to everything else which said `data` instead of `ASCII text`.
  - `cat inhere/-file07` reveals password for next level: `4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw`

### Follow-up

- I have no idea why I used the `for` loop to enumerate files to run `file` on. Instead, it was as trivial as `file inhere/*`

## Level 5 --> 6

- Given:
> The password for the next level is stored in a file somewhere under the inhere directory and has all of the following properties:
>
>     human-readable
>     1033 bytes in size
>     not executable
`find inhere -size 1033c -print0 | xargs -0 file` reveals `inhere/maybehere07/.file2: ASCII text, with very long lines (1000)`
  - Notes on `find`: `c` in `1033c` means bytes; `-print0` should come at the end so it only prints after processing; `print0` is specified with `0` (and therefore `xargs` is given `-0`) because we want to more safely process files with spaces and certain other special characters on the printout. It was not specifically required in this case.
  - This means we have found only a single 1033-byte human-readable file.
  - Checking for execution privileges, I alter the pipe to use `ls -l`: `find inhere -size 1033c -print0 | xargs -0 ls -l`. We verify that there are no execution privileges.
  - We've verified that this is the only file matching the target properties
  - `find inhere -size 1033c -print0 | xargs -0 cat` reveals the password for the next level: `HWasnPhtq9AVKe0dmk45nxy20cvUa6EG`

### Follow-up

- The `xargs` portion for manually checking for non-executable privileges can be removed in favor of adding to the `find` command `\! -executable` for "not executable":
  - `find inhere \! -executable -size 1033c -print0`

## Level 6 --> 7

- Given:
> The password for the next level is stored somewhere on the server and has all of the following properties:
>
>    owned by user bandit7
>    owned by group bandit6
>    33 bytes in size
- SSH in as in previous procedures
- `find / -user bandit7 -group bandit6 -size 33c` gets lots of "Permission denied", but finds `/var/lib/dpkg/info/bandit7.password`
  - The "Permission denied" errors can be filtered out trivially by redirecting stdout to /dev/null: `2>/dev/null`, for full command `find / -user bandit7 -group bandit6 -size 33c 2>/dev/null`
  - `find / -user bandit7 -group bandit6 -size 33c 2>/dev/null | xargs cat` reveals the next level's password: `morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj`

---

## Level 7 --> 8

- Given: *The password for the next level is stored in the file data.txt next to the word millionth*
- SSH in as in previous procedures
- `cat data.txt | grep "millionth"` discovers the line `millionth       dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc`, which contains our password.

## Level 8 --> 9

- Given: *The password for the next level is stored in the file data.txt and is the only line of text that occurs only once*
- SSH in as in previous procedures
- `cat data.txt | sort | uniq -u` -> `4CKMh1JI91bUIZZPXDqGanal4xvAg0JM`
  - `uniq` counts the occurrence of repeated adjacent lines, which is why we sort first. `-u` therefore prints out lines that only occur once in the entire file.

### Follow-Up

`cat data.txt | sort | uniq -u` -> can be simplified to `sort data.txt | uniq -u`

## Level 9 --> 10

- Given: *The password for the next level is stored in the file data.txt in one of the few human-readable strings, preceded by several ‘=’ characters.*
- SSH in as in previous procedures
- `strings data.txt | grep ==` the output tells us the next level's password is `FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey`

## Level 10 --> 11

- Given: *The password for the next level is stored in the file data.txt, which contains base64 encoded data*
- SSH in as in previous procedures
- `cat data.txt | base64 -d` -> `The password is dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr`

## Level 11 --> 12

- Given: *The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions*
  - Help: https://en.wikipedia.org/wiki/ROT13
- SSH in as in previous procedures
- `cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'` -> `The password is 7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4`
  - The `'A-Za-z'` array expands to `ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`
  - The `'N-ZA-Mn-za-m'` array expands to `NOPQRSTUVWXYZABCDEFGHIJKLMnopqrstuvwxyzabcdefghijklm`
  - A one-to-one mapping occurs from each element in the first array to each element of the second array
  - This hard-encodes and applies ROT13, which both encodes plaintext and decodes ROT13 text since ROT13 is symmetric

## Level 12 --> 13

- Given: *The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work. Use mkdir with a hard to guess directory name. Or better, use the command “mktemp -d”. Then copy the datafile using cp, and rename it using mv (read the manpages!)*
- SSH in as in previous procedures
- `mktemp -d /tmp/XXX` generates a random three-character directory name under `/tmp`. Copy `data.txt` into it. This is our working directory.
- Use the decompression utilities to decompress multiple different compressed file formats:
  - gzip: `gunzip -c file > fileuncomp`
    - `-c` writes to standard output and preserves original input file
  - `tar`: `tar xOf file`
    - `tar` doesn't require `-` for at least some of its options, which is probably because the archive names are specified after specific flags (in our case, `f`)
    - `x` extracts tar archive
    - `O` writes to standard output and preserves original input file
    - `f` specifies input file
  - `bzip2`: `bunzip2 -c file`
    - `-c` writes to standard output and preserves original input file
- The password for the next level is `FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn`

---

## Level 13 --> 14

- Given: *The password for the next level is stored in /etc/bandit_pass/bandit14 and can only be read by user bandit14. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. Note: localhost is a hostname that refers to the machine you are working on*
- SSH in as in previous procedures
- `ssh -i sshkey.private bandit14@localhost -p 2220`
- Now, as user `bandit14`, we can retrieve the next leve's password: `cat /etc/bandit_pass/bandit14` -> `MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS`

## Level 14 --> 15

- Given: *The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.*
- SSH in as in previous procedures
- `nmap localhost`
- `echo MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS | nc localhost 30000` -> `8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo`

## Level 15 --> 16

- Given: *The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL/TLS encryption.*
- SSH in as in previous procedures
- `openssl s_client localhost:30001` and paste password -> `kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx`

## Level 16 --> 17

- Given: *The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.*
- Help: `Getting “DONE”, “RENEGOTIATING” or “KEYUPDATE”? Read the “CONNECTED COMMANDS” section in the manpage.`
- SSH in as in previous procedures
- `nmap -sV localhost -p 31000-32000` takes a long while to reveal two ports hosting ssl services: `31518/tcp open  ssl/echo` and `31790/tcp open  ssl/unknown`
  - `-sV`: *Probe open ports to determine service/version info*
  - Port 31790 also happens to spit out a bunch of apparently malformed requests to enter the correct current password
  - Improvement on this command:
- `openssl s_client -connect localhost:31790`
  - Pasting the current level's password receives the `KEYUPDATE` message, while any other output seems to just receive `Wrong! Please enter the correct current password.`
  - Reading the `CONNECTED COMMANDS` section in `man openssl-s_client` (as suggested by the challenge), it mentions the use of `-ign_eof`
    - Inserting `-ign_eof` works, and we receive an RSA private key (almost certainly the "credentials" for the next level):
```
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----
```
  - Strangely, as `bandit15` in the previous level, I didn't have to specify `-connect` or `-ign_eof`. There must be some kind of user configuration or difference in the `openssl` version I'm using?
    - As `bandit16`: `openssl version` -> `OpenSSL 3.0.13 30 Jan 2024 (Library: OpenSSL 3.0.13 30 Jan 2024)`
    - As `bandit15`: `openssl version` -> `OpenSSL 3.0.13 30 Jan 2024 (Library: OpenSSL 3.0.13 30 Jan 2024)`
    - I guess they're the same... So perhaps a user configuration difference?
    - ... No idea
  - Store this private key in a file named `bandit17_private_key`

## Level 17 --> 18

- Given: *There are 2 files in the homedirectory: passwords.old and passwords.new. The password for the next level is in passwords.new and is the only line that has been changed between passwords.old and passwords.new*
- SSH in using the identity (private key) file created during the last level: `ssh -i bandit17_private_key bandit17@bandit.labs.overthewire.org -p 2220`
  - Uh oh! The identity file's permissions are too permissive:
    - `chmod go-rwx bandit17_private_key` to remove the `rwx` bits from `group` and `other` users, which should leave the user with the default read/write permissions
      - A better way to do this is `chmod 600 bandit17_private_key`
  - `diff passwords.old passwords.new` reveals `x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO`

## Level 18 --> 19

- Given: *The password for the next level is stored in a file readme in the homedirectory. Unfortunately, someone has modified .bashrc to log you out when you log in with SSH.*
- `ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme` -> `cGWpMaKXVwDUNgPAVJbWYuGHVn9zl3j8`

## Level 19 --> 20

- Given: *To gain access to the next level, you should use the setuid binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.*
- SSH in in the usual way
- `./bandit20-do cat /etc/bandit_pass/bandit20` -> `0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO`

## Level 20 --> 21

- Given: *There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).*
- SSH as usual
- In one Tmux pane: `nc -l 30004`. In another Tmux pane: `./suconnect 30004`
  - Paste the current level's password into netcat's end of the connection -> `EeoULMCra2q0dSkYj561DX7s1CpBuOBt`

## Level 21 --> 22

- Given: *A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.*
- Info: Read `man 5 crontab`
- SSH in as usual
- `ls -al /etc/cron.d`
- Ran `cat` on various files just to see what they all look quickly
- The primary `cronjob_<username>` files are running scripts from `/usr/bin/` such as `/usr/bin/cronjob_<username>.sh`
- `cat /usr/bin/cronjob_bandit22.sh` (the only readable cron script in this directory)
```bash
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```
  - `cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv` -> `tRae0UfB9v0UzbCdn9cY0gQnds9GF58Q`

## Level 22 --> 23

- Given: *A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.*
- SSH in as usual
- `cat /etc/cron.d/cronjob_bandit23`
- `cat /usr/bin/cronjob_bandit23.sh`
- Hardcoding `bandit23` in place of `$(whoami)`: `cat /tmp/$(echo I am user bandit23 | md5sum | cut -d ' ' -f 1)` -> `0Zf11ioIjMVN551jX3CmStKLYqjk54Ga`

## Level 23 --> 24

- NOTE: This is one of those that I think I will be able to provide a much better solution for the next time around.
- NOTE: Lots of copy-paste in documenting this one. There's bound to be some inaccuracies.
- Given: *A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.*
- SSH in as usual
- `bandit24`'s scripts directory (`/var/spool/bandit24/foo/`; containing scripts to be executed and deleted by their cronjob script) appears misconfigured to allow non-owner and non-group write and execute permissions (determined via `ls -l /var/spool/bandit24/`). We will exploit this.
  - `mktemp -d /tmp/XXX` (create a working directory for ourselves)
  - Create a script (`/tmp/XXX/b23`) that we will write (copy) to `bandit24`'s scripts directory, which we know we can do because it is writable
```bash
#!/bin/bash
echo $(find) > /tmp/XXX/b23out.txt
```
  - `cp /tmp/XXX/b23 /var/spool/bandit24/foo/`
  - Wait for our script to be executed by `bandit24`'s cronjob script
  - Once executed (and automatically deleted), our script lists all other files in the directory that were just present when it was executed
  - NOTE: I wonder if we can use `netcat` or something to see if we can run commands as `bandit24`
  - Let's see if we can find any interesting text from the files we were able to find: `for file in $(cat /tmp/0Zr.txt); do echo ---------------$file; cat /var/spool/bandit24/foo/$file; done`
    - I spot this rather interesting line toward the end of the output: `echo | cat /etc/bandit_pass/bandit24 >> /tmp/pw` coming from `/var/spool/bandit24/foo/102983scfec/myscript.sh`
    - `cat /var/spool/bandit24/foo/102983scfec/myscript.sh` -> `gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8`

## Level 24 --> 25

- NOTE: Explore different solutions
- Given: *A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.*
- SSH in as usual
- `netcat -N localhost 30002 < <(for i in {0..9999}; do printf "gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8 %.4d\n" $i; done)` -> `iCi86ttT4KSNe1armKiwbQNmB3YJP3q4`

## Level 25 --> 27

- Given: *Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not /bin/bash, but something else. Find out what it is, how it works and how to break out of it.*
- Copy `bandit26.sshkey` from `bandit25`'s home directory to our CWD: `ssh bandit25@bandit.labs.overthewire.org -p 2220 "cat bandit26.sshkey" > bandit26.sshkey`
- Remove group and other users' rwx permissions for ssh key file so we can pass it as an identity file to SSH: `chmod go-rwx bandit26.sshkey`
- `ssh -i bandit26.sshkey bandit26@bandit.labs.overthewire.org -p 2220`
- Since we apparently can't use the shell of `bandit26` for anything useful, we can log back into `bandit25` and `grep bandit26 /etc/passwd`, which gets us `bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext`, which reveals the shell program for `bandit26`: `/usr/bin/showtext`
- `cat /usr/bin/showtext` -> `...; exec more ~/text.txt; ...;`, revealing that `more` is actually our shell program
- Knowing that `more` stays open so long as there is input to display that has not yet been displayed, and knowing that `more` gives us some amount of interactivity, we can modify the viewport of our terminal such that `more` isn't able to just display `~/text.txt` and immediately quit, but must hold open.
- While `more` is held open, we can type `v` to open the file in the default editor, and then resize our viewport for a full view of the editor. We can now use our editor to open `/etc/bandit_pass/bandit26` to obtain the password for `bandit26`, for whatever it's worth (it's not required).
  - NOTE: We can't just use `more`'s `!command` subshell syntax because the subshell would just be another instance of the currently running shell, `showtext`.
- For the default editor, `vi`, we can view and modify our shell. `:set shell` will confirm that our current shell is `/usr/bin/showtext`. We can change this via `:set shell=bash`. We can then run `:!exec bash` to break out of our shell and replace it with a `bash` session.
- `./bandit27-do cat /etc/bandit_pass/bandit27` -> `upsNCc7vzaRDx6oZC6GiR6ERwe1MowGB`

## Level 27 --> 28

- Given: *There is a git repository at ssh://bandit27-git@localhost/home/bandit27-git/repo via the port 2220. The password for the user bandit27-git is the same as for the user bandit27. Clone the repository and find the password for the next level.*
- `git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo bandit27-git-repo`
- `cat bandit27-git-repo/README` -> `The password to the next level is: Yz9IpL0sBcCeuG7m9uQFt8ZNpS4HZRcN`

## Level 28 --> 29

- Given: *There is a git repository at ssh://bandit28-git@localhost/home/bandit28-git/repo via the port 2220. The password for the user bandit28-git is the same as for the user bandit28. Clone the repository and find the password for the next level.*
- `git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo bandit28-git-repo`
- `cat bandit28-git-repo/README.md` contains `password: xxxxxxxxxx` pertaining to `bandit29`
- `cd bandit28-git-repo; git log --oneline` reveals an entry of `674690a (HEAD -> master, origin/master, origin/HEAD) fix info leak`
- Assuming that `674690a` hid the password, we can do `git show 674690a` -> `password: 4pT1t5DENaYuqnqvadYs1oE4QLCdjmJ7`

## Level 29 --> 30

- Given: `There is a git repository at ssh://bandit29-git@localhost/home/bandit29-git/repo via the port 2220. The password for the user bandit29-git is the same as for the user bandit29. Clone the repository and find the password for the next level.`
- `git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo bandit29-git-repo`
- `cat bandit29-git-repo/README.md` contains `password: <no passwords in production!>` pertaining to `bandit30`
- `git branch -a` reveals several branches we could switch to and investigate
- `git switch dev` and `cat README.md` reveals the password for `bandit30`: `qp30ex3VLz5MDG1n91YowTv4Q8l7CDZL`
