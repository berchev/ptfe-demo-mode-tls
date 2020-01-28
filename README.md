# ptfe-demo-mode-tls
This repo is guideline on:
- TFE version 4 demo install using TLS certificate
- TFE version 4 restore from snapshot

There are two types of installation (Online or Airgapped).
We will focus on Online installation.


We are going to use vagrant in order to create appropriate development environment for that.
Our VM configuration include:
- private IP address (192.168.56.33) 
- /dev/mapper/vagrant--vg-root 83GB
- /dev/mapper/vagrant--vg-var_lib 113G

## Repo Content
| File                   | Description                      |
|         ---            |                ---               |
| [Vagrantfile](Vagrantfile) | Vagrant template file. TFE env is going to be cretated based on that file|
| [delete_all.sh](delete_all.sh) | Purpose of this script is to break our environment. We will use it during snapshot restore|

## Requirements
Please make sure you have fullfilled the reqirements before continue further:
- [Virtualbox](https://www.virtualbox.org/wiki/Downloads) installed
- Hashicorp [Vagrant](https://www.vagrantup.com/) installed
- [Basic Vagrant skills](https://www.vagrantup.com/intro/getting-started/) 
- PTFE license (You can contact Sales Team of HashiCorp - sales@hashicorp.com in order to purchase one)
- your own SSL/TLS certificates - In case you do not have such, you can generate for free using [Let's encrypt](https://letsencrypt.org/) 

## Getting started
- Clone this repo locally
```
git clone https://github.com/berchev/ptfe-demo-mode-tls.git
```
- Change into downloaded repo directory
```
cd ptfe-demo-mode-tls
```

## PTFE version 4 demo install using your own SSL/TLS certificate
- Start provision vagrant development environment
```
vagrant up
```
- connect to newly provisioned Vagrant machine
```
vagrant ssh
```
- Become `root`
```
sudo su -
```
- change to /vagrant directory (This is shared directory between the Host and the VM. It is better to keep the important stuff there.)
```
cd /vagrant
```
- Run the TFE installer from the shell of your instance, in order to start interractive installation
```
curl https://install.terraform.io/ptfe/stable | sudo bash
```
- Choose interface for TFE
```bash
vagrant@ptfe:/vagrant# curl https://install.terraform.io/ptfe/stable | sudo bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  134k  100  134k    0     0  46190      0  0:00:02  0:00:02 --:--:-- 46175
Determining local address
The installer was unable to automatically detect the private IP address of this machine.
Please choose one of the following network interfaces:
[0] enp0s3	10.0.2.15
[1] enp0s8	192.168.56.33
Enter desired number (0-1): 1
```
- Just hit enter on this step
```bash
The installer will use network interface 'enp0s8' (with IP address '192.168.56.33').
Determining service address
The installer was unable to automatically detect the service IP address of this machine.
Please enter the address or leave blank for unspecified.
Service IP address: 
```
- Type `N` or hit enter, if you do not use proxy
```
Does this machine require a proxy to access the Internet? (y/N) 
Installing docker version 18.09.2 from https://get.replicated.com/docker-install.sh
# Executing docker install script, commit: UNKNOWN
+ sh -c apt-get update -qq >/dev/null
+ sh -c apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
```
- Script will continue, installing Docker from Replicated site and pulling latest stable Replicated UI image
- Once script finish successfully, you will see following:
```bash
Created symlink /etc/systemd/system/docker.service.wants/replicated-operator.service → /etc/systemd/system/replicated-operator.service.

Operator installation successful

To continue the installation, visit the following URL in your browser:

  http://<this_server_address>:8800

vagrant@ptfe:/vagrant#
```

- Open your browser and type: `https://192.168.56.33:8800`, in oder to continue with the installation on UI
- Click on `Advanced`, since we are using self-signed certificate
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/1.png)

- Then click on `Proceed to 192.168.56.33 (unsafe)`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/2.png)

- On the next screen add the FQDN: `ptfe.georgiman.com`, upload your `privkey.pem`, `fullcahain.pem` and click on `Upload & Continue`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/3.png)

- Here you need to attach the TFE Licence purchased from HashiCorp
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/4.png)

- Once you provided licence, you need to choose between two types of installation (Online or Airgapped). We will proceed with Online
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/5.png)

- Type your password, and click on `Continue`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/6.png)

- After all the checks are passed, click on `Continue`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/7.png)

