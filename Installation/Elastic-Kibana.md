<h1 align=center><img align="center" align=center src="https://www.elastic.co/apple-icon-57x57.png" height="45px" width="45px">&nbsp;&nbsp;ElasticSearch & Kibana Installation/Setup</h1>

## AWS
Frist Item we'll need to get Elastic up and running is a server to run it on. AWS is the most popular standard today for running VMs today, though you are happy to use what ever plateform you would like. 
Here are some recommendations if you are look for something something cheap to fit in your bucket as this is the only part of the lab were you might have to spend some money to host a VM [Cheap VM Hostings](https://webhostingadvices.com/19-cheap-vm-hosting/).
Though I would recommend that if your lacking in experiance with usin the using one of the big 3 cloud providers (AWS,Azure,Google) this would be a perfect place to dip your toes in one of them. 

If your running AWS for this lab you'll want to got the EC2 Dashboard and down to instance to get your VM up and running.
1. Go to 'Launch Instance' in the top right hand side
3. Give your VM a Name under 'Name and tags'. I'll be Naming my Elastic_Ubuntu
> [!TIP]
> Always try stick to a Naming Convention that makes sense, as this will help you in the long term 
4. To pick your OS go to 'Application and OS Images'. Here we'll be choosing Ubuntu as our OS and leave the rest of the 'Application and OS Images' configs as default.
6. Next we'll go down to 'Instance type' to pick our specs. With Elastic you can get away with running it on a T2.Medium though I would recommend a T2.Large as Elastic runs alot smoother with it.
7. To login to our server we'll need to setup a key, which we can do under 'Key pair'. Here if you click 'Create new key pair' to create a key.
   - Provide a 'Key pair name' which I'll keep the same as my server name
   - 'Key pair type' can be left as default (RSA)
   - 'Private key file format' will be up to you depending on what you prefer on using to connect to your machine. I'll be using .ppk for a putty connection.
   - One your happy with your configs click 'Create Key Pair'
   - This will download the Private key to your local host, which I recommend you then move to your project folder
> [!CAUTION]
> Once you have created your instance you will not be able to regenerate your private key. So double check that you have it installed before moving on. 

8. Down to our 'Network settings' we can leave all the default configs of 'Create Security Group' and 'Allow SSH traffic from' Selected. We will later change these configs to allow use to reach the elastic web page,
though this will be alot later on and we don't want to be leaving ports on for no reason.
9.         
