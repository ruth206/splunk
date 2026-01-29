# splunk
BOTV3 Splunk analysis

**Introduction**

In enterprise environments, a Security Operations Centre (SOC) oversees, identifies, and reacts to security-related incidents. Collecting and assessing massive amounts of data from cloud platforms, endpoints, and infrastructure systems, modern SOCs rely on Security Information and Event Management (SIEM) platforms. In this paper, SOC-style analysis and investigative activities are carried out using Splunk, a popular SIEM system.(Palo Alto Networks) 

The Boss of the SOC version 3 (BOTSv3) dataset, which replicates a realistic enterprise environment using a variety of log sources, such as Windows endpoint monitoring data, S3 access logs, host hardware telemetry, and AWS CloudTrail logs, is used . The dataset allows practical SOC methods, including field analysis, filtering, structured querying, and event correlation across many data sources. 

Finding security-related behaviour and possible setup errors in a simulated enterprise setting using Splunk is the aim of this inquiry. Examining AWS Identity and Access Management (IAM) activity, detecting API calls made without multi-factor authentication (MFA), looking into cloud resource misconfigurations like publicly accessible S3 buckets, and identifying any resulting data exposure are the specific objectives of the analysis. Additionally, host-level analysis is carried out to find endpoint configuration inconsistencies within Windows hosts and to determine hardware features of key systems. 

This project's scope is restricted to employing Splunk exclusively for the analysis of the BOTSv3 dataset. There is no use of real-world infrastructure, live systems, or external tools. It is expected that the recorded logs are trustworthy for making security-related deductions in line with SOC operational analysis, and  the dataset faithfully replicates a simulated enterprise environment. 

**SOC Tier Responsibilities and Incident Handling**

The analysis carried out within this exercise mainly reflects the responsibilities of Tier 1 and Tier 2 SOC analysts. 

Tier 1 analysts serve as a SOC's first line of defence; - the first responders to security alerts and any possible suspicious activity.(radiantsecurity) 

Within the BOTSv3 exercise I performed tier 1 SOC detection by analysing AWS CloudTrail logs to identify IAM activity and to investigate  if AWS API calls were made without multi factor authentication potentially indicating  insecure authentication behaviour.

Tier 2 analysts lead an intensive study of tier 1 findings when  questionable activity is detected within tier 1. 

Within the BOTSv3 exercise an AWS S3 bucket was misconfigured and made publicly accessible. The investigation requirements were to identify the IAM user responsible, which 
bucket was impacted and what API action allowed public access. Further investigation was carried out to check discrepancies to the data whilst the S3 bucket was public. Endpoints in the network were checked finding one with a different version of windows. These activities reflect Tier 2 investigation within a SOC and directly support incident response decision-making. 

The Tier 1-Tier 2 escalation strategy helped to streamline triage, but relying on manual Splunk searches creates an operational gap. In a genuine SOC, adding SOAR would alleviate the burden on analysts and speed up containment, making the total response significantly more efficient. 

The exercise results demonstrate their connection to SOC operations through the prevention and detection and response and recovery phases.(r) Regarding  prevention, the absence of multi-factor authentication for AWS API activity and the exposure of a publicly accessible S3 bucket reveals security control weaknesses that should have existed for prevention purposes. Splunk  was used for detection to inspect CloudTrail logs, S3 access logs, and host monitoring data. This enabled identification of insecure behaviour and misconfigurations. The investigation results support the response phase by providing the information required for rectification and containment steps, e.g. identifying the affected bucket, responsible IAM user, API action, and potential data involvement. Lastly the recovery phase is used to lessen the possibility of similar incidents happening moving forward. By applying lessons learned from these findings, it would strengthen MFA enforcement, tightenS3 access controls and enhance monitoring rules. 

**LOADING THE BOTV3 DATA**

Figure 1 showing the BOTSv3 dataset obtained through the official GitHub-repository ensuring a trusted and widely used SOC training dataset was used.  Using BOTSv3  in this exercise simulates a real world scenario for threat hunting and log analysis. 


<img width="522" height="363" alt="2025-12-07" src="https://github.com/user-attachments/assets/857dacd8-a8e3-471c-a324-7de29bac9f3e" />

Figure 2 showing  downloaded data set and its correct path. 

<img width="489" height="331" alt="2025-12-07 (1)" src="https://github.com/user-attachments/assets/c33efb75-575c-4ee8-8f0d-c7104020ea2f" />

Figure 3 shows Splunk Enterprise installed on a virtual machine running Ubuntu Linux to simulate a typical SOC environment. Using a virtualized Linux system reflects common SOC practice, as SIEM platforms are usually deployed on dedicated servers or virtual machines for stability and isolation. Ubuntu was chosen because it is lightweight and well-suited for log ingestion and processing. The terminal screenshot showing Splunk-related files confirms the 
installation. 

<img width="792" height="395" alt="2025-12-07 (2)" src="https://github.com/user-attachments/assets/3289fee3-3886-431a-b283-f5306848e9d0" />

