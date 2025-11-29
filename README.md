# splunk
BOTV3 Splunk analysis

**LOADING THE BOTV3 DATA**
**SCREENSHOT**

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

   **SCREENSHOT**

  **How i determined the field for API activity without MFA**

I took the following actions to find the field in the CloudTrail logs that shows wheather MFA(Multi-factor-authenitcation) was utilised during an AWS API call.

I began by searching the CloudTrail data for any events that contained MFA information.

CloudTrail stores raw JSON, and the MFA flag appears inside the nested fields. To locate any event mentioning MFA, i performed a keyword search on the JSON:

index=botsv3 sourcetype=aws:cloudtrail "\*mfaAuthenticated*"

This search returned events that contained the string mfaAuthenticated anywhere in the event.

I looked at the events and expanded the JSON feilds of an event:

**SCREENSHOT**

And inside the object i observed:

**SCREENSHOT**

This verified that AWS CloudTrail explicity provides MFA staus in this field and that it uses a boolean value(Trude or False)

I ran the following to make sure Splunk could properly extract and search this field:

index=botsv3 sourcetype=aws:cloudtrail userIdentity.sessionContext.attributes.mfaAuthenticated=*
| stats values(userIdentity.sessionContext.attributes.mfaAuthenticated) BY eventType


This search filters events that contain the mfaAuthenticated field. Results are organised by event type(e.g. AwsApiCall) and it shows the actual True/False values.

The result showed:

**SCREENSHOT**

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

**SCREENSHOT**

**Identifying the processor number used on the web server**

I identified the processor number used on the web server by searching for hardware-related logs using the following filter:

index=botv3 sourcetype=hardware

This search returned 3 results. All results contained the same hardware information from different hosts. The results showed identical CPU information indicating that the web servers are using the same processor model. The processor number is:

CPU E5-2676

**SCREENSHOT**




	







  
 

  

	
