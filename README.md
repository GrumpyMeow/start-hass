# How i started with Home Assistant development

For a while i really wanted to do some development on Home Assistant to resolve issues with integrations i was using. As i have a busy life and a short attention span, it mostly never resulted in actually resolving the issues. And to be honest i still find it hard to actually resolve issues and get it accepted into the official Home Assistant code-base.

The documentation of Home Assistant is quite complete, but i found it hard to absord all the information which is there. I mostly just start developing and lookup information there, when i run into issues. With this write-up i'm hoping that others will be able to follow my steps and join into improving Home Assistant via development.

# Creating an Home Assistant Development Environment
I use Proxmox for self-hosting quite a few services in LXC containers, and wanted to also use a LXC-container for Home Assistant development. The advised way for Home Assistant development is to use the devcontainers feature of VSCode. But as this would result in having to install Docker into the LXC container. I thought this would result into overhead and decided to do without the devcontainer feature and install things like Python directly in the LXC-container. One thing i didn't take into account that i would run into issues which were related to the used Python version. Some Python packages which were needed for Home Assistant, were not compatible/available for the specific Python version which i was using. I just chose ambivalently chose a Python version to install and didn´t take into account that it's needed to align with the Python version which Home Assistant needed at that given point in time. This resulted into too much work and issues to resolve before actually getting around to do my intended development work.

## Create Proxmox Docker LXC container
I eventually chose to align with the preferred using of devcontainers. I still wanted to host the development environment using Proxmox, as this would allow me to switch between workingstations and don't having to maintain multiple development environments. I ended up creating a Docker LXC container using [TTeck Proxmox Helper scripts](https://tteck.github.io/Proxmox/). I opened up the Proxmox shell via the Proxmox GUI and ran the command: `bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/docker.sh)"`
![afbeelding](https://github.com/GrumpyMeow/start-hass/assets/12073499/c607e4d8-9142-4372-adfc-4016fb27ed7f)
![afbeelding](https://github.com/GrumpyMeow/start-hass/assets/12073499/07dd6b41-553e-4e4f-b686-58ce22857139)
![afbeelding](https://github.com/GrumpyMeow/start-hass/assets/12073499/d151af65-c95d-4599-af82-f0c9f2f2214c)
![afbeelding](https://github.com/GrumpyMeow/start-hass/assets/12073499/71e36af4-0f83-4bfb-9089-c5fcaaf608fd)
![afbeelding](https://github.com/GrumpyMeow/start-hass/assets/12073499/23785aed-83eb-4b02-a151-278078610a04)
![afbeelding](https://github.com/GrumpyMeow/start-hass/assets/12073499/032b2e06-696c-4e14-a407-faf2963b446e)

## SSH-enable Proxmox Docker LXC container
I normally connect via SSH using password-authentication to remote environments. But in the past i already experienced that i then had to enter the password many times when using VSCode in combination with a remote SSH. Thus i set out to enable authentication via key-authentication. I wanted to do development using VSCode installed on my laptop and looked if i had already create key-pair-files by running: `ls ~/.ssh`. Key-pair-files are files named: `id_rsa` and `id_rsa.pub`. Those files didn't exist, thus i created them using: `ssh-keygen -t rsa -b 4096`. The public-file of this key-pair now needs to be copied to the LXC container we created using the TTeck script. 

We will be going to use a snippet to copy the public-key-file to the remote-SSH-server via SSH. But by default it's not possible to login into the remote SSH-server as no password has been set for the root user and some settings needs to be changed for this. We will need to enable this in the LXC container:
Via the following command we will edit the configuration of the SSH-server: `nano /etc/ssh/sshd_config`. In this file we will be changing:
* `#PermitRootLogin prohibit-password` into `PermitRootLogin yes`
* `#PasswordAuthentication yes` into `PasswordAuthentication yes`
* `#PermitEmptyPasswords no` into `PermitEmptyPasswords yes`
Save the changes via Ctrl-O, Ctrl-X.
Restart the SSH-service: `/etc/init.d/ssh restart`
In the LXC-container we set the password of the root user via: `passwd root`. 

After this the following snippet can be used to copy public-key-file to the remote-SSH-server via SSH:
```
export USER_AT_HOST="root@192.168.178.123"
export PUBKEYPATH="$HOME/.ssh/id_rsa.pub"
ssh-copy-id -i "$PUBKEYPATH" "$USER_AT_HOST"
```
You will be prompted to enter the password of the root user you have configured.

## VSCode SSH
It's now time to start VSCode. We will now install the "Remote SSH VSCode extension" (`ms-vscode-remote.remote-ssh`) and connect to the LXC container via SSH. Pressing CTRL-SHIFT-P in VSCode will bring up the command input at the top of the screen where we'll search for the command: `Remote-SSH: Connect to Host` and choose "Add new host" and enter `ssh root@192.168.178.123`. VSCode will prompt the question to which file settings should be added, we choose for the "user" settings file. Extend the setting to use key-authentication:
```
Host 192.168.178.123
  HostName 192.168.178.123
  User root
  IdentityFile ~/.ssh/id_rsa
```
After saving the SSH-settings file, we will again press CTRL-SHIFT-P and choose to "Remote-SSH: Connect to Host", but this time our host is selectable. We are now in a situation where VSCode is running on our local-machine but it's connected to the remote LXC container via SSH.

## Clone repository and DevContainers
By default TTeck script created a LXC container with a disk-size of 4gb. This is not enough to clone the GIT Repository, so it's needed to increase the size via the Proxmox GUI to at least 10Gb.

In VSCode we will now also install the VSCode devcontainer extension (`ms-vscode-remote.remote-containers`). After this we use CTRL-SHIFT-P and choose to "Dev Containers: Clone Repository in Named Container Volume". I choose for this option to not run into file permission issues. After selecting to clone the repository in a volume, you get asked to select which repository to clone. Here choose GitHub as source and then your fork of Home-Assistant.

After a while VSCode has finished to clone the repository and create the devcontainers. We will then run: `pip install -r requirements_all.txt` to just install all the packages Home Assistant might need.

## Overview setup
On the local machine (my laptop) i've VSCode running on Linux. We installed the VSCode Remote-SSH-extension to connect to the remote LXC-container via SSH and PubKey-authentication.

On the remote LXC-container we installed the VSCode DevContainers-extension. We cloned our own fork of Home Assistant repository in a Container Volume. Using the repository VSCode has created a devcontainer. In the devcontainer we installed all the Python packages via the PIP-command.

This setup allows us to also do development from other machines with VSCode installed to continue development. 