Once downloading the dataset it was ingested into Splunk. I confirmed that the dataset files were present and then executed searches against the botsv3 index to confirm 
successful ingestion and searchability of events; - figure 4. 

<img width="773" height="380" alt="2025-12-07 (3)" src="https://github.com/user-attachments/assets/fbf0774e-917d-4e9d-8afc-60274a8b518b" />



**Finding the IAM  users that accessed an AWS service in Frothly's AWS environment.**

I commenced by running the splunk search: 

index=botsv3 sourcetype=aws:cloudtrail | fields user*  

The search filtered the dataset to AWS CloudTrail logs and displayed only fields 
beginning with user to focus on IAM-related information. 

Once found I updated my search: 
index=botsv3 sourcetype=aws:cloudtrail userIdentity.type="IAMUser" 

From the result I examined the field userIdentity.userName which contained the IAM 
usernames.  Usernames identified: - 

-splunk_access

-web_admin 

-bstoll 

-btun 

   
<img width="1892" height="964" alt="Screenshot From 2025-11-24 15-55-03" src="https://github.com/user-attachments/assets/6aa87918-c211-4aa2-a46c-f5319627cbb0" />

Identifying which IAM users access AWS services is important in a SOC because it 
shows who’s using cloud resources helping identify any unauthorised or unexpected 
access. 

  **How i determined the field for API activity without MFA**

I took the following actions to find the field in the CloudTrail logs that shows whether MFA(Multi-factor-authenitcation) was utilised during an AWS API call. 

I began by searching the CloudTrail data for events that contained MFA information. 

CloudTrail stores raw JSON, and the MFA flag is stored in nested fields. To locate any 
event mentioning MFA, I performed a keyword search in the JSON: 

index=botsv3 sourcetype=aws:cloudtrail "\*mfaAuthenticated*" 

This returned all events where the string **mfaAuthenticated** appeared anywhere within the log entry. 

I looked at the events displayed: 

<img width="1892" height="964" alt="Screenshot From 2025-11-25 11-13-40" src="https://github.com/user-attachments/assets/9f0baeba-551a-47d2-b8d9-f7ff6f73c112" />

I expanded the JSON feilds of an event and inside the object i observed:

<img width="793" height="370" alt="2025-12-06" src="https://github.com/user-attachments/assets/acb48ee3-02a6-4962-bc37-d4d77940df1e" />

This verified that AWS CloudTrail explicitly provides MFA status in this field and  it uses a 
boolean value. 

I ran the following to make sure Splunk could properly extract and search this field: 

index=botsv3 sourcetype=aws:cloudtrail userIdentity.sessionContext.attributes.mfaAuthenticated=* | 
stats values(userIdentity.sessionContext.attributes.mfaAuthenticated) BY eventType 

This search filters events  containing the mfaAuthenticated field. Results are organised by event type(e.g. AwsApiCall) and  shows the  True/False values. 

The result showed: 

<img width="749" height="191" alt="2025-12-06 (1)" src="https://github.com/user-attachments/assets/f152d889-9409-4d84-a1f1-f6c52f802c99" />

This confirmed  the field resolves properly in Splunk and that the API calls were done 
without MFA. 

I identified the correct field by combining: 

-AWS documentation. The documentation states that if MFA information is present it’s 
stored inside   
	userIdentity.sessionContext.attributes.mfaAuthenticated (AWS CloudTrail)

-The structure of the CloudTrail events. CloudTrail logs have a standard JSON nested 
format and all identity-related information appears under the userIdentity object and so 
it was a logical approach to look inside of: 

- userIdentity
-  sessionContext
- attributes

-The keyword search mfaAuthenticated. This let me confirm whether the dataset 
actually contained MDA information. 

Once I saw inside the eventJSON, I knew the full path was: 

userIdentity.sessionContext.attributes.mfaAuthenticated 

<img width="652" height="131" alt="2025-12-06 (3)" src="https://github.com/user-attachments/assets/beb9f5ce-d9f7-4aba-a895-3299ce04b7cc" />

It is important to check whether AWS API calls are using multi-factor authentication within a SOC,  because API activity without MFA increases the risk of unauthorised 
access: - an attacker would only need someone’s username and access keys to act as the user. They can then change cloud configurations, like making S3 buckets public and 
changing permissions or access control lists(ACLs). Analysts can use this information to find unsafe authentication behaviour and discover where to send alarms. 

**Identifying the processor number used on the web server**

I did this by searching for hardware-related logs using the following filter: 

index=botv3 sourcetype=hardware

It returned 3 results. All results contained the same hardware information from different hosts. The results showed identical CPU information indicating that the web servers are using the same processor model. The processor number is: 

CPU E5-2676

<img width="1840" height="830" alt="Screenshot From 2025-11-29 12-10-16" src="https://github.com/user-attachments/assets/e0fc28e4-877b-4bfc-8351-82f05b8285cb" />
A SOC that understands server hardware details can run system checks to confirm system functionality and identify any abnormal system behaviour. The understanding of 
server hardware enables analysts to identify unauthorized servers and systems that have incorrect configurations. The analysis of hardware data enables security teams to 
investigate incidents, confirming the protection status of vital system infrastructure. 


