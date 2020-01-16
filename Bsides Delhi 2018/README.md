# BSides Delhi - NTLM
## Never Too Late Mister - 200 points

### Description

My friend John is an environmental activist and a humanitarian. He really hated
the ideology of Thanos from the Avengers: Infinity War. He sucks at programming.
He used too many variables while writing any program. One day, John gave me a
memory dump and asked me to find out what he was doing while he took the dump.
Can you figure it out for me?

### Dump Analysis

We found a memory dump that we have to analyze Challenge.raw.

First we get the information from the image
```
volatility -f Challenge.raw imageinfo
          Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86
```
When analyzing the memory dump we see that there is a python script on the desktop.
Analyzing the image we see that the script has been executed and returns a
hexadecimal value.

Using Volatility to analyze, we see the output of the script using the following
command
```
volatility -f Challenge.raw --profile=Win7SP1x86 consoles

C:\Users\hello>C:\Python27\python.exe C:\Users\hello\Desktop\demon.py.txt
335d366f5d6031767631707f
```

We continued analyzing the image and found the following string
```
Thanos = xor and password
```
XOR brute force is tested on the hexadecimal chain and we find the last part of
the flag.
```
 IN = 335d366f5d6031767631707f
 KEY = 0x02
 OUT = 1_4m_b3tt3r}
```
We're still analyzing the image to find the first chain of the flag.

In this case we use a Mimikatz plugin for Volatility
```
Module    User             Domain            Password
-------- ---------------- ---------------- ------------------------------------
wdigest hello              hello-PC         flag{you_are_good_but
wdigest HELLO-PC$          WORKGROUP
```
And if we put the two flag pieces together, we've got the challenge solved.

Thanks to Wagiro and Patatas
