# Forensic challenges

## sysadmin1

Task:
```  
Our sysadmin has disappeared and we can no longer access our sysadmin web portal.

We managed to get an image of his harddrive, but we don't know which bitlocker password corresponds to his drive. Can you help us unlock it?
```

Provided is a file `sysadmin.zip` containing a .vhd bitlocker image file and a list of passwords.\
With no experience with bitlocker, googling around some suggest some kali tools to get some more information.

Firstly using the kali tool `bitlocker2john` to extract some hashes from the .vhd file\
Then using `john` to crack the hashes, with the passwords provided in the file bitlocker-passwords.txt to get the correct one.

```
┌──(root💀cf89b1317ce8)-[/Equinor-CTF/Forensic]
└─# bitlocker2john -i sysadmin.vhd
Encrypted device sysadmin.vhd opened, size 250MB

Signature found at 0x10003
Version: 8
Invalid version, looking for a signature with valid version...

Signature found at 0x5325000
Version: 2 (Windows 7 or later)

VMK entry found at 0x53250bd

VMK encrypted with Recovery Password found at 0x53250de
Salt: f655743e7840dabd862da0853b59fa18
Searching AES-CCM from 0x53250fa
Trying offset 0x532518d....
VMK encrypted with AES-CCM!!
RP Nonce: 40ffbcf9ae95d70106000000
RP MAC: 0cfdb0a61533e157eba77e136e6e8ae3
RP VMK: a3ba5e69617b312f9fb80c70e58232b80cc3cbfee8759c547f31b95c456b3e94f20b0564fdc14acea61e5f0b


VMK entry found at 0x5325249

VMK encrypted with User Password found at 532526a
VMK encrypted with AES-CCM
UP Nonce: c0bc89af7299d7010e000000
UP MAC: 1d2401ad85ea51764a4b7f1d1f342041
UP VMK: 0b83739da773e502713debe1ffec0ebef53ae4d2130793891e133b210c9d65ed35cf2260ca8ce322a83714d9


Signature found at 0x5335000
Version: 2 (Windows 7 or later)

VMK entry found at 0x53350bd

VMK entry found at 0x5335249

Signature found at 0x5a00000
Version: 2 (Windows 7 or later)

VMK entry found at 0x5a000bd

VMK entry found at 0x5a00249

User Password hash:
$bitlocker$0$16$877ac822ca83963fe55a44e603d9145c$1048576$12$c0bc89af7299d7010e000000$60$1d2401ad85ea51764a4b7f1d1f3420410b83739da773e502713debe1ffec0ebef53ae4d2130793891e133b210c9d65ed35cf2260ca8ce322a83714d9
Hash type: User Password with MAC verification (slower solution, no false positives)
$bitlocker$1$16$877ac822ca83963fe55a44e603d9145c$1048576$12$c0bc89af7299d7010e000000$60$1d2401ad85ea51764a4b7f1d1f3420410b83739da773e502713debe1ffec0ebef53ae4d2130793891e133b210c9d65ed35cf2260ca8ce322a83714d9
Hash type: Recovery Password fast attack
$bitlocker$2$16$f655743e7840dabd862da0853b59fa18$1048576$12$40ffbcf9ae95d70106000000$60$0cfdb0a61533e157eba77e136e6e8ae3a3ba5e69617b312f9fb80c70e58232b80cc3cbfee8759c547f31b95c456b3e94f20b0564fdc14acea61e5f0b
Hash type: Recovery Password with MAC verification (slower solution, no false positives)
$bitlocker$3$16$f655743e7840dabd862da0853b59fa18$1048576$12$40ffbcf9ae95d70106000000$60$0cfdb0a61533e157eba77e136e6e8ae3a3ba5e69617b312f9fb80c70e58232b80cc3cbfee8759c547f31b95c456b3e94f20b0564fdc14acea61e5f0b


 
┌──(root💀cf89b1317ce8)-[/Equinor-CTF/Forensic]
└─# john --wordlist=bitlocker-passwords.txt hashes
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (BitLocker, BitLocker [SHA-256 AES 32/64])
Cost 1 (iteration count) is 1048576 for all loaded hashes
Will run 16 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
ayRTBNm6kWG28FZVDgzuwxfj (?)
ayRTBNm6kWG28FZVDgzuwxfj (?)
Warning: Only 8 candidates left, minimum 16 needed for performance.
2g 0:00:01:01 DONE (2021-10-30 14:03) 0.03226g/s 48.39p/s 96.78c/s 96.78C/s SugcNKnJ26LkYpjysGxhaU57..DmszVJQrgyqL5PMukaAX67Ev
Session completed

```

My kali and linux instances were docker, in which it's non-trivial for third-time-user of docker to mount devices,\
so teammate to the rescue with how to mount it:


`losetup -o 65536 /dev/loop10 sysadmin.vhd` \
`dislocker -v -V /dev/loop10 -payRTBNm6kWG28FZVDgzuwxfj -- /media/mount/`\
`mount -t ntfs-3g -o loop /media/mount/dislocker-file /media/bitlockermount`\
In flag.txt: `EPT{bitlocker_go_brr}`