# ICS challenges

## bacnetfingerprint

Task: 
```
Can you fingerprint the Network Engine that produced the communication shown in this packet capture?\
Provide the model name, firmware revision and the Windows Embedded CE version.\
Separate all items with a comma. The flag is case sensitive.

Submit flag in form EPT{something,#.#.#.#,#.#}. 
```
Taskfile: bacnet.pcap

Opening the pcap in wireshark and finds the requested information:
- packet 16's info says it is an ACK for the request readProperty model-name: `MS-NAE4510-2`
- packet 10's info says the same for firmware-revision: `5-1-0-4400` 

Spending WAAAAAYYYYY too long trying to fingerprint the windows version from 26 UDP packets. Finding out that there exists\
some charts, by the SANS institute that several webpages referenced but none linked to was an interesting rabbit-hole.

Then google comes to the rescue!
Googleing the devicename and finding hits from a company Johnson Controls seems correct. (Previous googling had provided\
a vendor-id list for BACnet. Packet 6 says vendor ID 5, list says this is Johnson Controls Inc. Packet 8 says vendor-\
name is JCI). The specsheet for the device provides the answer straight out: OS = MS Windows Embedded CE 6.0

=> flag: `EPT{MS-NAE4510-2,5.1.0.4400,6.0}`

## modbusread
Task:
```
Identify the Exception Code for the packet with a Modbus Read Discrete Inputs function code.
Provide the Exception Code as text - including spaces, and the frame number. Separate both items with a comma.
Submit flag in form EPT{something goes here,#}
```
Taskfile: modbus.pcap

Sets the wireshark filter to "modbus" to filter the packets. Scrolls down to "Func: 2"\
Finds the packet with the info-text "Exception returned", which is packet number 133

Exception code:         "Illegal data value"\
Packet/Frame number:    "133"\
Flag: `EPT{Illegal data value,133}`

## modbuswrite
Task:
```
Identify the IP addresses of the devices that are being written to through Modbus.\
Order the IP's in increasing order and separate them by commas. Don't use spaces within your flag.\

Submit flag in form EPT{something,#.#.#.#,#.#}.
```

Since this seems to be needing some text-manipulation shenanigans I jump into my conveniently accessible linux terminal\
where I have `tshark` available. Which is a CLI-based wireshark.


`tshark -r modbus.pcap |grep "Write" | grep "Modbus"| grep "Query" |awk '{print $5}'| sort |uniq`\
* -r - read pcap
* grep for packets with mode  Write
* grep only for packets with the string "Modbus" to try an avoid false positives
* grep Query to get packets in only one direction (from server to unit)
* awk-prints the  5th field to get the recipient IP(packet#, timestamp, source, arrow-character, destination, etc.etc)\
* sorts the output and then prints only the unique IPs
```
[root@686b9ce9d4ce ICS]# tshark -r modbus.pcap |grep "Write" | grep "Modbus"| grep "Query" |awk '{print $5}'| sort |uniq
Running as user "root" and group "root". This could be dangerous.
10.0.0.3
10.10.5.85
166.161.16.230
```
==> flag `EPT{10.0.0.3,10.10.5.85,166.161.16.230}`