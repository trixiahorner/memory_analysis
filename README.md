# Memory Analysis with Volatility
In this lab, we will perform memory analysis on a compromised system using Volatility. When a cloud-based or virtual machine server shows signs of compromise, taking a snapshot of the virtual machine is a stealthy way to capture its memory without alerting the attacker. This snapshot allows for thorough analysis without detection.
<br>

We will start by decompressing the memory dump of the compromised system. Using Volatility, we will then analyze network connections and process information to identify malware. While Volatility is a powerful free tool, it has some limitations, particularly with network PIDs. However, the concepts demonstrated here are applicable to any commercial tools you might use in your environment.

## Methodology

### 1. First start by extracting the memory dump using 7zip.
![7zip](https://github.com/trixiahorner/memory_analysis/blob/main/images/M1.png?raw=true)
<br>
<br>

### 2. Open Ubuntu-18.04 Prompt in Windows Terminal and extract Volatility
```
tar xvfz ./volatility3-1.0.0.tar.gz
```
<br>
<br>

### 3. Look at network connections (this takes a while)
```
python3 vol.py -f /mnt/c/tools/volatility_2.6_win64_standalone/memdump.vmem windows.netscan
```
![netscan](https://github.com/trixiahorner/memory_analysis/blob/main/images/M2.png?raw=true)


The above screenshot is... Concerning. We would want to look further into this because it is a SMB (port 445) connection to another computer. 
We know it is compromised (because it is a lab) but any time a "suspect" computer has another open connection to an internal system is, without question, a cause for concern. We have a strange, outbound connection with a strange executable on the system. And at the exact same time we have an internal connection to another host. What this is basically telling us is that there is a pivot relay for the command control. 




