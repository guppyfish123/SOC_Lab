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
We should now be able to access MISP by going to `https:\\*Public-IP*` and login with our defualt creds provided.
<br>
We should first be presented with changing our password to our admin account when we login for the first time. 

### Make our Organisation 
In order for us to make our first organisation, go to Administration > Add Organisation. 
1. Enter ***ORG name*** into Organisation Identifier
2. Select ***Generate UUID***
3. Select ***Submit** once completed at the bottom to make your Organisation

<br><br>

## <div id="startup">🚀 Startup
