# Misc tasks 

## Uncrackable zip
Task: \
Take a zip of my uncrackable drink\

An image showing the files and process of creating the zip is shown;
* flag.txt
* hint.txt
```
cat hint.txt
This is a zip file that you will never be able to crack, the password has 39 characters. Go ahead and use johns/hashcats etc., if you have 1000 years to spare :)

7z a -p challenge.zip *

<7zip version info>
<...output..>
Files read from dis: 2
Archive size: 484 bytes (1 KiB)
Everything is Ok
```
Taskfile is challenge.zip

The image gives us the plaintext for hint.txt so we have an ecnrypted zip with known plaintext content.\
Recreating the hint.txt file, and creating a new plaintext.zip file with 7z without a password to use to solve.\
Googling for variations of decrypting a zip-file with known content says it's possible\
`pkcrack` looked like it was a handy thing, but did not work for this challenge [https://github.com/keyunluo/pkcrack]  \
`bkcrack` to the rescut! [https://github.com/kimci86/bkcrack]  \
Downloading the tarball and off we went!
````
[root@686b9ce9d4ce Misc]# bkcrack/bkcrack-1.3.1-Linux/bkcrack -C uncrackable-challenge.zip -c hint.txt -P plaintext.zip -p hint.txt
bkcrack 1.3.1 - 2021-08-16
[22:53:02] Z reduction using 119 bytes of known plaintext
100.0 % (119 / 119)
[22:53:02] Attack on 73138 Z values at index 7
Keys: d3caefe5 21950626 24e09e13
74.2 % (54263 / 73138)
[22:53:25] Keys
d3caefe5 21950626 24e09e13
[root@686b9ce9d4ce Misc]# bkcrack/bkcrack-1.3.1-Linux/bkcrack -C uncrackable-challenge.zip -k d3caefe5 21950626 24e09e13 -U secrets_with_new_password.zip easy
bkcrack 1.3.1 - 2021-08-16
[23:05:48] Writing unlocked archive secrets_with_new_password.zip with password "easy"
100.0 % (2 / 2)
Wrote unlocked archive.
[root@686b9ce9d4ce Misc]# unzip secrets_with_new_password.zip
Archive:  secrets_with_new_password.zip
[secrets_with_new_password.zip] flag.txt password:
 extracting: flag.txt
replace hint.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: n
[root@686b9ce9d4ce Misc]# cat flag.txt
EPT{d1d_y0u_gu3$$_th3_p4$$w0rd_0r_pl41nt3xt_cr4ck_1t?}
````