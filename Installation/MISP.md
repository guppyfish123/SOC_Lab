# <img align="center" src="https://docs.sekoia.io/assets/playbooks/library/misp.png" height="40px" width="40px"> &nbsp; MISP

<br><br>

## <div id="installation">💻 Installation
Make sure your host is up to date by running the following commands:

```bash
sudo apt update -y
sudo apt upgrade -y
```
<br>

Now that your host is up to date, proceed to install MISP by downloading the installation script:

```bash
wget --no-cache -O /tmp/INSTALL.sh https://raw.githubusercontent.com/MISP/MISP/2.4/INSTALL/INSTALL.sh
```
<br>

MISP provides an installation script that automates the process and downloads all required packages. Execute the following commands to run the script:
```bash
# Running this command will tell us the download options
bash /tmp/INSTALL.sh

# We'll be using the -A parameter to download all modules and packages for MISP
bash /tmp/INSTALL.sh -A
```
> [!NOTE]
> The installation process may take some time as it involves the installation of several dozen packages. There are a couple of manual interventions required, but the script handles most of the work to install MISP.

<br><br>

## <div id="configurations">⚙️ Configurations
Once `INSTALL.sh` has stop running, the finaly ouput should be the configs that it has setup for MISP including all creds the services it has setup on the host. Most of these will not be need besides as defualt creds to MISP to sign in which should be the first thing listed.
We should now be able to access MISP by going to `https:\\*Public-IP*` and login with our defualt credentials:
```
username: admin@admin.test
password: admin
```
<br>
When logging in for the first time, misp we prompt up to change the defualt password to the admin account.

### Make our Organisation 
In order for us to make our first organisation, go to Administration > Add Organisation. 
1. Enter ***ORG name*** into Organisation Identifier
2. Select ***Generate UUID***
3. Select ***Submit** once completed at the bottom to make your Organisation
<br>

### Creating a User
To make a new user for your organization go to Administration > Add User.
Fill in the following details for your user:
1. Emails address (This can be made up)
2. Tick ***Set Password*** and set a password for the account
3. Select your organisation that you have just made
4. Assign role as ***Org Admin***
5. Click Create User once completed 

<br>

### Feeds 
To enable feed to have them appear on your List Events, go to Synce Actions > Feeds. By defualt MISP should have ~73 feeds avaliable for you to enable at choice under the ***List Feeds*** tab.
If there are not ~73 feeds listed, they can be added manually by using the ***Import Feeds from JSON*** tab on the left.
Copy and Past the following JSON into MISP and click add:
https://github.com/MISP/MISP/blob/2.4/app/files/feed-metadata/defaults.json
<br>
Now that we have our feeds listed we can enable the ones we want by select them on the left hand side and click ***Enable Selected***. 
Once you have enable the feeds that you want selected you need to fetch the data from them by clicking the blue ***Fetch and Store all Feed Data*** Button.
To confirm that our feeds are syncing to MISP you can go to Administration > Jobs, where you can see each feed downloading data to MISP.

<br><br>

