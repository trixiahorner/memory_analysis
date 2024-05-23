# Memory Analysis with Volatility
In this lab, I will perform memory analysis on a compromised system using Volatility. When a cloud-based or virtual machine server shows signs of compromise, taking a snapshot of the virtual machine is a stealthy way to capture its memory without alerting the attacker. This snapshot allows for thorough analysis without detection.
<br>

I will start by decompressing the memory dump of the compromised system. Using Volatility, I will then analyze network connections and process information to identify malware. While Volatility is a powerful free tool, it has some limitations, particularly with network PIDs. However, the concepts demonstrated here are applicable to any commercial tools you might use in your environment.

## Methodology

### 1. Extracting the memory dump using 7zip.
![7zip](https://github.com/trixiahorner/memory_analysis/blob/main/images/M1.png?raw=true)
<br>
<br>

### 2. Opening Ubuntu-18.04 Prompt in Windows Terminal and extract Volatility
```
tar xvfz ./volatility3-1.0.0.tar.gz
```
<br>
<br>

### 3. Network connections (this takes a while)
```
python3 vol.py -f /mnt/c/tools/volatility_2.6_win64_standalone/memdump.vmem windows.netscan
```
![netscan](https://github.com/trixiahorner/memory_analysis/blob/main/images/M2.png?raw=true)


The above screenshot is concerning. I want to look further into this because it is a SMB (port 445) connection to another computer. 
I know it is compromised (because it is a lab) but any time a "suspect" computer has another open connection to an internal system is, without question, a cause for concern. It is a strange, outbound connection with a strange executable on the system. And at the exact same time there is an internal connection to another host. What this is basically telling me is that there is a pivot relay for the command control. 
<br>
<br>

### 4. Processes on this system
```
python3 vol.py -f /mnt/c/tools/volatility_2.6_win64_standalone/memdump.vmem windows.pslist
```
![processes](https://github.com/trixiahorner/memory_analysis/blob/main/images/M3.png?raw=true)

The cmd.exe catches my attention. Generally, users and day to day usage of a system does not spawn a cmd.exe session. We may see it briefly as part of some sysadmin scripts. However, it is not seen all that often in normal day-to-day user interactions
<br>
<br>

### 5. PStree to see a bit more detail on what spawned the process
```
python3 vol.py -f /mnt/c/tools/volatility_2.6_win64_standalone/memdump.vmem windows.pstree
```
![pstree](https://github.com/trixiahorner/memory_analysis/blob/main/images/M4.png?raw=true)

In the above example, I can see that the parent process for TrustMe was Explorer.exe. This means it was invoked by the user on the system, as Explorer.exe is the GUI process for Windows 10.

I trace back the parent process for one of the cmd.exe files back to TrustMe.exe. When hunting down these processes it helps to track the parent processes. It can help create a sort of timeline for the actions on the system. 
<br>

![pstree](https://github.com/trixiahorner/memory_analysis/blob/main/images/M5.png?raw=true)

I see a connection out to the Internet. I also see that cmd.exe is being invoked. Then, the net command being invoked on this computer system. This is the pivot. This system is using this system to pivot and access another one with the net command. 
<br>
<br>

### 6. Let's dive into the TrustMe.exe process a bit further with dlllist
```
python3 vol.py -f /mnt/c/tools/volatility_2.6_win64_standalone/memdump.vmem dlllist --pid 5452
``` 
![dll](https://github.com/trixiahorner/memory_analysis/blob/main/images/M6.png?raw=true)

The above command shows the dll's asscociated with the TrustMe process. I also see the command line invocation of this process. This is great as it tells me any flags used to start the process and it can tell us where on the system it was executed from
<br>
<br>

### 7. Finally we can look at the easy button with *malfind*. 
This module will look at the processes for any suspicious activities.
```
python3 vol.py -f /mnt/c/tools/volatility_2.6_win64_standalone/memdump.vmem windows.malfind.Malfind
```
![malfind](https://github.com/trixiahorner/memory_analysis/blob/main/images/M7.png?raw=true)

This looks at how the executable is established. Here it saying what caused it to freak out. There’s a page in memory that’s execute, read, and write at the same time (ERW file). It’s not read or write or execute. It’s all of them at the same time, which is something that we see malware do.
<br>
<br>
The presence of an ERW memory page is a red flag because it goes against standard security practices that separate code execution from data modification. This combination is often exploited by malware to inject, modify, and execute code dynamically, making it a strong indicator of malicious activity.
<br>
<br>

# Conclusion
By leveraging Volatility, a powerful and freely available tool, I was able to conduct a thorough investigation without alerting potential attackers. This approach highlights the importance of stealth in incident response and the value of memory analysis in uncovering hidden threats. Understanding how to effectively analyze memory dumps is a crucial skill for any cybersecurity professional, enabling proactive defense against sophisticated attacks.