**Identifying the event ID of the API call that enabled public access**

To identify the event where an S3 bucket was made publicly accessible,  I searched AWS cloudTrail logs with the following: 

index=botsv3 sourcetype=aws:cloudtrail eventName=PutBucketAcl 

An S3 bucket's ACL can be changed using the PutBucketAcl API function. Publicly accessible S3 buckets are frequently the result of improperly configured ACLs, which 
specify who’s  permitted to read/ write to the bucket. 

Two events were found by this search. The ACL field was empty in both instances, which is noteworthy since a blank or altered ACL may suggest permissions have been 
intentionally altered or eliminated, possibly allowing public access. 

I found the API call linked to this misconfiguration by looking at the event information. The event ID that made public access was: 

eventID: ab45689d-69cd-41e7-8705-5350402cf7ac 

<img width="1514" height="702" alt="Screenshot From 2025-12-02 13-55-35" src="https://github.com/user-attachments/assets/cc646fd5-aff8-4454-b39f-cb9a12e94f9a" />


The name of Buds username is:

bstoll

<img width="1508" height="439" alt="Screenshot From 2025-11-29 14-19-45" src="https://github.com/user-attachments/assets/dc3edfc2-07cf-407a-a998-c9cf59f96a11" />



The name of the bucket that was made publicly accessible was:

bucketName: frothlywebcode 


<img width="1012" height="815" alt="Screenshot From 2025-11-29 14-14-52" src="https://github.com/user-attachments/assets/1597deb7-86ee-4503-9b86-a97845b4c93c" />

S3 buckets being made publicly accessible is a common security issue within cloud environments,  especially true when being made public by mistake. Having public 
access enabled on a system can reveal stored data to unauthorised users. By identifying the API action, the responsible user and the affected bucket, a SOC can take 
immediate action through public access removal and data exposure assessment to determine the extent of any potential breach. 

To see what AWS of files were available in the dataset I ran the following command: 

index=botsv3 sourcetype=aws:* | stats count by sourcetype 

This returned a list of AWS-related sourcetypes and confirmed that aws:s3:accesslogs was available. I already knew the name of the S3 bucket from previously identifying it 
(frothlywebcode). I also knew the file was a text file leading to search the S3 access logs using: 

index=botsv3 sourcetype=aws:* txt frothlywebcode 

I then identified a successful file upload event associated with the publicly accessible S3 bucket:- 

OPEN_BUCKET_PLEASE_FIX.txt 

<img width="1840" height="717" alt="Screenshot From 2025-11-29 14-49-19" src="https://github.com/user-attachments/assets/825a6e87-5967-4700-9bae-494915bba0b3" />
Checking the S3 access logs helps a SOC  understand whether any activity occurred whilst the bucket was publicly accessible. Finding a file that was uploaded within this 
period helps discover if data was involved and the  enormity of the incident. This information helps a SOC  with incidence response by  determining response by severity. 

By running: 

index=botsv3 sourcetype=winhostmon | stats count by OS  

I was able to identify the OS that appears fewer times.  

Microsoft Windows 10 Enterprise 30 

Microsoft Windows 10 Pro 174 

To determine which Windows operating system version appeared less frequently, I queried the winhostmon sourcetype using stats count by OS. The results showed  
Windows 10 Enterprise had significantly fewer events compared to Windows 10 Pro, making it the least frequent operating system. 

<img width="1850" height="405" alt="Screenshot From 2025-11-29 15-24-02" src="https://github.com/user-attachments/assets/5acad9fa-d383-4190-a64a-6f179b516e56" />


I used the host field to identify the endpoint associated with this OS. The FQDN of the endpoint running the different Windows edition was BSTOLL-L.froth.ly 


<img width="1846" height="631" alt="Screenshot From 2025-12-02 13-42-22" src="https://github.com/user-attachments/assets/2871341e-22fd-4a5a-92eb-0bbc1ac9d5e2" />
A SOC can detect abnormal systems in their environment through endpoint identification of operating system version differences. The combination of security 
configurations and patch levels between OS versions creates security risks for systems. The SOC can perform additional analysis determining necessary actions by identifying the specific endpoint. 

**Conclusion**

The implementation of stronger preventative controls through mandatory multi-factor authentication (MFA) for AWS API activity and strict S3 access policies  would enhance 
SOC detection and response capabilities. Enhanced SIEM alerting mechanisms for identifying insecure authentication patterns, cloud misconfigurations, and endpoint 
irregularities will support earlier threat detection. The combination of continuous monitoring with regular configuration validation will reduce the likelihood of similar 
security incidents occurring in operational environments. 

This exercise demonstrates the importance of centralized logging and structured analytical methods in SOC operations, highlighting how effective detection, 
investigation, and response processes strengthen organisational security posture. While Splunk is highly effective for in-depth log analysis, native AWS services such as 
GuardDuty and Security Hub are better suited for rapid, automated threat detection. A hybrid approach—using Splunk for detailed investigations and cloud-native services for real-time alerting—would significantly improve SOC efficiency and responsiveness. 








	







  
 

  

	
