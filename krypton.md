# OverTheWire: Krypton

Playthrough of the Krypton wargame hosted by OverTheWire

<https://overthewire.org/wargames/krypton>

## Level 0 -> 1

### Given

> Welcome to Krypton! The first level is easy. The following string encodes the password using Base64:
>
> S1JZUFRPTklTR1JFQVQ=
>
> Use this password to log in to krypton.labs.overthewire.org with username krypton1 using SSH on port 2231. You can find the files for other levels in /krypton/

### Solution

- `man base64` to look for a decode flag
- `echo S1JZUFRPTklTR1JFQVQ= | base64 -d` -> `KRYPTONISGREAT`

### Additional

For convenience, we can update our `~/.ssh/config` to include
```ssh
Host krypton
  Hostname krypton.labs.overthewire.org
  Port 2231
```
for using the shorthand `ssh <user>@krypton` in place of `ssh <user>@krypton.labs.overthewire.org -p 2231` for future logins

## Level 1 -> 2

### Given

> The password for level 2 is in the file 'krypton2'. It is 'encrypted' using a simple rotation. It is also in non-standard ciphertext format. When using alpha characters for cipher text it is normal to group the letters into 5 letter clusters, regardless of word boundaries. This helps obfuscate any patterns. This file has kept the plain text word boundaries and carried them to the cipher text. Enjoy!

### Solution

- `cd /krypton/krypton1`
- `less README` reveals that the password for `krypton2` is *"'encrypted' using a simple rotation called ROT13"*
- `which rot13` reveals that `rot13` is not installed on the machine, which is good. We have to do a little work.
- `cat krypton2 | tr A-MN-Z N-ZA-M` -> `LEVEL TWO PASSWORD ROTTEN`
  - `tr A-MN-Z N-ZA-M` maps `A-M` to `N-Z` and `N-Z` to `A-M`, encoding/decoding ROT13

## Level 2 -> 3

### Given

> The password for level 3 is in the file krypton3. It is in 5 letter group ciphertext. It is encrypted with a Caesar Cipher. Without any further information, this cipher text may be difficult to break. You do not have direct access to the key, however you do have access to a program that will encrypt anything you wish to give it using the key. If you think logically, this is completely easy.

> The encrypt binary will look for the keyfile in your current working directory. Therefore, it might be best to create a working direcory in /tmp and in there a link to the keyfile. As the encrypt binary runs setuid krypton3, you also need to give krypton3 access to your working directory.

### Solution

- `cd krypton/krypton2` and look around...
- `mkdir /tmp/holy` (or use your own name or `mktemp -d /tmp/XXX`)
- `cp encrypt krypton3 /tmp/holy/`
- `cd /tmp/holy`
- `echo {a..z} > keyfile.dat` to generate characters a-z and redirect them into `keyfile.dat`
- `./encrypt keyfile.dat` produces a file `ciphertext` with the contents `MNOPQRSTUVWXYZABCDEFGHIJKL`, which shows us that the keyfile is rotated by 12
- `cat ciphertext | tr M-ZA-L A-ZM-Z` -> `ABCDEFGHIJKLMNOPQRSTUVWXYZ` verifies correct usage of `tr` for de-rotation by 12
- `cat krypton3 | tr M-ZA-L A-ZM-Z` -> `CAESARISEASY` looks like a successful de-rotation to me
