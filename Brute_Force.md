# Brute Force Attack & Analysis
Summary:   
I conducted a brute force attack on a Windows 10 machine with Hydra in Kali Linux, and analyzed the forwarded log data in Splunk to help strengthen my attack pattern recognition, practice doing investigations in a SIEM, and have a deeper understanding of what to look for in a brute force attack scenario.

## The Attack
* My first step was to enable RDP on the victim, and add my created Active Directory users "edumbra" and "mbrown" to be able to remotely connect to the machine.
<img width="766" height="236" alt="remote_enabled" src="https://github.com/user-attachments/assets/01adfae0-7e81-4d8c-935f-b36f61a1bc29" />

* My next step was to create a password list to pass into hydra to brute force the target. I had copied rockyou.txt to this new directory, then to create my pasword.txt file, I cut the first 20 lines from rockyou.txt, and added the correct password at the end of the list.
<img width="488" height="82" alt="password-file" src="https://github.com/user-attachments/assets/94ef7a91-c212-4ab3-a048-a318f24a9ffb" /><img width="245" height="521" alt="passwords txt" src="https://github.com/user-attachments/assets/dc739c5b-65d8-4d01-b8ce-abde92ab9f61" />

* I then used hydra to conduct the attack by passing mbrown as the user, my created password.txt file as the password list, and specified the connection attempt as RDP to my 10.0.10.10 victim. After execution, hydra ran through the password list and came upon a successful login attempt for mbrown!
<img width="916" height="459" alt="hydra-crack" src="https://github.com/user-attachments/assets/bb2b6752-7fac-44a2-bfcf-1eb07f98ccca" />

* To test this password against the open RDP port, I attempted to connect to the victim with the newly discovered credentials. A couple moments after execution, I had a successful RDP connection!

<img width="544" height="51" alt="rdp-shell" src="https://github.com/user-attachments/assets/3db4770d-b301-4623-9478-a929323aae10" /><img width="561" height="439" alt="rdp success" src="https://github.com/user-attachments/assets/a28d94d6-2a37-412b-866f-469b7df35cd2" />

## Splunk Analysis
* To begin my analysis, I connected to my splunk server in a browser. Assuming an alert had come in for a potential brute force attempt, I started by doing a simple search for the user in question mbrown, event code 4625 to find failed login attempts, and set the time range for a 30 minute window. My query returned 21 events.
<img width="1922" height="900" alt="user-4625" src="https://github.com/user-attachments/assets/ec6614ff-cc6c-400e-a257-0c8e301f2e00" />

* My next query was aimed to sort the events in an easier to interpret format. I piped my first query into a stats count to see if all of the alerts were for the same host, which they were. I now knew that all 21 events were related to failed logon attempts from mbrown on the Victim-WIN10 host.

<img width="1916" height="340" alt="event-count" src="https://github.com/user-attachments/assets/73e24316-de39-4352-9476-4bbc126da4c7" />

* Trying to dive deeper, I tuned my query to output into a table. I included _time to get an idea of the timeline of events, I included the Workstation_Name and Source_Network_Address to determine where the login attempts were coming from, and included the Logon_Type to understand how each logon was being attempted. I was able to determine that all of the failed login attempts for mbrown on the Victim-WIN10 host were attempted non-interactively over the network due to the logon type being 3 by a host named kali with an IP of 10.0.10.100. Now, moving my attention to the time attributed to these events, all 21 events occured within the same minute, with only a couple events being a second or two spaced out.
* In a real scenario I would want to determine how abnormal these actions are for these assets. I would note if the source hostname follows similar naming conventions that my organization uses or not, as a clue if this may be malicious. I would also question if the source IP is normally in use, or if this was the first time it had been used as a clue. This is an internal IP address, so if it is determined malicious, how did the attacker gain initial access into the network? Is it normal for this user to login non-interactively over the network? Has the user ever used this source host before? These could all point to malicious intent. My largest alarm though, is the timeline of events that are consistent with automation with each event being so close together. Considering all of these clues I would conclude that this seems to be a malicious brute force attempt.

