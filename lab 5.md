 convert binary number to equivalent ASCII number
 ```
 LXI SP FFFFh
LXI H 0000h
MOV A M
CALL BTA
MVI H D0h
MOV M A
HLT

BTA: CPI 00h
JC INVALID
CPI 10h
JNC INVALID
CPI 0Ah
JC SMALL
ADI 07h
SMALL: ADI 30h
RET
INVALID: HLT

```
 convert binary number to equivalent BCD number
convert ASCII number to equivalent binary number
convert and copy lower case  ASCII code to uppercase ASCII from memory location 9050h if any, otherwise copy as they are. Assume 50 ASCII codes in memory
 WAP to convert ten packed BCD numbers from the memory location 4350h to binary and store the result starting from 4360h.