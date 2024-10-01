# Summary
Volatility is used for analyzing a memory dump forensics of a hacked machine.  

In this exercise, we examine a vmem file, of a hacked machine, using volatility to obtain evidentiary findings pertaining to what happened.

Here are some useful commands that I used during the investigations.

```bash
python3 vol.py -f mem.vmem windows.pstree # list processes & PPIDs
python3 vol.py -f mem.vmem windows.pslist # lists proceses
python3 vol.py -o outputdir/ -f mem.vmem windows.memmap --pid 123 --dump # dump suspicious process to outputdir/pid.123.dmp
python3 vol.py -f mem.vmem windows.dlllist | grep "process_name" # used to find path to executable or dll
python3 vol.py -f mem.vmem windows.dlllist --pid 123 # lists dlls associated with provided process id
strings pid.123.dmp | grep -iF "xxxxx" -C 5 # -C 5 shows 5 lines above / below; -i case insensitive; -F Fixed string;
```

----------------------
# Case 001
## Instructions
Your SOC has informed you that they have gathered a memory dump from a quarantined endpoint thought to have been compromised by a banking trojan masquerading as an Adobe document. Your job is to use your knowledge of threat intelligence and reverse engineering to perform memory forensics on the infected host. 

You have been informed of a suspicious IP in connection to the file that could be helpful. `41.168.5.140`

The memory file is located in /Scenarios/Investigations/Investigation-1.vmem

----------------------------
## Questions:

**1. What is the build version of the host machine in Case 001?**

```
2600.xpsp.080413-2111
```

**2. At what time was the memory file acquired in Case 001?**

```
2012-07-22 02:45:08
```

By using Volatility plugin **windows.info** we can obtain the answers to the first two questions:
```bash
python3 vol.py /Scenarios/Investigations/Investigation-1.vmem windows.info
```

![](b93eb825b1e7a752933b3a67dd2a9e13.png)

---

**3. What process can be considered suspicious in Case 001?**

```
reader_sl.exe 
```

**4. What is the parent process of the suspicious process in Case 001?**

```
explorer.exe
```

**5. What is the PID of the suspicious process in Case 001?**

```
1640
```

**6. What is the parent process PID in Case 001?**

```
1484
```

By using Volatility plugins **windows.psscan** & **windows.pstree** we can view the processes to obtain the answers to questions 3, 4, 5 & 6. The `reader_sl.exe` process was created by explorer.exe at the same exact time, which indicates the process was started when the desktop started/just after logon. `reader_sl.exe` is typically an Adobe Reader speed loader according to Google, but the rest of the processes are core Windows processes.

![](0eda1119a13f3a826c5ce1ac5bacd604.png)

```bash
python3 vol.py -f /Scenarios/Investigations/Investigation-1.vmem windows.psscan

python3 vol.py -f /Scenarios/Investigations/Investigation-1.vmem windows.pstree
```

---

**7. What user-agent was employed by the adversary in Case 001?**

```bash
Mozilla/5.0 (Windows; U; MSIE 7.0; Windows NT 6.0; en-US)
```

To obtain the user-agent employed by the adversary let's first dump the process memory for the suspicious process 1640. To do so, we will use the windows.memmap module with the following command:

Let's see what options we need to provide:
![](53f5cca570e4b26f5de2d83522fecd51.png)

```bash
sudo python3 vol.py -o /opt/volatility3/test/ -f /Scenarios/Investigations/Investigation-1.vmem windows.memmap --pid 1640 --dump
```
The `-o` flag is for the output directory, `--dump` is to create a memory dump file in the directory we provided, and `--pid` is the suspicious process id. By using the windows.memmap module and specifying the suspicious PID, we can dump the process memory. 

After running the command, we can see a `pid.1640.dmp` file was generated. Let's use strings on it to search for the IP we were provided in the beginning. (sudo was only used for read/write privileges)
```bash
sudo strings test/pid.1640.dmp | grep -iF '41.168.5.140' -C 5
```

![](aa8698565e13302f8e09890a9fcbfba3.png)

Using grep `-iF` is for case **i**nsensitive & **F**ixed String format. The `-C 5` option will show 5 lines above and below a match.

---

**8. Was Chase Bank one of the suspicious bank domains found in Case 001? (Y/N)**

```
Y
```

By using strings on the process memory dump, we can see chase.com along with other bank industry-related domains.
![](a105cea1247ec7ca75647b5a846957dd.png)

---

# Case 002 - That Kind of Hurt my Feelings

## Instructions

You have been informed that your corporation has been hit with a chain of ransomware that has been hitting corporations internationally. Your team has already retrieved the decryption key and recovered from the attack. Still, your job is to perform post-incident analysis and identify what actors were at play and what occurred on your systems. You have been provided with a raw memory dump from your team to begin your analysis.

