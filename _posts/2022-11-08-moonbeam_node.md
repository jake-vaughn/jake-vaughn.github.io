---
title: Fast Moonbeam Node Setup   
date: 2022-11-08 8:00:00 +/-1111
author: Jake Vaughn
categories: [Guide]
tags: [moonbeam, node, para-chain, fast, server, guide]
---

# Overview:
This is a guide on how to quickly setup and sync a Moonbeam node on a hosted server. Running a node on a Moonbeam-based network allows you to connect to the network, sync with a bootnode, obtain local access to RPC endpoints, and more. In this tutorial we will be using systemd to set up a service to run the node beacause we are going to be using ubuntu linux. If you want to or need to run a different OS know that you can run the node with docker just as easily.
Topics in order are
- [Creating a Hetzner Server](#creating-a-hetzner-server)
- [Enabling SwapFile (optional)](#enabling-swapfile-optional)
- [Downloading Snapshot (optional)](#downloading-snapshot-optional)
- [Service and Binary Setup](#service-and-binary-setup)
- [Configuration](#configuration)
- [Running the Service](#running-the-service)

# Creating a Hetzner Server
If you want to run your node on a server then Hetzner is your best bet. There are other options out there like Digital Ocean and Vultr, but neither of them get to the price point that Hetzner provides. 

First Create an account at [Hetzner](https://accounts.hetzner.com/signUp). Then login at [hetzner.cloud](https://console.hetzner.cloud/projects) and make sure to bookmark this page. Go ahead and create a new project naming it whatever you like. Once it is created click on it and then click on ADD SERVER. In the options pick any location you want and Ububuntu for the Image.

![Server Creation 1](/images/MoonbeamNode/createServer1.png)

Next pick type standard and the option CPX41 that has 240 GB of storage.

![Server Creation 2](/images/MoonbeamNode/createServer2.png)

Current requirements for a node based on the [moonbeam docs](https://docs.moonbeam.network/node-operators/networks/run-a-node/overview/#requirements) say that we need a 8 Core CPU, 16 GB Ram, and a 1 TB SSD. These requirements are a bit over the top as they are intended for Validators/Collators and are future proof. The main minimum spech we need to worry about is storage. As of writing this post a Full node on moonbeam currently needs roughly 200 GB of SSD storage (160 GB if you don't need to full block history). This storage requirement will grow with time so keep that in mind. For now we can use the CPX41 option as it meets the reqirements, but most importantly the storage.

> Node: Going under the 8 Core and 16 GB Ram requirements will work fine for a noncritical node and will only result in slower sync time. However it is not recomended for Validators/Collators.

Next create and add your SSH key. If you do not know how I sugest using ssh-keygen. An article explaining ["How to Use ssh-keygen to Generate a New SSH Key?"](https://www.ssh.com/academy/ssh/keygen).

![Add SSH](/images/MoonbeamNode/addSSH.png)

Once the SSH key is added leave everything else default and click CREATE & BUY NOW

![Server Creation 3](/images/MoonbeamNode/createServer3.png)

With the server created we can now login using ssh. On linux it would look something like this 

```bash
ssh root@<SERVER_IP>
```

You should then get a response as such.

![SSH Login](/images/MoonbeamNode/sshLogin.png)

Once logged in I recomend updating the server using `apt-get update` and `apt-get upgrade` rebooting if necissary.

# Enabling SwapFile (optional)
Creating a linux SwapFile is not nessisary but is a nice to have if you are running other memory intensive programs on your server. Using swap is a very useful way to extend the RAM because it provides the necessary additional memory when the RAM space has been exhausted and a process has to be continued. If you want to create a swap file run the folowing commands.

```bash
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```
then append the following line to /etc/fstab

`/swapfile swap swap defaults 0 0`

To verify that the swap file is active use

```bash
sudo swapon --show
```

It should result in something like the following.

```console
NAME      TYPE  SIZE   USED PRIO
/swapfile file 1024M 507.4M   -1
```

# Downloading Snapshot (optional)
Something that can GREATLY increase sync time is by downloading an already synced snapshot of either the moonbeam parachain or the relay chain or both. If you need your node up quick or don't want to bother waiting this is your best option. My current recomendation is to use [Certhum's backups](https://www.certhum.com/moonbeam-databases) as they are updated reguarly. If you want to sync the node from scratch skip down to the section [Service and Binary Setup](#service-and-binary-setup).


Now Because the files are quite large and our drive is roughly 240 GB we will not be able to download and unzip the files without running out of storage. The workaround for this is attaching a volume to our server and downloading the backup there. In Hetzner click on the server you created and go to the VOLUMES tab on the left. Then click Create Volume and set the storge to 300 GB and name it volume1. Now click CREATE & BUY NOW. Don't worry about the cost, we will be deleting the volume after we download and unzip the backups.

![Create Volume](/images/MoonbeamNode/createVolume.png)

After the volume is created go back to where you logged into the server and configure the volume by following the commands given in the settings of the volume on Hetzner.

![Volume Config](/images/MoonbeamNode/configVolume.png)

This mounts the volume at `/mnt/volume1` where we will download the backups. To download the most recent backup use


<u> Moonbeam Relay Chain (Polkadot pruning 1000) Backup </u> 

Download the backup to volume1. This will take time depending on your network speed.

```bash
wget -O /mnt/volume1/moonbeam-polkadot-backup.tar.zst https://db.certhum.com/moonbeam-polkadot-backup.tar.zst
```
> Note: If the download fails or gets canceled reenter the command with the -c flag to continue the download from where it left off.

![wgetBackup](/images/MoonbeamNode/wgetBackup.png)

next download the sst

```bash
wget -O /mnt/volume1/moonbeam-polkadot-sst.dict https://db.certhum.com/moonbeam-polkadot-sst.dict
```

install zstd (ZSTD lets you specify how many processors you want to assign to the job)

```bash
sudo apt install zstd
```

Create the directory for the database in var/lib

```bash
sudo mkdir -p /var/lib/moonbeam-data/polkadot/chains/polkadot/db
```

and finaly unzip the backup to the database using

```bash
sudo tar -I 'zstd -vd -T0 -D /mnt/volume1/moonbeam-polkadot-sst.dict' -xvf /mnt/volume1/moonbeam-polkadot-backup.tar.zst -C /var/lib/moonbeam-data/polkadot/chains/polkadot/db --strip-components=7
```

This should take some time wait until it is finnished.


<u> Moonbeam Parachain (state-pruning=archive) RocksDB Backup </u> 

The process for downloading the Moonbeam Parachain is almost the same as the Relay Chain so I won't go into as much detail.

```bash
wget -O /mnt/volume1/moonbeam-backup-archive.tar.zst https://db.certhum.com/moonbeam-backup.tar.zst
wget -O /mnt/volume1/moonbeam-sst.dict https://db.certhum.com/moonbeam-sst.dict
sudo mkdir -p /var/lib/moonbeam-data/chains/moonbeam/db
sudo tar -I 'zstd -vd -T0 -D /mnt/volume1/moonbeam-sst.dict' -xvf /mnt/volume1/moonbeam-backup-archive.tar.zst -C /var/lib/moonbeam-data/chains/moonbeam/db --strip-components=6
```

Now that we have our backups unziped we can delete the volume so that we don't have to pay monthly for it. To delete the volume first detatch the volume and then delete it.

![Detatch Volume](/images/MoonbeamNode/detatchVolume.png)

![Delete Volume](/images/MoonbeamNode/deleteVolume.png)


⚠️ Make Sure the Volume is Deleted or you will be charged monthly for it! ⚠️

# Service and Binary Setup
The following commands will set up everything regarding running the service.

1. Create a service account to run the service:
```bash
adduser moonbeam_service --system --no-create-home
```
2. Create a directory to store the binary and data:
```bash
sudo mkdir /var/lib/moonbeam-data
```
3. Download the Latest [Moonbeam Binary](https://github.com/PureStake/moonbeam/releases):
```bash
sudo wget -O /var/lib/moonbeam-data/moonbeam https://github.com/PureStake/moonbeam/releases/latest/download/moonbeam
```
4. Make sure you set the ownership and permissions accordingly for the local directory that stores the chain data:
```bash
sudo chmod +x moonbeam
sudo chown -R moonbeam_service /var/lib/moonbeam-data
```

# Configuration

> Note: Full nodes or "archive" nodes keep the full history of the chain and require a bit more space. In this case roughly 40GB more.

# Running the Service