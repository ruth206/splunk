# splunk
BOTV3 Splunk analysis

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

	
