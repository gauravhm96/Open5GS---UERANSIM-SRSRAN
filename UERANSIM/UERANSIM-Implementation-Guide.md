# Open5GS with UERANSIM Implementation

- This Guide includes all steps required to put the configured emulation into operation.
- Initially we implemented both 5Gcore and the UERANSIM in the same VM but after the [issue##310](https://github.com/aligungr/UERANSIM/issues/310). We used 2 seperate VM's like the figure shown below.

![image](https://user-images.githubusercontent.com/93492067/221634125-f98c0c55-0302-49ed-91df-8d1abaa30afa.png)

## Contents/Steps:

1. VirtualBox Installation.
2. Network settings to SSH.
3. How to install and SSH into the server?. 
4. Install Open5GS Core and Configuration.
5. Install WebUI
6. Setting up the Simulator,We’re using UERANSIM as our UE & RAN Simulator.
7. Adding the Subscriber.
8. Running the UE Simulator.
9. Test 5G Network.
10. Wireshark installation.
11. Trobleshooting.
12. Public Website Sources.

## VirtualBox Installation:

- Virtual Machine version - Oracle Virtual Box6.1 [Link to Download](https://www.virtualbox.org/wiki/Download_Old_Builds_6_1)
- Load a Linux image - Ubuntu Version - Ubuntu 20.04.5 LTS (Focal Fossa) Desktop. [Link to download](https://releases.ubuntu.com/focal/)
- Go for the minimal install.
- We have used these Versions for Better Compatibility and test Environment.

![image](https://user-images.githubusercontent.com/93492067/221420452-718e24e8-6ecd-4b7d-928d-7a51eeefd489.PNG)

![image](https://user-images.githubusercontent.com/93492067/221633754-f98eea68-a8c0-4bf0-acb2-d077bff18ee7.PNG)


## Network Settings before starting the VirtualMachine.

- Choose the virtualBox which is installed.
- Go to settings > NetworkSettings > enable the adapter 1 and 2.
- Select as per the images below.
- Start the VM.

![image](https://user-images.githubusercontent.com/93492067/221420838-f451a5de-6208-4cd8-9c24-ab2cf5744a06.PNG)


![image](https://user-images.githubusercontent.com/93492067/221420919-6f182941-730d-4626-8b6d-74acf1879543.PNG)


## How to install and SSH into the server?

It is better to install the ssh server, so it is easier to configure further (making it user friendly to use).

1. Install SSH on Ubuntu

```sh
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
```
2. Open command promt and ssh into the server.

```sh
ssh userName@Your-server-name-IP
ssh sandhya@192.168.56.116
```

![image](https://user-images.githubusercontent.com/93492067/221421646-bdf3140a-764f-40d0-863b-6218020a4963.PNG)

## Install Open5GS Core.

[Link to the official website](https://open5gs.org/open5gs/docs/guide/01-quickstart/)

1. Getting MongoDB (We have installed Mongodb 4.4 for compatibility)[Link](https://computingforgeeks.com/how-to-install-latest-mongodb-on-ubuntu/)

```console
sudo apt update
sudo apt install gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
```
```console
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
```
```console
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl enable --now mongod
systemctl status mongod
```
![image](https://user-images.githubusercontent.com/93492067/221422251-adcda809-0532-4d41-bdde-ea4479257112.PNG)

2. Ubuntu makes it easy to install Open5GS as shown below.

```console
$ sudo add-apt-repository ppa:open5gs/latest
$ sudo apt update
$ sudo apt install open5gs
```
- The first point of contact we’ll need to talk about is the AMF,

- The AMF – the Access and Mobility Function is reached by the gNodeB over the N2 interface. The AMF handles our 5G NAS messaging, which is the messaging used by the UEs / Devices to request data services, manage handovers between gNodeBs when moving around the network, and authenticate to the network.

- By default the AMF binds to a loopback IP, which is fine if everything is running on the same box, but becomes an issue for real gNodeBs or if we’re running UERANSIM on a different machine.

- This means we’ll need to configure our AMF to bind to the IP of the machine it’s running on, by configuring the AMF in /etc/open5gs/amf.yaml, so we’ll change the ngap addr to bind the AMF to the machine’s IP, for me this is 192.168.10.17.

```console
ngap:
  - addr: 192.168.10.17
```
![image](https://user-images.githubusercontent.com/93492067/221626017-702cbfc3-8dfa-4760-a217-678ecc61727c.PNG)

- Also, Edit the GTPU ip in UPF.yaml file

```console
gtpu:
  - addr: 192.168.10.17
```
![image](https://user-images.githubusercontent.com/93492067/221626606-f14912a0-e605-495b-a064-495c1e601144.PNG)

- Restart Both the services

```console
$ sudo systemctl restart open5gs-amfd
$ sudo systemctl restart open5gs-upfd
```
## Install the WebUI of Open5GS.

```console
 $ sudo apt update
 $ sudo apt install curl
 $ curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
 $ sudo apt install nodejs
```
```console
$ curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
```
```console
Username: admin
Password: 1432
```
![image](https://user-images.githubusercontent.com/93492067/221422760-aa346e50-63ef-4ef9-acf8-c9bba792699a.PNG)

##  Setting up the Simulator, We’re using UERANSIM as our UE & RAN Simulator.

[Link for reference](https://nickvsnetworking.com/my-first-5g-core-open5gs-and-ueransim/)

Install this in the VM2 

```console
$ sudo apt update 
$ sudo apt install make g++ libsctp-dev lksctp-tools iproute2 
$ sudo snap install cmake --classic
```
- We’ll clone the Github repository, move into it and make from source.
 
```console
$ git clone https://github.com/aligungr/UERANSIM
$ cd UERANSIM
$ make
```
Now we wait for everything to compile.

- Once we’ve got the software installed we’ll need to put together the basic settings.
- You should see these files in the /build/ directory and they should be executable.

![image](https://user-images.githubusercontent.com/93492067/221423911-dbc2c89a-6d9e-4e01-acff-6108502a572f.PNG)

## Configuring & Starting the gNodeB

- While we’re not actually going to bring anything “on air” in the RF sense, we’ll still need to configure and start our gNodeB.
  All the parameters for our gNodeB are set in the config/open5gs-gnb.yaml file,
  
- Inside here we’ll need to set the the parameters of our simulated gNodeB. for us this means (unless you’ve changed the PLMN etc) just changing the Link IPs that the gNodeB binds to, and the IP of the AMFs (for me it’s 192.168.10.15) 
 
 ![image](https://user-images.githubusercontent.com/93492067/221629508-ab1e8266-28c1-4150-9728-d7fce415dacb.PNG)
 
- Also, Edit the open5gs-ue.yaml file and modify gnbsearchList ip ( for me it is: 192.168.10.15 ) 
 
 ![image](https://user-images.githubusercontent.com/93492067/221629578-64fdb204-9c97-496a-bdd4-40d98ba60fa6.PNG)
  
- We’ll start the gNodeB service from the UERANSIM directory by running the nr-gnb service with the config file in config/open5gs-gnb.yaml

```console
$ build/nr-gnb -c config/open5gs-gnb.yaml
```
All going well you’ll see something like:

![image](https://user-images.githubusercontent.com/93492067/221632578-f2f74f09-3109-48fb-902a-b13f568c4b78.PNG)

And if you’re running Wireshark you should see the NG-AP (N2) traffic as well; Follow stepNo: to install wireshark

![image](https://user-images.githubusercontent.com/93492067/221644759-16d5e8b0-5d11-4c7a-8add-035df6c430f5.PNG)

![image](https://user-images.githubusercontent.com/93492067/221644884-0e2bd831-3f73-473a-aa2c-62f3ecace8ae.PNG)

If we tail the logs on the Open5GS AMF we should also see the connection too:

![image](https://user-images.githubusercontent.com/93492067/221632437-f904a04e-3eb3-4dcc-8528-04aa9b75e072.PNG)

## Adding the Subscriber in WebUI

- So we’ll browse to the web interface for Open5GS HSS/UDR and add a subscriber,
- We’ll enter the IMSI, K key and OP key (make sure you’ve selected OPc and not OP), and save. You may notice the values match the defaults in the Open5GS Web UI, just without the spaces.

![image](https://user-images.githubusercontent.com/93492067/221425228-f4225416-d4ea-4e3b-9311-649933a6d90f.PNG)

## Running the UE Simulator

We’ll leave the nr-gnb service running and open up a new terminal to start the UE with:

```console
sudo ./build/nr-ue -c config/open5gs-ue.yaml
```
Output Console

```console
UERANSIM v3.2.6
[2023-02-27 18:14:56.788] [nas] [info] Selected plmn[999/70]
[2023-02-27 18:14:58.138] [rrc] [info] Selected cell plmn[999/70] tac[1] category[SUITABLE]
[2023-02-27 18:14:58.138] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-02-27 18:14:58.138] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-02-27 18:14:58.138] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-02-27 18:14:58.138] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-02-27 18:14:58.138] [nas] [debug] Sending Initial Registration
[2023-02-27 18:14:58.138] [rrc] [debug] Sending RRC Setup Request
[2023-02-27 18:14:58.138] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-02-27 18:14:58.139] [rrc] [info] RRC connection established
[2023-02-27 18:14:58.139] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-02-27 18:14:58.139] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-02-27 18:14:58.156] [nas] [debug] Authentication Request received
[2023-02-27 18:14:58.164] [nas] [debug] Security Mode Command received
[2023-02-27 18:14:58.165] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-02-27 18:14:58.180] [nas] [debug] Registration accept received
[2023-02-27 18:14:58.180] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-02-27 18:14:58.180] [nas] [debug] Sending Registration Complete
[2023-02-27 18:14:58.181] [nas] [info] Initial Registration is successful
[2023-02-27 18:14:58.181] [nas] [debug] Sending PDU Session Establishment Request
[2023-02-27 18:14:58.181] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-02-27 18:14:58.390] [nas] [debug] Configuration Update Command received
[2023-02-27 18:14:58.414] [nas] [debug] PDU Session Establishment Accept received
[2023-02-27 18:14:58.421] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-02-27 18:14:58.569] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.7] is up.
```
![image](https://user-images.githubusercontent.com/93492067/221633466-29b3e9f8-91b4-4be3-8676-1b88eebc2984.PNG)

## Test 5G Network

- uesimtun0 is the default network interface that gets created automatically with running UERANSIM.
- You can find the PDU Session IP and TUN address in the UE logs:

```console
[2023-02-26 18:09:19.699] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.7] is up.
```
Open a new terminal
```console
./nr-binder 10.45.0.7 ping google.com
```
To confirm the ping from the uesimtun0 interface use this command
```console
ping -I uesimtun0 8.8.8.8
```
![image](https://user-images.githubusercontent.com/93492067/221639304-8423f5ac-7e5a-4464-bbdd-51ee31950eac.PNG)

In wireshark:
![image](https://user-images.githubusercontent.com/93492067/221645224-0af030f3-6dd8-487b-a1b2-a3c7b0a1412f.PNG)

## Wireshark Installation

Open Terminal
Add the Wireshark stable official PPA:
```console
sudo add-apt-repository ppa:wireshark-dev/stable
```
Update the repository:
```console
sudo apt update
```
Install Wireshark:-
```console
 sudo apt install wireshark
```
After Installation, Execute the command below so that non-root users can also capture the live packets.
```console
 sudo chmod +x /usr/bin/dumpcap
```
![image](https://user-images.githubusercontent.com/93492067/220407357-a2a3036e-9d56-4ac4-a44e-c815fd0fd735.PNG)

![image](https://user-images.githubusercontent.com/93492067/221424684-1a82580d-e9da-4b60-913c-894a84daecab.PNG)

## Troubleshooting:

### 1. How to remove a package:

- Get the package complete name:
```sh
dpkg --list | grep partial_package_name*
```
- Remove the package:
```sh
sudo apt-get remove package_name
```
- Remove all the dependencies:
```sh
sudo apt-get purge package_name
```
- Remove the unneeded packages that were once installed as a dependency:
```sh
sudo apt-get autoremove
```
- Remove the retrieved packages from the local cache:
```sh
sudo apt-get autoclean
```
- Check that it was completely removed:
```sh
dpkg --list | grep partial_package_name*
```
### 2. Errors while building UERANSIM

```Console
rm -fr logs # Old version log files
mkdir -p build
rm -fr build/*
CMake Error: The source directory "/home/mimo/UERANSIM-3.0.1/cmake-build-release" does not exist.
Specify --help for usage, or press the help button on the CMake GUI.
makefile:5: recipe for target 'build' failed
make: *** [build] Error 1
```
![image](https://github.com/FRA-UAS/mobcom-project-noobies/blob/22f9187d15c6c356318045ca447b483092f430f1/SandhyaBagadi/Documentation/Images%20for%20readme/Error%20while%20building.PNG)

- Remove cmake 
```sh
sudo apt remove cmake
```
```sh
sudo snap remove cmake
```
- Install Once again through classic and make it should work
```sh
snap install cmake --classic
```
### 3. SCTP could not connect: Connection refused.

![image](https://github.com/FRA-UAS/mobcom-project-noobies/blob/084a9f7a038f72ca7f1535e5d076739535ed2032/SandhyaBagadi/Documentation/Images%20for%20readme/Error%20while%20connecting%20sctp.PNG)

- This might be fixed with port forwarding in the global NAT configuration.

```sh
- Enable IPv4/IPv6 Forwarding
$ sudo sysctl -w net.ipv4.ip_forward=1
$ sudo sysctl -w net.ipv6.conf.all.forwarding=1

- Add NAT Rule
```sh
$ sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
$ sudo ip6tables -t nat -A POSTROUTING -s 2001:db8:cafe::/48 ! -o ogstun -j MASQUERADE
```
```sh
$ sudo ufw disable
Firewall stopped and disabled on system startup
$ sudo ufw status
Status: inactive
```
```sh
- Ensure that the packets in the `INPUT` chain to the `ogstun` interface are accepted
$ sudo iptables -I INPUT -i ogstun -j ACCEPT

- Prevent UE's from connecting to the host on which UPF is running
$ sudo iptables -I INPUT -s 10.45.0.0/16 -j DROP
$ sudo ip6tables -I INPUT -s 2001:db8:cafe::/48 -j DROP
```
![image](https://github.com/FRA-UAS/mobcom-project-noobies/blob/084a9f7a038f72ca7f1535e5d076739535ed2032/SandhyaBagadi/Documentation/Images%20for%20readme/NG%20Setup%20procedure%20is%20successful.PNG)

### 4. nr-binder Command Not Found.

```sh
chmod +x nr-binder
```
![image](https://github.com/FRA-UAS/mobcom-project-noobies/blob/084a9f7a038f72ca7f1535e5d076739535ed2032/SandhyaBagadi/Documentation/Images%20for%20readme/chmod%20to%20nrbinder.PNG)


## Public Website Sources:

- https://www.cyberciti.biz/faq/ubuntu-linux-install-openssh-server/
- https://computingforgeeks.com/how-to-install-latest-mongodb-on-ubuntu/
- https://nickvsnetworking.com/my-first-5g-core-open5gs-and-ueransim/
- https://open5gs.org/open5gs/docs/guide/01-quickstart/
- https://kkohls.org/guides_open5gs.html#machine-1-open5gs
- https://www.virtualbox.org/wiki/Download_Old_Builds_6_1
- https://askubuntu.com/questions/151941/how-can-you-completely-remove-a-package




 
