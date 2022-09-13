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

![1](https://user-images.githubusercontent.com/95281045/189846281-7a645d99-fef1-42b2-afc5-e9b8c53c19c9.png)



add name as per project requirement  
select runtime - python 3.9
click on create function 

![2](https://user-images.githubusercontent.com/95281045/189846692-5324a415-9aca-4a0a-a0c2-8028aec58b6e.png)



we need s3 bucket for backup restoring data 
first create new s3 bucket 

![3](https://user-images.githubusercontent.com/95281045/189846774-b100786a-1cfc-4c8a-b4ce-1280a4f28f37.png)



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


![4](https://user-images.githubusercontent.com/95281045/189846906-597b014a-6e55-4933-aba8-58c1386f8bc2.png)




click Create rule 


![5](https://user-images.githubusercontent.com/95281045/189846950-f8cd7ead-bf01-4a6f-ba60-35a29f4214da.png)



type hear rule name 
then click to Schedule 
and click on the next button 


![6](https://user-images.githubusercontent.com/95281045/189847012-7a57b416-9bc0-47eb-9187-9966faa23c70.png)




hear add cron time for hit as per your time 


![7](https://user-images.githubusercontent.com/95281045/189847097-325cf9be-9cf7-429c-891f-b32f6b747118.png)



ex. :-  Schedule expression: cron(0 10 8/14 * ? *)
then click to next button 
 
![8](https://user-images.githubusercontent.com/95281045/189847171-96fd0dbe-b326-414a-9af0-0e6259796b8e.png)




select a target lambda function 
and select your function and then click next button 


![9](https://user-images.githubusercontent.com/95281045/189848362-9d80e6fe-4a0b-44c0-a750-5f5e7cb67110.png)



now check your lambda function its automatically attack to EventBridge (CloudWatch Events) 


Then add life cycle Policy for s3 bucket logs backup in 14 days after delete automatically

hear we are using AWS service Lifecycle configuration

Lifecycle rules
yes i add new lifecycle policy 
Noncurrent versions actions - 0 
Days after objects become noncurrent - 30 days 
Number of newer versions - 5

![10](https://user-images.githubusercontent.com/95281045/189847345-34e32c1d-d4ab-4a0c-8f75-2d4678da97ad.png)
