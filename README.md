# Arista Virtual Network using GCP and ContainerLab

This repository outlines the steps needed to create a virtual network with Arista cEOS container image running on a Virtual Instance in Google Cloud Platform (GCP) utilizing ContainerLab.

> **_NOTE:_**  It is not a requirement to run a VM in GCP.  You may use a VM of your choice that supports Docker and ContainerLab and skip steps 1 & 2.

<p align="center">
<img src="images/gcp-clab-v3.png" width="450">
</p>

## Step 1: Create GCP Account

You need a Gmail account and then you can activate your GCP account. Details on how to create your GCP Free Account can be found [here](https://cloud.google.com/).
Free $300 Credit to use for 90 days.  No autocharge after free trial ends.

Typical VM Usage Fee is:  $0.25/hr (8 vCPUs 32GM RAM)

Disk and Public Static IP Address - additional cost (but minimal)

## Step 2: Create VM Instance

From GCP [Console](https://console.cloud.google.com/), go to the Compute Engine section.  If this is your first time creating a VM instance in GCP, you will need to enable Compute Engine API.  This takes a few seconds to complete.

<p align="center">
<img src="images/compute_api.png" width="200">
</p>

Once complete, you will be able to add a new VM Instance by clicking `CREATE INSTANCE`.  Use the follow instance attributes to create a VM capable of sufficiently running several cEOS instances for your ContainerLab environiment.  Detailed instructions to create a VM Instance can be found [here](https://cloud.google.com/compute/docs/create-linux-vm-instance).
### 2.1 - Create VM Instance with the following attributes:

| Parameter | Value |
| --------- | ----- |
| Name | < hostname of VM > |
| Region | < your choice > |
| Zone | < your choice > |
| Machine Series | E2 |
| Machine type | e2-standard-8 (8vCPUs and 32GB RAM) |
| Boot Disk | |
| &nbsp;&nbsp;&nbsp;&nbsp; Operating System | Ubuntu |
| &nbsp;&nbsp;&nbsp;&nbsp; Version | 20.04 LTS |
| &nbsp;&nbsp;&nbsp;&nbsp; Boot disk type | Balanced persistent disk |
| &nbsp;&nbsp;&nbsp;&nbsp; Size (GB) | 20 |
| Firewall | |
| &nbsp;&nbsp;&nbsp;&nbsp; Allow HTTP Access | Yes |

Now click `CREATE` at the bottom.
### 2.2 Add SSH Keys fort Authentication

After Instance boots, add your Public SSH Key to the VM Instance.  In GCP, edit your VM instance and scroll down to the `Security and Access` section.  Click `ADD ITEM` to add your ssh key.  Then click `SAVE` at the bottom.

<p align="center">
<img src="images/ssh-keys-v2.png" width="450">
</p>

- Mac:
  - Use existing keys or generate a new key pair with ssh-keygen
  - `ssh-keygen -t rsa`
  - Detailed instructions can be found [here](https://www.digitalocean.com/community/tutorials/how-to-create-ssh-keys-with-openssh-on-macos-or-linux)
  - Add Public Key to Host
  - Connect to Host using favorite SSH client

- Windows (Putty):
  - Use Puttygen to create Key Pair
  - Detailed instructions can be found [here](https://www.ssh.com/academy/ssh/putty/windows/puttygen)
  - Add Public Key to the Host
  - Connect to Host via Putty using Private Key
### 2.3 - Connect to VM.

Now SSH to VM instance using the assigned public IP address.  Note you can make this a static address if desired.

## Step 3: Install Docker
Below is a simplified one liner bash script that will install docker to your Linux VM.

```bash
bash -c "$(curl http://www.packetanglers.com/installdocker.sh)"
```

> **_NOTE:_** Logout and log back in to enable sudo permissions to Docker.


## Step 4: Download cEOS Image and import into Docker
The following 2 commands will download an Arista cEOS Container image file and then import it into Docker.

```bash
curl http://www.packetanglers.com/images/cEOS-lab-4.27.3F.tar -o cEOS-lab-4.27.3F.tar
```

Now import this image into Docker - takes approximately 30 secs.  Be patient.

```bash
docker import cEOS-lab-4.27.3F.tar ceos:4.27.3F
```
## Step 5: Install ContainerLab

This is a one line install script. It will detect the OS

```bash
bash -c "$(curl -sL https://get-clab.srlinux.dev)"
```

## Step 6: Clone Example L2LS ContainerLab Topology Repo

```bash
git clone https://github.com/PacketAnglers/containerlab.git
```

## Step 7: Start ContainerLab

Below is a simple L2LS Topology that is part of the repo you just cloned.

<p align="center">
<img src="images/l2ls-topo.png" width="700">
</p>

```bash
sudo clab deploy -t containerlab/topologies/L2LS/L2LS.yaml --reconfigure
```

## Step 8: Connect to your ContainerLab
Point your browser to the the public IP of your VM at this URL:

http://< VM public ip >/graphite

<p align="center">
<img src="images/topology.png" width="450">
</p>

Click on SPINE1 to connect via a web/ssh session (IPv4/SSH) link.

<p align="center">
<img src="images/spinepopup.png" width="200">
</p>

Login with credentials:

- `username:` admin
- `password:` admin

<p align="center">
<img src="images/websession.png" width="300">
</p>

Connect to HostA and try pinging HostB.

```bash
admin@HOSTA:~$ ping 10.20.20.100
```

## Step 9: Destroy Lab

When you are done using your lab, run the destroy command to shutdown the lab clean up docker containers.

```bash
sudo clab destroy -t containerlab/topologies/L2LS/L2LS.yaml
```

## Step 10: Shutdown VM

From GCP Console, make sure you shutdown your VM when you are not using it.  You also can create Instance Schedules to ensure VMs are shutdown at a certain time of day.

> **_NOTE:_** This works as well. `sudo shutdown -h now`

# Next Steps

- Try modifying the topology file to add more nodes and connections.
- You can save/update startup configs to match your environment.