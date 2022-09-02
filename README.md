# Create-AWS-logs-backup-in-14-days-after-delete-automatically

requirement of Whole process

```sh
AWS sesrvice i am using name like that 
AWS service lambda function
IAM role 
s3
pythone script 
```





# Install SSM Agent in server 


click link https://www.decodingdevops.com/how-to-install-ssm-agent-on-linux-ec2-instance/  and install SSM Agent  

note if instance are not install ssm agent so first check iam role and attached the policy  

```
AmazonSSMFullAccess
```

then stop instance and restart again 

note  :- if Elastic IP not associate you instance so do not stop instance and restart again 


# create AWS service lambda function 
 
click on create function 




add name as per project requirement  
select runtime - python 3.9
click on create function 





we need s3 bucket for backup restoring data 
first create new s3 bucket 






then check IAM role 

policy required  

```sh
AmazonEC2FullAccess
AmazonS3FullAccess
AmazonSSMFullAccess
```



we need for the script 

```sh
import time
import json
import boto3
    
def lambda_handler(event, context):

    # boto3 client
    client = boto3.client('ec2')
    ssm = boto3.client('ssm')
    
    # getting instance information and ssm instance information
    describeInstance = client.describe_instances()
    ssm_describeInstance = ssm.describe_instance_information()

    InstanceId="pute hear instance id"	#Define the Instance

    # command to be executed on instance
    response = ssm.send_command(
        InstanceIds=[InstanceId],
        DocumentName="AWS-RunShellScript",
        Parameters={'commands': [' ]})
                
    # fetching command id for the output
    json_response = response['Command']['CommandId']
    time.sleep(3)

    # fetching command output
    json_output = ssm.get_command_invocation(
        CommandId=json_response,
        InstanceId=InstanceId
        )
        
    return json_output 
```


there is Parameters={'commands': [' ]})

inside the parameters command add command as per your project 

# what you need for this

click on the fist deploy then click on test  


Now hit cron for every 14 days 

now go to AWS service Amazon EventBridge 
and click to Rules under the events 






click Create rule 





type hear rule name 
then click to Schedule 
and click on the next button 






hear add cron time for hit as per your time 





ex. :-  Schedule expression: cron(0 10 8/14 * ? *)
then click to next button 
 




select a target lambda function 
and select your function and then click next button 





now check your lambda function its automatically attack to EventBridge (CloudWatch Events) 


Then add life cycle Policy for s3 bucket logs backup in 14 days after delete automatically

hear we are using AWS service Lifecycle configuration

Lifecycle rules
yes i add new lifecycle policy 
Noncurrent versions actions - 0 
Days after objects become noncurrent - 30 days 
Number of newer versions - 5
