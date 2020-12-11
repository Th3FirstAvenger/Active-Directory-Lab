# Active Directory Lab

## Overview

The automation within this repository builds out a simple Active Directory lab with Packer and Vagrant. The first step is to use Packer to build a Windows Server 2019 base image. Then, a lab environment is created by Vagrant using the image output from Packer.

The lab consists of:
- DIR1: Domain Controller
- SRV1: Member Server
- SRV2: Member Server

You may modify the included *Vagrantfile* to add or remove servers within the environment.

## Login credentials

As with all Vagrant boxes, the default credentials for the domain Administrator user are:

```
Username: vagrant
Password: vagrant
```

## VirtualBox

1. Download and install VirtualBox: https://www.virtualbox.org/wiki/Downloads

## Packer

### Install Packer

1. Download Packer: https://www.packer.io/
2. Extract the downloaded zip
3. Copy the *packer.exe* file to C:\Windows\System32\

### Build the base image

1. Open Command Prompt as Administrator
2. Set the shell location to the *Active Directory Lab* directory
3. Run the base image build:

    ```
    packer build ./Packer/server-2019.json
    ```

The base image build can take a couple hours. First, it will download a Windows Server 2019 ISO (~5 GBs). Then it will create a VM, install Server 2019, install Windows Updates (~1 GB download), and then package the VM as a Vagrant box. The Windows Server 2019 ISO will be cached on computer so subsequent runs of the image build process will run much faster. 

## Vagrant

1. Download and install Vagrant: https://www.vagrantup.com/downloads.html
2. Install Vagrant plugin for reboot Windows VMs:

    ```
    vagrant plugin install vagrant-reload
    ```

3. Install Vagrant plugin for automatically installing VirtualBox guest additions (drivers, supporting software, etc.):

    ```
    vagrant plugin install vagrant-vbguest
    ```

### Build the lab

1. Open Command Prompt as an Administrator
2. Set the location to the *vagrant* directory in the *Active Directory Lab* directory
3. Build the lab:

    ```
    vagrant up
    ```

The first build of the lab will take ~30 minutes because VirtualBox needs to import the base box and prepare a parent VM. The individual servers in the lab are linked clones of the parent VM to save disk space. Once the parent VM has been imported, subsequent builds of the lab will be significantly faster because the servers are simply cloned from the parent VM.

### Other Vagrant commands

```
# Stop the lab
vagrant halt

# Start the lab
vagrant reload

# Destroy the lab
vagrant destroy
```