<img width="1912" height="836" alt="4625_IP_timeline" src="https://github.com/user-attachments/assets/9f41cf43-d01b-435e-a6d5-d3385e8ba2ed" />

* After deciding this seems malicious, I wanted to make sure to check if there was a successful logon attempt. I changed the event code in my query to 4624 for a successful logon attempt. The results do show that there was a successful logon for mbrown over the network from the kali host at 10.0.10.100 at 10:16:35.489. This happened almost immediately after the last failed login attempt. This incident is now a lot more dangerous upon discovering the attacker achieved a successful logon. Another thing to note, is that several minutes following the successful logon over the network, we see a logon type 10, meaning that it is possible that the attacker has also gained access to this machine using this user over RDP, having complete access in consideration to the permissions that mbrown has on this host.

<img width="1888" height="498" alt="image" src="https://github.com/user-attachments/assets/4aebcdb7-e86b-4f9a-afb7-6a773cb9e9e5" />

* These screenshots are showing the expanded events related to the successful logons above. The left one is the logon that occured over the network showing what we saw before with mbrown logging into the Victim-WIN10 host from kali at 10.0.10.100. The screenshot on the right shows the successful logon to Victim-WIN10 as mbrown over RDP. This alert shows the source host being Victim-WIN10, but noting the source IP address 10.0.10.100 which is consistent with all of the failed logon attempts and the successful logon over the network, showing that this actually came from the same source as before. I would conclude this logon is connected and malicious.

<img width="358" height="256" alt="successfullogin-IPsrc" src="https://github.com/user-attachments/assets/4faefbe4-ed1a-4154-89c4-7fb9ca571089" />

<img width="338" height="293" alt="RDP_IP-logon" src="https://github.com/user-attachments/assets/881fb585-3811-44a6-9845-4be9c5b2de99" />

* At this point in a real scenario I would keep digging to see what other suspicious events may have occured related to this user, this workstation, this IP, and consider any of those assets infected as well if found to have the same indicators of compromise. In this case the attack came from inside the network, so it would be important to identify how this threat actor got access to the network in the first place, or if this was an insider threat.

## Containment
* After determining the spread of this attack, I would want to contain it to prevent any further damage from happening. In this case with only the user account being breached, disabling it and revoking sessions would be my first action. If any other accounts were compromised they should get the same treatment.
* If anything suspicious was downloaded, any system changes like registry edits, new local user creation, file creations that could possibly be malicious, or any suspicions of installed persistence mechanisms, it may be worth looking into, and may be necessary to isolate the host off of the network.

## Eradication
* In the case of it only being the user account that was compromised, reset the user's password, and remove any changes they may have made post-contamination.
* If futher suspicious action had taken place, all traces of any persistence mechanisms must be removed, any malicious binaries removed, and any system changes reverted. If the host had been very deeply contaminated and there is low confidence in manually removing all malicious traces, a system restore or even a reimage may be necessary.

## Recovery
* Once everything has been properly cleaned of contamination, re-enable affected users, and bring systems back onto the network with a higher level of monitoring on the affected assets.
* Password policies should be enforced and stregthened to attempt to prevent this from occuring again in the future, along with enforcing account lockout policies, and enforcing MFA for each user.
* Recommendations for hardening network and endpoint security could be made in several different ways. Blocking usage of Hydra on endpoints using host level security configurations like a host based firewall, and blocking on the network level could be done using an IPS to detect Hydra's traffic patterns from this tool in the future as well.

## Reporting
* Following conclusion of the incident, I would report the timeline of everything that happened. Starting from initial access, and list the adversary, what actions were taken by the adversary, all affected assets, while including timestamps to put the incident on a timeline. Along with reporting the attackers actions, I would report any actions that I or my team had taken to contain, eradicate, and recover all affected assets and prevent similar incident from occuring again in the future.





