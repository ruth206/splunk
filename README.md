# splunk
BOTV3 Splunk analysis

**Introduction**

A security operations centre(SOC) serves a critical role in monitoring and evaluating security data throughout an organisations infrastructure. It is used to prevent, detect and respond to cyber threats in real time. SOC primarily uses security information and event management(SIEM) platforms in modern enterprise setups to collect and analyse mass amounts of log data from servers, cloud services, and endpoints. Splunk is the main SIEM tool in this inquiry to facilitate SOC-style analysis and research.

In this report the Boss of SOC v3 (BOTSv3) dataset is used to simulate an actual enterprise setting and perform an analysis on it. BOTSv3 has multiple log sources, including cloud audit logs, host monitoring data, and system level information. This allows users to practice analysis and investigative techniques that are frequently used within a SOC. This report matches real world SOC operations by involving structured querying, field analysis, and evidence based conclusions.

In order to demonstrate effective SOC analysis, this report aims to illustrate how to use Splunk to examine security-related behaviour and system aspects in the environment. The report focusses on analysing identity and access behaviour, system configurations, and infrastructure visibility in order to detect noteworthy patterns, discrepancies, and security-relevant findings.

The scope of this project is limited to the data found in the BOTSv3 dataset and the log analysis performed with Splunk. There is no use of external tools or real-world systems. The dataset is presumed to be reliable for drawing conclusions and to accurately reflect the simulated environment.


**SOC Tier Responsibilities and Incident Handling**

The analysis that was carried out within this exercise mainly reflects the responsibilities of Tier 1 and Tier 2 SOC analysts.
Tier 1 analysts serve as a SOC's first line of defence. They are the first responders to security alerts and any possible suspicious activity.
Within the BOTSv3 exercise I performed tier 1 SOC detection by analysing AWS CloudTrail logs to identify IAM activity and to see whether AWS API calls were made without multi factor authentication which could indicate insecure authentication behaviour.

Tier 2 analysts are in charge of carrying out a more thorough study of tier 1 findings when any questionable activity has been detected within tier 1.
 Within the BOTSv3 exercise an AWS S3 bucket had been misconfigured and made publicly accessible. The investigation requirements were to identify the IAM user responsible, which bucket was impacted and what API action allowed public access. Further investigation was carried out to check if anything had happened to the data whilst the S3 bucket was public. Endpoints in the network were then checked finding one with a different version of windows than the others. These activities reflect Tier 2 investigation within a SOC and directly support incident response decision-making.
 
The exercise results demonstrate their connection to SOC operations through the prevention and detection and response and recovery phases. With regards to prevention The absence of multi-factor authentication for AWS API activity and the exposure of a publicly accessible S3 bucket reveals security control weaknesses that should have existed for prevention purposes. Detection was carried through with the use of Splunk to inspect CloudTrail logs, S3 access logs, and host monitoring data. This made it possible to identify insecure behaviour and misconfigurations within the environment. The investigation results support the response phase by providing the information required for rectification and containment steps such as identifying the affected bucket, responsible IAM user, API action, and potential data involvement. Lastly the recovery phase is used in order to lessen the possibility of such incidents like this happening in the future. This would entail applying lessons learned from these findings, such as strengthening MFA enforcement, tightening S3 access controls  and enhancing monitoring rules.



**LOADING THE BOTV3 DATA**

Figure 1 shows that the BOTSv3 dataset was obtained through the official GitHub repository this was to make sure that a trusted and widely used SOC training dataset was used. BOTSv3 was used in this exercise to simulate a real world scenario for threat hunting and log analysis.

<img width="522" height="363" alt="2025-12-07" src="https://github.com/user-attachments/assets/857dacd8-a8e3-471c-a324-7de29bac9f3e" />

Figure 2 shows a screenshot demonstrating that the dataset was downloaded and in the correct path.

<img width="489" height="331" alt="2025-12-07 (1)" src="https://github.com/user-attachments/assets/c33efb75-575c-4ee8-8f0d-c7104020ea2f" />

