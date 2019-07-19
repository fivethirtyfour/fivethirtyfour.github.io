---
layout: post
title: Self-hosted VPN with Algo and Azure
permalink: algo-vpn-and-azure
---

# What is Algo?

[Algo VPN](https://blog.trailofbits.com/2016/12/12/meet-algo-the-vpn-that-works/) is an open source self-hosted VPN service. There's tons of VPN services available but if you want to control what data is collected, used, and/or sold then rolling your own VPN service couldn't be much easier. 

Algo VPN is an on-demand VPN service in the cloud (or on a local machine that is open to the internet) that uses modern protocols and ciphers, and minimal software. Since the VPN is self-hosted the ability to teardown and rebuild couldn't be much easier.

There's little configuration and pretty close to "one-click" installs via the Ansible packager. 

# What you'll need

Listed below are what we will need for the required installation and configuration.

- An account on Azure*
- An installation of Ansible
- Access to a debian-base operating system (WSL, Ubuntu, Debian, VM w/Debian, etc)
- Support for macOS is also available

**Some assumptions**

- Windows Subsystem for Linux installed - since I will be doing this from a Windows machine I will be using WSL for this demonstration.
- Access to a bash shell

* *This is what we will be using for this demonstration however this is pretty universal for installation on all the other major cloud providers*

# Getting started

## Pre-requisites

### Azure-cli

We will need to install the **Azure CLI** in order for us to get shell access to our Azure environment. This will help automate a lot of the process and we won't have to make special API keys since we'll already be authenticated to the Azure environment.

Install the Azure-cli pre-requisites
```bash
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

### Ansible

Add Ansible repository
```bash
    sudo apt-add-repository -y ppa:ansible/ansible
```

Update repositories
```bash
    sudo apt update -y
    sudo apt upgrade -y
```

### Python tools

Install Python tools
```bash
    sudo apt install -y build-essential libssl-dev libffi-dev python-dev python-pip python-setuptools python-virtualenv
```

## Algo VPN server installation

Clone the repository
```bash
    git clone https://github.com/trailsofbits/algo
    cd algo
```

Install the remaining environment and requirements
```bash
    python -m virtualenv env
    source env/bin/activate
    python -m pip install -U pip
    python -m pip install -r requirements.txt
```

Configure the users that will be using the VPN service. In order to do this you will need to edit `[config.cfg](http://config.cf)` file that is located in the root of the *algo* directory. Use your favorite editor of choice to do this. *Substitute my users for whatever your users you want.*
```bash
    Users:
      -  itsame
      -  bestfriend
      -  themisses
```

**Not required:** *At this point if you want to change the default VM that Algo uses to create the image you can. You will need to change the pre-defined size.*
```bash
    cloud_providers:
      azure:
        size: Standard_B1S # This can be changed to something else if need be but this is pretty cheap already and enough for 200+ simultaneous connections
        image: 19.04
```

Save the file and exit out of the editor

**For Azure environments only:**

Log into your Azure environment via the Azure-cli that we installed earlier. Run the command and a window will pop open for you to authenticate to your environment.
```bash
    az login
```

Execute the command to start the Algo installation. I ran into some issues when running it without `sudo` so that's why we're doing this here. 
```bash
    sudo ./algo
```

You will be prompted to select the cloud provider you would like to install Algo on.

![Algo Cloud Provider](/assets/images/post-images/algo-vpn-azure/cloud-provider.png)

Since we're installing with Azure we will select `5`

Most of the defaults will be more than sufficient for you but just go through them and select what's more pertinent to you and your situation.

Select the region you would like to install your VPN server.

![Algo Region Selection](/assets/images/post-images/algo-vpn-azure/region.png)

From here the installer will run on its own without much interaction. 

**There is one caveat though, when it prompts for you to accept the fingerprint, if you don't do in time it will time out and the installation *WILL* fail.** 

![Algo SSH Prompt](/assets/images/post-images/algo-vpn-azure/ssh.jpg)

At this point if everything went well the installation should have completed successfully. 

![Algo Completed Message](/assets/images/post-images/algo-vpn-azure/completed.png)

## Connecting via a client

To configure the VPN clients Algo generates WireGuard configuration files for all the users that you specified in your `config.cfg` file. Those will be located at the following directories:

*For WireGuard configuration files used for "importing tunnel(s) from file"*

```
/algo/config/<ip of where the server was installed>/wireguard/<username>.conf
```

*For WireGuard QR code used for authentication on mobile devices*

```
/algo/config/<ip of where the server was installed>/wireguard/<username>.png
```

To install the WireGuard software you will need to install the specific software for your device. To find the installation files for your device you can head over to [https://www.wireguard.com/install/](https://www.wireguard.com/install/).

After installing the software on your given device import the tunnel configuration and `Activate` the tunnel in WireGuard. 

## You're all done!