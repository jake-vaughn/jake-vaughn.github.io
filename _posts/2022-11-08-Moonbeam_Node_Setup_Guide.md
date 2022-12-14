---
title: Moonbeam Node Setup Guide
date: 2022-11-08 8:00:00 +/-1111
author: Jake Vaughn
categories: [Guides]
tags: [moonbeam, node, para-chain, server, guide, setup]
---

# Overview:
This is a guide on how to quickly setup and sync a Moonbeam node on a hosted server. Running a node on a Moonbeam-based network allows you to connect to the network, sync with a bootnode, obtain local access to RPC endpoints, and more. In this tutorial, we will be using systemd to set up a service to run the node because we will be using Ubuntu Linux. If you want to or need to run a different OS know that you can run the node with Docker just as easily.

Topics in order are
- [Creating a Hetzner Server](#creating-a-hetzner-server)
- [Enabling SwapFile (optional)](#enabling-swapfile-optional)
- [Downloading Snapshot (optional)](#downloading-snapshot-optional)
  - [Moonbeam Relay Chain Backup](#moonbeam-relay-chain-backup)
  - [Moonbeam Parachain RocksDB Backup](#moonbeam-parachain-rocksdb-backup)
- [Service and Binary Setup](#service-and-binary-setup)
- [Configuration File](#configuration-file)
- [Running the Service](#running-the-service)
- [Telemetry](#telemetry)

# Creating a Hetzner Server
If you want to run your node on a server then Hetzner is your best bet. There are other options out there like Digital Ocean and Vultr, but neither of them gets to the price point that Hetzner provides.

First Create an account at [Hetzner](https://accounts.hetzner.com/signUp). Then login at [hetzner.cloud](https://console.hetzner.cloud/projects) and bookmark this page. Go ahead and create a new project naming it whatever you like. Once it is created click on it and then click on ADD SERVER. In the options pick any location you want and Ubuntu for the Image.

![Server Creation 1](/images/MoonbeamNode/createServer1.png)
Next pick the type standard and the option CPX41 which has 240 GB of storage.

![Server Creation 2](/images/MoonbeamNode/createServer2.png)

Current requirements for a node based on the [moonbeam docs](https://docs.moonbeam.network/node-operators/networks/run-a-node/overview/#requirements) say that we need an 8 Core CPU, 16 GB Ram, and a 1 TB SSD. These requirements are a bit over the top as they are intended for Validators/Collators and are future-proof. The main minimum specs we need to worry about is storage. As of writing this post a Full node on moonbeam currently needs roughly 200 GB of SSD storage (160 GB if you don't need to full block history). This storage requirement will grow with time so keep that in mind. For now, we can use the CPX41 option as it meets the requirements, but most importantly the storage.

> Node: Going under the 8 Core and 16 GB Ram requirements will work fine for a noncritical node and will only result in slower sync time. However, it is not recommended for Validators/Collators.

Next, create and add your SSH key. If you do not know how I suggest using ssh-keygen. An article explaining ["How to Use ssh-keygen to Generate a New SSH Key?"](https://www.ssh.com/academy/ssh/keygen).

![Add SSH](/images/MoonbeamNode/addSSH.png)

Once the SSH key is added leave everything else default and click CREATE & BUY NOW

![Server Creation 3](/images/MoonbeamNode/createServer3.png)

With the server created we can now login using ssh. On Linux, it would look something like this

```bash
ssh root@<SERVER_IP>
```
You should then get a response as such.

![SSH Login](/images/MoonbeamNode/sshLogin.png)

Once logged in I recommend updating the server using??
```bash
sudo apt update && sudo apt upgrade
```
 and rebooting if necessary.

# Enabling SwapFile (optional)

Creating a Linux SwapFile is not necessary but is nice to have if you are running other memory-intensive programs on your server. Using a swap is a very useful way to extend the RAM because it provides the necessary additional memory when the RAM space has been exhausted and a process has to be continued. If you want to create a swap file run the following commands.

```bash
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Open the /etc/fstab file
```bash
sudo nano /etc/fstab
```

add the following to the end of the file

```text
/swapfile swap swap defaults 0 0
```
Save the file with `Ctrl+s` and close it with `Ctrl+x`

To verify that the swap file is active use
```bash
sudo swapon???-???show
```

It should result in something like the following.
```console
NAME TYPE SIZE USED PRIO
/swapfile file 1024M 507.4M -1
```

# Downloading Snapshot (optional)
Something that can GREATLY increase sync time is downloading an already synced snapshot of either the moonbeam para-chain, the relay chain, or both. If you need your node up quickly or don't want to bother waiting this is your best option. My current recommendation is to use [Certhum's backups](https://www.certhum.com/moonbeam-databases) as they are updated regularly. If you want to sync the node from scratch skip down to the section [Service and Binary Setup](#service-and-binary-setup).

Now Because the files are quite large and our drive is roughly 240 GB we will not be able to download and unzip the files without running out of storage. The workaround for this is attaching a volume to our server and downloading the backup there. In Hetzner click on the server, you created and go to the VOLUMES tab on the left. Then click Create Volume and set the storage to 300 GB and name it volume1. Now click CREATE & BUY NOW. Don't worry about the cost, we will be deleting the volume after we download and unzip the backups.

![Create Volume](/images/MoonbeamNode/createVolume.png)

After the volume is created go back to where you logged into the server and configure the volume by following the commands given in the settings of the volume on Hetzner.

![Volume Config](/images/MoonbeamNode/configVolume.png)

This mounts the volume at `/mnt/volume1` where we will download the backups. To download the most recent backup use

## Moonbeam Relay Chain Backup

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

and finally unzip the backup to the database using
```bash
sudo tar -I 'zstd -vd -T0 -D /mnt/volume1/moonbeam-polkadot-sst.dict' -xvf /mnt/volume1/moonbeam-polkadot-backup.tar.zst -C /var/lib/moonbeam-data/polkadot/chains/polkadot/db???-???strip-components=7
```

This should take some time so wait until it is finished.

## Moonbeam Parachain RocksDB Backup

The process for downloading the Moonbeam Parachain is almost the same as the Relay Chain so I won't go into as much detail.

```bash
wget -O /mnt/volume1/moonbeam-backup-archive.tar.zst https://db.certhum.com/moonbeam-backup.tar.zst
```

```bash
wget -O /mnt/volume1/moonbeam-sst.dict https://db.certhum.com/moonbeam-sst.dict
```

```bash
sudo mkdir -p /var/lib/moonbeam-data/chains/moonbeam/db
```

```bash
sudo tar -I 'zstd -vd -T0 -D /mnt/volume1/moonbeam-sst.dict' -xvf /mnt/volume1/moonbeam-backup-archive.tar.zst -C /var/lib/moonbeam-data/chains/moonbeam/db???-???strip-components=6
```

**Delete Volume**

Now that we have our backups unzipped we can delete the volume so that we don't have to pay monthly for it. To delete the volume first detach the volume and then delete it.

![Detatch Volume](/images/MoonbeamNode/detatchVolume.png)

Once detached you need to go to the left side tab click on Volumes and then again go to setting add click Delete

![Delete Volume](/images/MoonbeamNode/deleteVolume.png)

?????? Make Sure the Volume is Deleted or you will be charged monthly for it! ??????

# Service and Binary Setup

The following commands will set up everything regarding running the service.

1. Create a service account to run the service:

```bash
adduser moonbeam_service???-???system???-???no-create-home
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

# Configuration File

The next step is to create the systemd configuration file. Depending on your needs there are a few different options between creating a full node or a pruning node. If you used the backups above you will need to have state-pruning set to archive. If you want a pruned node you can remove `???-???state-pruning=archive` from the config file and the node will default to pruning the state to the last 256 blocks. More info about flags can be found [here](https://docs.moonbeam.network/node-operators/networks/run-a-node/flags/).

> Note: Full nodes a.k.a "archive" nodes keep the full history of the chain and require a bit more space than pruned state nodes. In this case roughly 40GB.

Create the config file at /etc/systemd/system/moonbeam.service

```bash
sudo nano /etc/systemd/system/moonbeam.service
```

Add the following to the file changing YOUR-NODE-NAME to a name you want.

```text
[Unit]
Description="Moonbeam systemd service"
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=moonbeam_service
SyslogIdentifier=moonbeam
SyslogFacility=local7
KillSignal=SIGHUP
ExecStart=/var/lib/moonbeam-data/moonbeam \
     --port 30333 \
     --rpc-port 9933 \
     --ws-port 9944 \
     --execution wasm \
     --wasm-execution compiled \
     --state-pruning=archive \
     --trie-cache-size 0 \
     --base-path /var/lib/moonbeam-data \
     --chain moonbeam \
     --name "YOUR-NODE-NAME" \
     -- \
     --port 30334 \
     --rpc-port 9934 \
     --ws-port 9945 \
     --execution wasm \
     --state-pruning 1000 \
     --name="YOUR-NODE-NAME (Embedded Relay)"

[Install]
WantedBy=multi-user.target
```

Save the file with `Ctrl+s` and close it with `Ctrl+x`

# Running the Service

The last thing to do is enable and run the service.

```bash
systemctl enable moonbeam.service
systemctl start moonbeam.service
```

Check the output logs from the node with
```bash
journalctl -f -u moonbeam.service
```
Your node should now be running and should start back up if you ever reboot the system. Depending on if you downloaded the optional backups complete syncing of the node should take around 2-3 days.

If you ever need to stop the service do it with

```bash
systemctl stop moonbeam.service
```

and if you want to disable the service from starting on reboot

```bash
systemctl disable moonbeam.service
```

# Telemetry
View your nodes telemetry by searching the name of your node at [Polkadot's Telemetry dashboard](https://telemetry.polkadot.io/#list/0xfe58ea77779b7abda7da4ec526d14db9b1e9cd40a217c34892af80a9b332b76d).