Figure 3 shows that Splunk Enterprise was installed on my virtual machine running Ubuntu Linux to simulate a typical SOC infrastructure environment. The virtualized Linux environment for Splunk deployment matches SOC operational practices because SIEM platforms commonly run on dedicated servers or virtual machines to achieve stability and maintain system isolation. The system runs on Ubuntu Linux since it is lightweight and works well with a wide range of log ingestion and processing tasks. A screenshot of the terminal output showing Splunk-related files on the machine is proof of this installation. 


<img width="792" height="395" alt="2025-12-07 (2)" src="https://github.com/user-attachments/assets/3289fee3-3886-431a-b283-f5306848e9d0" />

Once downloading the dataset it was ingested into Splunk. I confirmed that the dataset files were present on the system. I then executed searches against the botsv3 index to confirm successful ingestion and searchability of events. This is shown in figure 4.


<img width="773" height="380" alt="2025-12-07 (3)" src="https://github.com/user-attachments/assets/fbf0774e-917d-4e9d-8afc-60274a8b518b" />



**Finding the IAM (Identity & Access Management) users that accessed an AWS service in Frothly's AWS environment.**

I started by running the splunk search: 

index=botsv3 sourcetype=aws:cloudtrail | fields user* 

This search filtered the dataset to only AWS Cloudtrail logs and displayed only the fields that begin with user. This was to explore the fields sidebar and focus specifically on IAM-related information.

Once found i updated my search:

index=botsv3 sourcetype=aws:cloudtrail userIdentity.type="IAMUser"

From the result i examined the field userIdentity.userName which contained the IAM usernames. the usernames identified were:

  -splunk_access
  
  -web_admin
  
  -bstoll

   -btun

  


   
<img width="1892" height="964" alt="Screenshot From 2025-11-24 15-55-03" src="https://github.com/user-attachments/assets/6aa87918-c211-4aa2-a46c-f5319627cbb0" />

  **How i determined the field for API activity without MFA**

I took the following actions to find the field in the CloudTrail logs that shows wheather MFA(Multi-factor-authenitcation) was utilised during an AWS API call.

I began by searching the CloudTrail data for any events that contained MFA information.

CloudTrail stores raw JSON, and the MFA flag appears inside the nested fields. To locate any event mentioning MFA, i performed a keyword search on the JSON:

index=botsv3 sourcetype=aws:cloudtrail "\*mfaAuthenticated*"

This search returned events that contained the string mfaAuthenticated anywhere in the event.

I looked at the events displayed:

<img width="1892" height="964" alt="Screenshot From 2025-11-25 11-13-40" src="https://github.com/user-attachments/assets/9f0baeba-551a-47d2-b8d9-f7ff6f73c112" />

I expanded the JSON feilds of an event and inside the object i observed:


<img width="793" height="370" alt="2025-12-06" src="https://github.com/user-attachments/assets/acb48ee3-02a6-4962-bc37-d4d77940df1e" />



This verified that AWS CloudTrail explicity provides MFA staus in this field and that it uses a boolean value(True or False)

I ran the following to make sure Splunk could properly extract and search this field:

index=botsv3 sourcetype=aws:cloudtrail userIdentity.sessionContext.attributes.mfaAuthenticated=*
| stats values(userIdentity.sessionContext.attributes.mfaAuthenticated) BY eventType


This search filters events that contain the mfaAuthenticated field. Results are organised by event type(e.g. AwsApiCall) and it shows the actual True/False values.

The result showed:

<img width="749" height="191" alt="2025-12-06 (1)" src="https://github.com/user-attachments/assets/f152d889-9409-4d84-a1f1-f6c52f802c99" />

This confirmed that the field resolves properly in Splunk and that the API calls were done without MFA.


I identified the correct field by combining:

-AWS documentation. The documentation states that if MFA information is present it is stored inside 					userIdentity.sessionContext.attributes.mfaAuthenticated	

[(https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-user-identity.html)]

**REFRENCE**

-The astructure of the CloudTrail events. CloudTrail logs have a standard JSON nested format and all identity-related information appears under the userIdentity object and so it was a logical approach to look inside of:

- userIdentity
- sessionContext
- attributes

-The keyword seach mfaAuthenticated. This let me confirm weather the dataset actually contsained MDA information.