The memory file is located in /Scenarios/Investigations/Investigation-2.raw

----------------------
## Questions

**9. What suspicious process is running at PID 740 in Case 002?**

```bash
@WanaDecryptor@
```

By using the windows.pstree module, we can locate the process name with PID 740.
```bash
sudo python3 vol.py -f /Scenarios/Investigations/Investigation-2.raw windows.pstree
```

![](6e89d6fab88c3a985141a2e8d9698878.png)

---

**10. What is the full path of the suspicious binary in PID 740 in Case 002?**

```bash
C:\Intel\ivecuqmanpnirkt615\@WanaDecryptor@.exe
```

Using the `windows.dlllist` module and grep, we can find the full path of the suspicious binary:
```bash
sudo python3 vol.py -o case002/ -f /Scenarios/Investigations/Investigation-2.raw windows.dlllist | grep -i 'wanadecryptor'
```
![](39bb3fd514a98a003c3491a12445ad42.png)

---

**11. What is the parent process of PID 740 in Case 002?**

```
tasksche.exe
```

---

**12. What is the suspicious parent process PID connected to the decryptor in Case 002?**

```
1940
```

---

**13. From our current information, what malware is present on the system in Case 002?**

```
Wannacry
```

---

**14. What DLL is loaded by the decryptor used for socket creation in Case 002?**

```
WS2_32.dll
```

For this answer, I listed the DLLs for process ID 740 using the windows.dlllist module. This was the command:
```bash
sudo python3 vol.py -o case002/ -f /Scenarios/Investigations/Investigation-2.raw windows.dlllist --pid 740
```

![](8cd4e450108b86d872da334a215fb493.png)

I went on Google and found out that [WS2_32.dll](https://learn.microsoft.com/en-us/windows/win32/winsock/initialization-2) is typically triggered by an application calling either **socket or WSASocket** to create a new socket to be associated with a service provider whose interface DLL is not currently loaded into memory.

---

**15. What mutex can be found that is a known indicator of the malware in question in Case 002?**

```
MsWinZonesCacheCounterMutexA
```

To answer this question, since I already knew the name of the malware, I looked up on Google what the **known mutex** was:
![](586bed5b20cfb89ff14a1ca5dc273942.png)

---

**16. What plugin could be used to identify all files loaded from the malware working directory in Case 002?**

```
windows.filescan
```

Using the `python3 vol.py --help` command, I browsed through the modules and found the one I thought fit best.

![](c1d9b797965efaade30c1df3566ac3da.png)
![](8cd4e450108b86d872da334a215fb493.png)
I went on google and found out that [WS2_32.dll](https://learn.microsoft.com/en-us/windows/win32/winsock/initialization-2) is typically triggered by an application calling either <b>socket or WSASocket</b> to create a new socket to be associated with a service provider whose interface DLL is not currently loaded into memory.

15. What mutex can be found that is a known indicator of the malware in question in Case 002?

```
MsWinZonesCacheCounterMutexA
```

To answer this question, since I already knew the name of the malware, I looked up on google what the <b>known mutex</b> was:
![](586bed5b20cfb89ff14a1ca5dc273942.png)
16. What plugin could be used to identify all files loaded from the malware working directory in Case 002?

```
windows.filescan
```

Using the `python3 vol.py --help` command, I browsed through the modules and found the one I thought fit best.

![](c1d9b797965efaade30c1df3566ac3da.png)

# Commonly used commands / Cheatsheet

```bash
python3 vol.py -o outputdir/ -f file windows.pstree # no hidden processes
python3 vol.py -o outputdir/ -f file windows.psscan # malware
python3 vol.py -o outputdir/ -f file windows.pslist
python3 vol.py -o outputdir/ -f file windows.dlllist --pid 740
python3 vol.py -o outputdir/ -f file windows.dlllist | grep -i "string"
python3 vol.py -o outputdir/ -f file windows.memmap --pid 1234 --dump
python3 vol.py --help

python3 vol.py -f file.dmp windows.dumpfiles.DumpFiles --pid <pid> #Dump the .exe and dlls of the process in the current directory

strings outputdir/pid.1234.dmp | grep -iF "string" -C 5 # -C 5 shows 5 lines above and below the match.

```

--------------------------------------------------
# References
* [https://tryhackme.com/r/room/volatility](https://tryhackme.com/r/room/volatility)
* [https://book.hacktricks.xyz/generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet](https://book.hacktricks.xyz/generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet)
* [https://github.com/volatilityfoundation/volatility/wiki/Command-Reference](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference)
