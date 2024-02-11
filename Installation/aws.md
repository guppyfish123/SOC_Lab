# <img align="center" src="https://files.softicons.com/download/social-media-icons/free-social-media-icons-by-uiconstock/png/512x512/AWS-Icon.png" height="45px" width="45px">&nbsp;  AWS
While you're free to choose any platform, AWS is highly recommended due to its popularity and reliability. If you're looking for cost-effective options, check out these Cheap VM Hostings. Keep in mind that this might be the only part of the lab where some expenses could be involved.
[Cheap VM Hostings](https://webhostingadvices.com/19-cheap-vm-hosting/).
If you lack experience with major cloud providers (AWS, Azure, Google), this is an excellent opportunity to dive into one of them.
<br><br>
## ☁️ AWS Setup Instructions
1. Navigate to the EC2 Dashboard on AWS and select ***Launch Instance***.
2. Give your VM a meaningful name under ***Name and tags***.

> [!TIP]
> Stick to a Naming Convention for clarity and future reference.

3. Under ***Application and OS Images***, choose Ubuntu as the OS and leave other configurations as default.
4. Move to ***Instance type*** to specify your specs.
    |     Host    | Instance type  |                        
    |     :---:   |      :---:     |
    |   Elastic   |T2.medium <br> (Best performance) <br> T2.Large|
    |   TheHive   |    T2.medium   |
    |   Cortex    |    T2.medium   |
    |   Filebeat  |    T2.small    |
    |   Shuffle   |    T2.medium   |
    |Windows-Endpoint|  T2.medium  |
6. Set up a key for login under ***Key pair***. Click Create new key pair.
    - Provide a ***Key pair name***, keeping it consistent with your server name.
    - ***Key pair type*** can be left as default (RSA).
    - Choose the ***Private key file format*** depending on your preference (e.g., .ppk for Putty).
    - After configuring, click ***Create Key Pair***, and move the downloaded Private key to your project folder.<br><br>

> [!CAUTION]
> Double-check your private key installation, as it cannot be regenerated once the instance is created.

6. In ***Network settings***, leave the default settings for Create Security Group and Allow SSH traffic from selection. We will modify this later.
7. In ***Configure storage***, set the storage to 15 GiBs(For ubuntu), with the option to add more volumes later if needed.
8. ***Advanced details*** can be left as default.
9. Review your summary and ensure under ***Number of instances***, only 1 instance is being created.
10. Deploy your instance by clicking ***Launch Instance***.<br><br>

Your VM is created, and while it might take some time to complete, head back to your instance dashboard to find the details, including your Private & public IP needed for later.

To connect to your machine via AWS, use the Connect button on the top right. If it's greyed out, your instance might not be ready or started. In the Connect to instance tab, focus on EC2 Instance Connect for AWS connection or SSH Client for connecting locally through SSH.
<br><br>

## 🔒 Putty

For Windows users, connecting to your VM using Putty is a straightforward process. Putty is a commonly used software for SSH connections. Follow this quick walkthrough:

Download: [Putty](https://www.putty.org/)

1. Open up the Putty Application on your machine
2. Putty will load into the session detail tab, here you will enter in your Public IPv4 DNS to your instance into the ***Host Name (or IP Address)*** field. If your following along in the ***Connect to instance*** SSH Client Tab then your Public IPv4 DNS to your instance is provided in listing 4. ***Port*** should be set to 22 and ***Connection Type*** as SSH though this should be set automatically as default.
3. Next we need to add the user we are wanting to connect to, which can be done in the ***Data*** tab on the left under ***Connection***. Set the ***Auto-Login username*** to Ubuntu, which is the default user that is create when you launch a Ubuntu AWS Instance.
4. Lastly we need to apply our Private key that we downloaded when we made our instance. Expand the ***SSH*** and ***Auth*** tabs on the left hand side till you see the ***Credentials*** tab. In the credentials tab we want to select our Private key that we have on file by hitting ***Browse*** under the ***Private key file for authentication*** field.
5. Finally we can connect to our VM by clicking the ***Open*** button on the bottom right on the window

You'll most likely be greeted with a pop-up warning which you can just accept. Though beside from that you have now connected to your AWS VM. 
<br><br>

## Security     



### Security Groups
To ensure that our EC2 instances remain secure throughout their lifespan, it's crucial to configure their inbound and outbound ports in the attached security group following the principle of least privilege. This ensures that only the necessary ports are open for the instances to operate and perform their essential functions. The choice of open ports depends on the specific services running on a host at any given time. In this lab, we'll take our Elasticsearch instance as an example and outline the ports that should be open in our security group along with the reasons behind each.
<br>
Let's examine the services running on our Elasticsearch host, their associated ports, and whether they need to be accessed publicly:
<br>
- ElasticSearch: Running on port 6900 (Only needs internal VPC access)
- Kibana: Running on port 5601
- SSH: Running on port 22 (Only needs access from my IP)

Now that we have identified the services, associated ports, and their accessibility requirements, we can set up inbound and outbound rules for our security group.
<br><br>
**Inbound Rules**<br>

|     TYPE     |      PORT      |     SOURCE    |                             
|     :---:    |      :---:     |      :---:    |
|      TCP     |      5601      |   0.0.0.0/0   |
|      TCP     |      5601      |     ::/0      |
|      TCP     |      6900      | 172.31.0.0/16 |
|      SSH     |       22       |180.150.80.0/22|
<br>

**Outbound Rules**<br>

|     TYPE     |      PORT      |     SOURCE    |
|     :---:    |      :---:     |      :---:    |
|      TCP     |      5601      |   0.0.0.0/0   |
|      TCP     |      5601      |     ::/0      |
|      TCP     |      6900      | 172.31.0.0/16 |
|      SSH     |       22       |180.150.80.0/22|

<br>

> [!NOTE]
> SSH access has been restricted to my personal Private IP subnet. Ideally, it would be set to a dedicated IP. However, due to my ISP's dynamic IP assignment to my home network, regular updates would be required.

<br>

### Patch Manager
Following best practises, keeping your machines up to date with the latest security patches is crucial in maintaining our secure environment. Patch Manager is a AWS service that enables you to setup regular compliance checks with set policies on patches for your host that you have running. Patch Manager can ensure that devices are checked regularly to ensure that are running the latest version for their OS and there applications.
<br><br>
Patch Manager can bet setup in the AWS Systems Manager > Patch Manager > Dashboard. Here we will create a new patch policy by clicking ***Create Patch Policy*** and set your desired parameters in occurrence, to install or just scan, instances to include, and which policies to apply for each OS. For my patch policy I applied the standard OS policy and set to to scan & install any updates found for all instances every 2nd thursday (cron(18 0 ? * THU#2 *)). Scans can also be done manually during any time for a selected group of instances or all.
<br><br>

**user data**
In addition to employing patch management while instances are running, we can enhance the maintenance process by including a start up command in the User Data for our instances. The User Data is a script or cloud-init metadata that can be provided to an EC2 instance during launch. By adding a bash script, we can automate tasks to be executed on every boot.
<br>
Here's an example of a simple bash script in User Data:
```bash
#!/bin/bash
apt update -y
apt upgrade -y
```
By incorporating such scripts into User Data, we streamline the process of keeping our instances up to date from the moment they are launched. It's a proactive approach to maintaining system health and security by automatically applying updates as soon as the instance becomes operational.

<br>

### Secrets Manager
Secrets Manager is an AWS services which helps users in managing and retrieving credentials such as API keys, OAuth tokens and application credentails. Improving overall security posture as note to have sensitive information hard coded into application configurations files in your host. 
<br>
**Store a New Secret:**
1. To create a secret, go to your AWS Secrets Manager in AWS. Here we can create our secret by clicking the ***Store a new secret*** button.
2. First select the type of secret you want to store under ***Secret Type***. We'll select ***Other***.
3. In ***Key/value pairs*** we can make two rows, one for our username and password. Enter in a key and value format (e.g. Username: Cassandra).
4. Once completes, click ***Next*** on the bottom right of the window to continue.
5. Give the Secret a Name & Description under ***Secret name and description***
6. Leave all other settings as default and click ***Next*** until Secret is created
<br>

**IAM**<br>
Since our VM is running on AWS's EC2 instance service and our secrets are being stored using AWS's Secret Manager, we need to give permissions for each of the services to communicate with each other.
<br>
Creating Policy:
1. Navigate to the ***Identity and Access Management (IAM)*** Dashboard in AWS and under ***Access Management*** select ***Policies****.
2. Click on ***Create Policy*** to create a new policy
3. Change from ***Visual*** mode to ***JSON*** on the top right and paste the following JSON into the ***Policy Editor***:
    ```json
    {
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "VisualEditor0",
			"Effect": "Allow",
			"Action": "secretsmanager:GetSecretValue",
			"Resource": "<Secret_ARN>"
		}
	  ]
    }
    ```
> [!NOTE]
> Ensure when making permissions that we are following the best practise of least privileged, meaning we are assigning the least amount of access required that is needed to completed the required task.

4. Once completed, click ***Next*** to continue
5. Under ***Policy details*** provide a Name and Description for your policy
6. Complete your policy now by clicking ***Create Policy*** 
<br>

**Create Role**<br>
1. Navigate to the ***Identity and Access Management (IAM)*** Dashboard in AWS and under ***Access Management*** select ***Roles****.
2. Click on ***Create Role*** to create a new role.
3. Set ***Trusted entity type*** to ***AWS Service*** and set ***Use Case*** to  ***EC2***.
4. Clicking ***Next*** to continue, In the List of ***Permissions policies*** select the policy we had previously just made.
5. Clicking ***Next***, lastly provided a Name and Description under ***Role Details***.
6. Once completed, finalize your new role by clicking ***Create Role***
<br>

**Assign Role**<br>
Now that we have created our new role that provides permission for our EC2 Instance to pull a Secret from our Secret Manager, we need to assign it over to our required instance.
1. Navigate to ***EC2 Instance*** service to your ***Instances*** listing.
2. Select the Instance you want to assign our role to and navigate to ***Actions*** > ***Security*** > ***Modify IAM Role***.
3. Search for the Role that we had just created and select it.
4. Once completed, click on ***Update IAM Role*** to assign the new role to our EC2 Instance.
<br>

**CLI Get_Value**<br>
Our EC2 Instance will now access our secret that we created from our Secret Manager in AWS. To test this we can use the following command while in our host:
```bash
# Replace your_secretID & your_region with your own 
aws secretsmanager get-secret-value --secret-id your_secretName --region your_region
```
The output will look similar to this JSON structure:
```json
{
    "ARN": "arn:aws:secretsmanager:ap-southeast-342:234234:secret:secret-2342",
    "Name": "secret-name",
    "VersionId": "2344467-7f9d-4361-8517-fs923421290",
    "SecretString": "{\"username\":\"username\",\"password\":\"password\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
}
```
As we only need to pull only the username and password from the JSON ouput, we'll need to filter the results down to just those two values:
```bash
# Replace your_secretID & your_region with your own 
USERNAME=$(aws secretsmanager get-secret-value --secret-id your_secretName --region your_region | jq -r '.SecretString | fromjson | .username')
PASSWORD=$(aws secretsmanager get-secret-value --secret-id your_secretName --region your_region | jq -r '.SecretString | fromjson | .password')
```