- On the next screen type the password you would like to use for data encryption, and choose `Demo` for installation type, into `Certificate Authority (CA) Bundle` field, add your `fullchain.pem` choose `TLSv1.2 and TLSv1.3` and click `Save` on the bottom of the page. Note that `fullchain.pem` need to be in following format:
```
-----BEGIN CERTIFICATE-----
MIIFtTCCA52gAwIBAgIIYY3HhjsBggUwDQYJKoZIhvcNAQEFBQAwRDEWMBQGA1UE
AwwNQUNFRElDT00gUm9vdDEMMAoGA1UECwwDUEtJMQ8wDQYDVQQKDAZFRElDT00x
CzAJBgNVBAYTAkVTMB4XDTA4MDQxODE2MjQyMloXDTI4MDQxMzE2MjQyMlowRDEW
MBQGA1UEAwwNQUNFRElDT00gUm9vdDEMMAoGA1UECwwDUEtJMQ8wDQYDVQQKDAZF
....
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIB5zCCAY6gAwIBAgIUNJADaMM+URJrPMdoIeeAs9/CEt4wCgYIKoZIzj0EAwIw
UjELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAkNBMRYwFAYDVQQHEw1TYW4gRnJhbmNp
c2NvMR4wHAYDVQQDExVoYXNoaWNvcnAuZW5naW5lZXJpbmcwHhcNMTgwMjI4MDYx
....
-----END CERTIFICATE-----
```
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/8.png)

- On the next dialog window, choose `Restart Now`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/9.png)

- You are currently on the dashboard. Need to wait the configuration process to finish
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/10.png)

- If you see that on screen, you are ready!
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/11.png)

## TFE version 4 restore from snapshot
Once we have installed TFE, we would like to have a snapshot of the last stable state. That's why we will create snapshot.
- In your browser type: `https://ptfe.georgiman.com:8800/dashboard`
- Click on `Start Snapshot` and wait the process to complete:
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/12.png)

- In order to verify that everything works as expected, we will break our system and then will restore it from snapshot

- Go to Terminal shell, become root
```
sudo su -
```
- Run `delete_all.sh` script located on `/vagrant` directory 
```
bash delete_all.sh 
```
- Exit from VM (most probbably you will need to type `exit` twice)
```
exit
```
- Restart your VM 
```
vagrant reload
```
- ssh to your VM again
```
vagrant ssh
```
- become root
```
sudo su -
```
- change to /vagrant directory
```
cd /vagrant
```
- Install replicated 
```
curl https://install.terraform.io/ptfe/stable | sudo bash
```
- When the installer asks for private IP of the machine, choose `1`
```
vagrant@ptfe:/vagrant# curl https://install.terraform.io/ptfe/stable | sudo bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  134k  100  134k    0     0  46190      0  0:00:02  0:00:02 --:--:-- 46175
Determining local address
The installer was unable to automatically detect the private IP address of this machine.
Please choose one of the following network interfaces:
[0] enp0s3	10.0.2.15
[1] enp0s8	192.168.56.33
[2] docker0 172.17.0.1
Enter desired number (0-2): 1
```

- Hit `enter` on the next question
```
The installer will use network interface 'enp0s8' (with IP address '192.168.56.33').
Determining service address
The installer was unable to automatically detect the service IP address of this machine.
Please enter the address or leave blank for unspecified.
Service IP address: 
```

- Choose `N` or hit `enter` about proxy option and wait the process to complete
```
Does this machine require a proxy to access the Internet? (y/N) N
Installing docker version 18.09.2 from https://get.replicated.com/docker-install.sh
# Executing docker install script, commit: UNKNOWN
+ sh -c apt-get update -qq >/dev/null
+ sh -c apt-get install -y -qq apt-transport-https ca-certificates curl >/dev/null
```

- After Replicated is installed successfully, you will see these line on the bottom of the screen:
```
Operator installation successful

To continue the installation, visit the following URL in your browser:

  http://<this_server_address>:8800

vagrant@ptfe:/vagrant#
```

- Open your browser and type: `https://192.168.56.33:8800`, in oder to continue on UI

- Click on `Advanced`, in order to accept the self-signed sertificate
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/2.png)

- On the next screen add the FQDN: `ptfe.georgiman.com`, upload your `privkey.pem`, `fullcahain.pem` and click on `Upload & Continue`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/3.png)

- Choose option `Restore from a snapshot`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/13.png)

- Click on `Browse snapshots` button
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/14.png)

- Next hit the orange button called `Restore`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/15.png)

- Type your password on the next screen and click `Unlock`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/16.png)

- Wait for all checks to pass and click `Continue`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/17.png)

- Click `Restore`
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/18.png)

- If everything is OK, you should see something like this:
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/19.png)
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/20.png)
![](https://github.com/berchev/ptfe-demo-mode-tls/blob/master/screenshots/21.png)
