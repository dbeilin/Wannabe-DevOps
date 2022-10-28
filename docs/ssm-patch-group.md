Recently I needed to add multiple instances to a Patch Group.
- This is done by adding a tag to each instances where the key is Patch Group and the value is a name of your choice.
If you use AWS’s SSM service, you know that through the Console you are only able to add one tag at a time.

This short script will show you how to tag multiple instances at once.
Before that, we need to differentiate between EC2 instances and SSM Managed Instances.

* EC2 Instance – a regular EC2 instance. Represented by an instance ID that starts with i-
* Managed Instance – An on-premise or non EC2 managed instance that you can manage in SSM. Represented by and instance ID that starts with mi-

<u>Before you start, make sure you have 2 things:</u>

- Boto3 installed
- The appropriate IAM permissions (use aws configure to set your test environment)

I want to add all of my Amazon Linux 2 machines to their own Patch Group.
In my example, I only add the tags to EC2 instances, but the same logic applies to managed instances as well.

```py
import boto3
ssm_client = boto3.client('ssm')
ec2_client = boto3.client('ec2')
 
tags = {'Key': 'Patch Group',
        'Value': 'AL2-Test'
}
 
all_instances = []
for instance in ssm_client.describe_instance_information()['InstanceInformationList']:
        if instance['PlatformName'] == 'Amazon Linux' and instance['PlatformVersion'] == '2':
                all_instances.append(instance['InstanceId'])
 
for instance in all_instances:
        ec2_client.create_tags(
                # DryRun=True,
                Resources=[instance],
                Tags=[tags]
        )
```

???+ tip "You can choose whatever tags you want for your key/value"
    ```py
    tags = {'Key': 'Patch Group',
        'Value': 'AL2-Test'}
    ```

    You can also use more tags.

The first loop iterates over all of the instances in SSM, and if the instance’s OS is Amazon Linux 2, it gets added to a list of instances.

The second loop goes over the list of instances we just filled and simply creates the tags we configured.
As you can see, by running we get our desired result:

![alt](../site/assets/images/ssm-patch-group/LLZvZ75.png)