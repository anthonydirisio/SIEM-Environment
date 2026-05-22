# My Active Directory/SOC Environment Configuration
## Nat Network Setup
* Logical Topology map of the network I created with VirtualBox to simulate a real life Active Directory/SOC Environment so I can deepen my understanding of and practice working with Active Directory, common attack techniques, and forwarding endpoint telemetry to Splunk for analysis.

<img width="601" height="632" alt="image" src="https://github.com/user-attachments/assets/a39ea0ce-9a73-41bf-b331-1fdb60c6f311" />


## Active Directory Domain Controller
* I Installed a Windows Server 2022 machine, and set it up as the Domain Controller for my "jackismycat.local" domain and configured it with a static IPv4 address of 10.0.10.15/24.
<img width="349" height="191" alt="serverDC" src="https://github.com/user-attachments/assets/060b5ca6-e20e-4965-9995-e33ba6eaaa63" />

* I then configured 2 users in separate Organizational Units. Eileen Dumbra (edumbra) works in the HR department, and Malcolm Brown (mbrown) work in IT. These user accounts will serve as targets for my attacks.

<img width="311" height="170" alt="eileen_acct" src="https://github.com/user-attachments/assets/8ab526cb-7a7a-4897-8bb2-18537ac5b63b" /><img width="310" height="171" alt="malcolm_acct" src="https://github.com/user-attachments/assets/4bfadf80-0cbe-4b6b-8d59-1b543d6d74ad" />

## Splunk Server
* I installed an Ubuntu 24.04.4 LTS machine, and then installed and configured Splunk on it. I also configured this server to a static IPv4 address of 10.0.10.20/24.
<img width="409" height="224" alt="splunk-server-sertup" src="https://github.com/user-attachments/assets/3c15b707-937e-4912-89f8-48dec4a534cf" />
  
* Below is an example of Splunk being served on port 8000 from my Ubuntu server at 10.0.10.20. There are also a handful of events that have already been generated from my testing and forwarded to my server from the other endpoints I have setup, after configuring a Splunk Universal Forwarder on each endpoint.
<img width="1920" height="968" alt="splunk-server-serving" src="https://github.com/user-attachments/assets/ffb26650-8c25-4bcd-b5dd-b997feacc81d" />

## Kali Linux
* I also have installed Kali Linux to use as my attacking machine to generate telemetry on my victim machines to forward to Splunk. I set my Kali machine at a static IPv4 address of 10.0.10.100/24. Kali has most of the tools I need installed out of the box, but I have installed various other tools like Crowbar to perform various different attacks.
<img width="527" height="260" alt="kali-addresssetup" src="https://github.com/user-attachments/assets/fac0c0ab-7e40-42e4-9694-a63213ad2f45" />

## Target Machine
* For my target machine I installed a Windows 10 host, and configured it with a static IPv4 address of 10.0.10.10/24. Shown below as well, I have added this endpoint to my "jackismycat.local" domain. I also have installed Sysmon and Splunk Universal Fowarder to capture activity from this machine.
<img width="376" height="331" alt="victim-config" src="https://github.com/user-attachments/assets/7cd44621-5c85-4234-b55e-395327e2d8dd" />











 
