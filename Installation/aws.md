# AWS
To get Elastic up and running, our first step is to set up a server, and AWS is the go-to platform for VMs. While you're free to choose any platform, AWS is highly recommended due to its popularity and reliability. If you're looking for cost-effective options, check out these Cheap VM Hostings. Keep in mind that this might be the only part of the lab where some expenses could be involved.
[Cheap VM Hostings](https://webhostingadvices.com/19-cheap-vm-hosting/).
If you lack experience with major cloud providers (AWS, Azure, Google), this is an excellent opportunity to dive into one of them.
<br><br>
### AWS Setup Instructions
1. Navigate to the EC2 Dashboard on AWS and select ***Launch Instance***.
2. Give your VM a meaningful name under ***Name and tags***. For example, I'll name mine Elastic_Ubuntu.

> [!TIP]
> Stick to a Naming Convention for clarity and future reference.

3. Under ***Application and OS Images***, choose Ubuntu as the OS and leave other configurations as default.
4. Move to ***Instance type*** to specify your specs. While Elastic can run on T2.Medium, T2.Large is recommended for smoother performance.
5. Set up a key for login under ***Key pair***. Click Create new key pair.
    - Provide a ***Key pair name***, keeping it consistent with your server name.
    - ***Key pair type*** can be left as default (RSA).
    - Choose the ***Private key file format*** depending on your preference (e.g., .ppk for Putty).
    - After configuring, click ***Create Key Pair***, and move the downloaded Private key to your project folder.<br><br>

> [!CAUTION]
> Double-check your private key installation, as it cannot be regenerated once the instance is created.

6. In ***Network settings***, leave the default settings for Create Security Group and Allow SSH traffic from selected. Later, we'll modify these to reach the Elastic web page, avoiding unnecessary open ports.
7. In ***Configure storage***, set the storage to 15 GiBs, with the option to add more volumes later if needed.
8. ***Advanced details*** can be left as default.
9. Review your summary and ensure under ***Number of instances***, only 1 instance is being created.
10. Deploy your instance by clicking ***Launch Instance***.<br><br>

Your VM is created, and while it might take some time to complete, head back to your instance dashboard to find the details, including your Private & public IP needed for later.

To connect to your machine via AWS, use the Connect button on the top right. If it's greyed out, your instance might not be ready or started. In the Connect to instance tab, focus on EC2 Instance Connect for AWS connection or SSH Client for connecting locally through SSH.
<br><br>
### Putty
For Windows users, connecting to your VM using Putty is a straightforward process. Putty is a commonly used software for SSH connections. Follow this quick walkthrough:

Download: [Putty](https://www.putty.org/)

1. Open up the Putty Application on your machine
2. Putty will load into the sesion detail tab, here you will enter in your Public IPv4 DNS to your instance into the ***Host Name (or IP Address)*** field. If your following along in the ***Connect to instance*** SSH Client Tab then your Public IPv4 DNS to your instance is provided in listing 4. ***Port*** should be set to 22 and ***Connection Type*** as SSH though this should be set automatically as default.
3. Next we need to add the user we are wanting to connect to, which can be done in the ***Data*** tab on the left under ***Connection***. Set the ***Auto-Login username*** to Ubuntu, which is the default user that is create when you launch a Ubuntu AWS Instance.
4. Lastly we need to apply our Private key that we downloaded when we made our insatnce. Expand the ***SSH*** and ***Auth*** tabs on the left hand side till you see the ***Credentials*** tab. In the credentials tab we want to select our Private key that we have on file by hitting ***Browse*** under the ***Private key file for authentication*** field.
5. Finally we can connect to our VM by hittin the ***Open*** button on the bottom right on the window

You'll most likily be greeted with a pop-up warning which you can just accept. Though beside from that you have now connected to your AWS VM. 
<br><br>