Once I saw inside the eventJSON, I knew the full path was:

userIdentity.sessionContext.attributes.mfaAuthenticated

<img width="652" height="131" alt="2025-12-06 (3)" src="https://github.com/user-attachments/assets/beb9f5ce-d9f7-4aba-a895-3299ce04b7cc" />


**Identifying the processor number used on the web server**

I identified the processor number used on the web server by searching for hardware-related logs using the following filter:

index=botv3 sourcetype=hardware

This search returned 3 results. All results contained the same hardware information from different hosts. The results showed identical CPU information indicating that the web servers are using the same processor model. The processor number is:

CPU E5-2676

<img width="1840" height="830" alt="Screenshot From 2025-11-29 12-10-16" src="https://github.com/user-attachments/assets/e0fc28e4-877b-4bfc-8351-82f05b8285cb" />



**Identifying the event ID of the API call that enabled public access**

To identify the event where an S3 bucket was made publicly accessible, i searched AWS cloudTrail logs using the following query:

index=botsv3 sourcetype=aws:cloudtrail eventName=PutBucketAcl

An S3 bucket's Access Control List (ACL) can be changed using the PutBucketAcl API function. Publicly accessible S3 buckets are frequently the result of improperly configured ACLs, which specify who is permitted to read or write to the bucket.

Two events were found by this search. The ACL field was empty in both instances, which is noteworthy since a blank or altered ACL may suggest that permissions have been intentionally altered or eliminated, possibly allowing public access.

I found the API call linked to this misconfiguration by looking at the event information. The event ID that made public access was:

eventID: ab45689d-69cd-41e7-8705-5350402cf7ac

<img width="1514" height="702" alt="Screenshot From 2025-12-02 13-55-35" src="https://github.com/user-attachments/assets/cc646fd5-aff8-4454-b39f-cb9a12e94f9a" />


The name of Buds username is:

bstoll

<img width="1508" height="439" alt="Screenshot From 2025-11-29 14-19-45" src="https://github.com/user-attachments/assets/dc3edfc2-07cf-407a-a998-c9cf59f96a11" />



The name of the bucket that was made publicly accessible was:

bucketName: frothlywebcode 



<img width="1012" height="815" alt="Screenshot From 2025-11-29 14-14-52" src="https://github.com/user-attachments/assets/1597deb7-86ee-4503-9b86-a97845b4c93c" />

7.To see what AWS of files were availible in the dataset i ran the following command:

index=botsv3 sourcetype=aws:* | stats count by sourcetype

This returned a list of AWS-related sourcetypes and confirmed that aws:s3:accesslogs was availible. I already knew the name of the S3 bucket from previously identifying it(frothlywebcode) i also knew that the file was a text file this led me to search the S3 access logs using:

index=botsv3 sourcetype=aws:* txt frothlywebcode

I then could identify a successful file upload event associated with the publicly accessible S3 bucket this was:

OPEN_BUCKET_PLEASE_FIX.txt


<img width="1840" height="717" alt="Screenshot From 2025-11-29 14-49-19" src="https://github.com/user-attachments/assets/825a6e87-5967-4700-9bae-494915bba0b3" />


By running index=botsv3 sourcetype=winhostmon | stats count by OS i was able to identify the os that appears fewer time 

Microsoft Windows 10 Enterprise	30
Microsoft Windows 10 Pro	174





To determine which Windows operating system version appeared less frequently, I queried the winhostmon sourcetype and used stats count by OS. The results showed that Windows 10 Enterprise had significantly fewer events compared to Windows 10 Pro, making it the least frequent operating system.

<img width="1850" height="405" alt="Screenshot From 2025-11-29 15-24-02" src="https://github.com/user-attachments/assets/5acad9fa-d383-4190-a64a-6f179b516e56" />

I then used the host field to identify the endpoint associated with this OS. The FQDN of the endpoint running the different Windows edition was BSTOLL-L.froth.ly


<img width="1846" height="631" alt="Screenshot From 2025-12-02 13-42-22" src="https://github.com/user-attachments/assets/2871341e-22fd-4a5a-92eb-0bbc1ac9d5e2" />








	







  
 

  

	
