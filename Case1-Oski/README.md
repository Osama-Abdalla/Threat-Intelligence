# Case1 Oski 

## Scenario
The accountant at the company received an email titled "Urgent New Order" from a client late in the afternoon. When he attempted to access the attached invoice, he discovered it contained false order information. Subsequently, the SIEM solution generated an alert regarding downloading a potentially malicious file. Upon initial investigation, it was found that the PPT file might be responsible for this download. Could you please conduct a detailed examination of this file?


## Oski Walkthrough 

### 1. Set Up Environment & Extract Challenge File
First, ensure you’re working within a safe, isolated analysis environment such as REMnux to prevent accidental infection. Extract the challenge files in this controlled environment. Inside the archive, locate hash.txt, which contains the MD5 hash of the suspicious PowerPoint file that triggered the SIEM alert. This hash serves as your entry point for threat intelligence research.

### 2. Identify Malware Family
Use the MD5 hash to query VirusTotal. Navigate to the Relations or Graph Summary tab to uncover malware family associations. In this case, the file is linked to Stealc, a known information-stealing malware commonly offered as Malware-as-a-Service.

### 3. Determine Malware Creation Time
Still in VirusTotal, check the Details → History tab to find the "Creation Time" of the sample. This metadata helps establish a timeline for the attack, although it should be treated cautiously, as timestamps can be manipulated.

<img width="753" height="224" alt="image" src="https://github.com/user-attachments/assets/4f39c705-10d1-4d10-a64a-2c987807b51e" />


### 4. Identify C2 Server
In VirusTotal’s Relations tab, review the list of contacted URLs. Often, the URL with a .php extension is designated as the Command and Control (C2) endpoint. Crowd-sourced context on VirusTotal may confirm this — in this case, the .php URL is identified as the C2 server.

<img width="1280" height="204" alt="image" src="https://github.com/user-attachments/assets/161f2d1c-e9cd-4b23-9062-e6b7d42ec9a7" />


### 5. Find Initial Post-Infection Library
Switch to the Behavior tab in VirusTotal to observe network communications. You’ll notice that the first library the malware requests is sqlite3.dll, which is retrieved from the C2 server—highlighting how Stealc dynamically loads external components.

### 6. Retrieve RC4 Key via Threat Intelligence Pivot
Pivot to Recorded Future Tria.ge using the sample’s hash. Their report includes a Malware Config section where strings and settings are revealed. Here, you can identify the embedded RC4 key used by the malware to decrypt payloads.

<img width="1471" height="820" alt="image" src="https://github.com/user-attachments/assets/0de0f996-6481-408b-bf3c-4b5fcc847ec0" />


### 7. Map to MITRE ATT&CK — Credentials Theft
Locate the comment containing a link to an Any.Run sandbox report. In Any.Run, click the ATT&CK button to view mapped techniques. The malware is found to use Credentials from Password Stores (T1555) to access browser-stored credentials.

<img width="800" height="381" alt="image" src="https://github.com/user-attachments/assets/31c991f6-57c7-4596-8b3c-7128072f8711" />


### 8. Map to MITRE ATT&CK — File Deletion
Further in Any.Run’s ATT&CK data, observe the malware performing Indicator Removal via File Deletion (T1070.004). A log entry reveals the malware executing a del command to remove DLLs from the C:\ProgramData directory.

<img width="593" height="848" alt="image" src="https://github.com/user-attachments/assets/016080b9-e8dc-435e-8859-e784f2aa0530" />


### 9. Identify Self-Deletion Delay
Inspect the process logs in Any.Run to locate the command with a timeout value. The malware waits 5 seconds before deleting itself and its components, a self-cleanup tactic used to avoid detection.

### 10. Summary
Through methodical intelligence gathering across multiple platforms—VirusTotal, Tria.ge, and Any.Run—we identified the malware’s family (Stealc), its behavior (library download and credential theft), its C2 infrastructure, encryption keys, and evasion mechanisms (self-deletion). Each step aligns with MITRE ATT&CK techniques, enhancing our understanding of the actor's methods and enabling better defensive strategies.


