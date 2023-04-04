# Implementation of 5G SA using srsRAN
The objective is to develop a 5G-SA (stand alone) End-to-End system using Open5GS (Open5GS, 2022) and srsRAN. To implement the network, Open5gs is used to develop the 5G packet core network functionalities and srsRAN is used as the 5G-RAN and ZeroMQ networking library is used as a virtual radio (to support srsUE and srseNodeB)

The following network architecture (regarding srsRAN part) has been implemented

![image](https://user-images.githubusercontent.com/74201353/221945923-0ca61dae-28ad-4574-a161-953772fb5055.png)

Here comes the step by step implemetation guide 
## 1. Setting Up Virtual Environment
Specifications used
 1. Virtual Machine version - Oracle Virtual Box 7.0.6v 
 2. Ubuntu Version - Ubuntu 18.04.6 LTS (Bionic Beaver)
 3. nodejs version - 17
 
 Oracle Virtual Box 7.0.6v is used to set up the virtual environment to implement the project within a linux based distribution.
 Link to Download VirtualBox 7.0.6 -  [VirtualBox Download](https://www.virtualbox.org/wiki/Downloads)
 ![image](https://user-images.githubusercontent.com/74201353/221549908-0782a4dc-c2b6-4dc8-b8ab-a9f7ca143792.png)
 
 After installing oracle virtual box, a linux based distribution needs to be downloaded and install to implement the desired network.
 In this project, Ubuntu 18.04.6 LTS (Bionic Beaver) is used. Later versions ca also be used. [Ubuntu 18.04.6 LTS (Bionic Beaver) Download](https://releases.ubuntu.com/18.04/)
 
![image](https://user-images.githubusercontent.com/74201353/221551331-9b7d6a65-397d-4e54-9a38-c15bff28e0b7.png)

## 2. Installing and Configuring open5gs
### 1. Installing MongoDB
Before installing open5gs, Mongodb needs to be installed first. NRF/PCF/UDR and PCRF/HSS both use MongoDB as their storage of choice. MongoDB version 4.0 is installed for our project.
Add the MongoDB GPG key
```sh
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 –recv 9DA31620334BD75D9DCB49F368818C72E52529D4
```
After importing the key, MongoDB repository needs to be added
```sh
sudo add-apt-repository 'deb [arch=amd64] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse'
```
Then mongodb-org meta package is installed after updating the packages list
```sh
sudo apt install mongodb-org
```
Starting the MongoDB daemon and setting it to automatically launch at system boot will finish the installation
```sh
sudo systemctl start mongod
sudo systemctl enable mongod
```
![image](https://user-images.githubusercontent.com/74201353/221555029-5f579117-fc0d-4d55-94b1-e300c2b9d206.png)

### 2 (i). Installation - Open5GS  (Install as Package) 

```sh
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs
```

### 2 (ii). Installation - Open5GS (From GitHub Repository) 

Install the dependencies for building the source code.

In this project, the open5gs is built from the source code

To install all the packages individually 

```sh
sudo apt install python3-pip
sudo apt install python3-setuptools 
sudo apt install python3-wheel 
sudo apt install ninja-build 
sudo apt install build-essential 
sudo apt install flex 
sudo apt install bison 
sudo apt install git 
sudo apt install cmake 
sudo apt install libsctp-dev 
sudo apt install libgnutls28-dev 
sudo apt install libgcrypt-dev 
sudo apt install libssl-dev 
sudo apt install libidn11-dev 
sudo apt install libmongoc-dev 
sudo apt install libbson-dev 
sudo apt install libyaml-dev 
sudo apt install libnghttp2-dev 
sudo apt install libmicrohttpd-dev 
sudo apt install libcurl4-gnutls-dev 
sudo apt install libnghttp2-dev 
sudo apt install libtins-dev 
sudo apt install libtalloc-dev 
sudo apt install meson
```

To Install Packages in one shot

```sh

sudo apt install python3-pip python3-setuptools python3-wheel ninja-build build-essential flex bison git cmake libsctp-dev libgnutls28-dev libgcrypt-dev libssl-dev libidn11-dev libmongoc-dev libbson-dev libyaml-dev libnghttp2-dev libmicrohttpd-dev libcurl4-gnutls-dev libnghttp2-dev libtins-dev libtalloc-dev meson

```
 
Git Repository clone Open5GS.

```sh
git clone https://github.com/open5gs/open5gs
```
compile with meson

```sh
cd open5gs
meson build --prefix=`pwd`/install
ninja -C build
```
 Run all test programs as below.

```sh
cd build
meson test -v
```
![image](https://user-images.githubusercontent.com/74201353/221556456-76e4a310-b48c-4fdd-945d-c8d8779aa295.png)
After passing all the test the installation process is done by the following.

```sh
cd build
ninja install
```
### 3. Install the WebUI of Open5GS

The WebUI allows you to interactively edit subscriber data. While it is not essential to use this, it makes things easier when you are just starting out on your Open5GS adventure. (A command line tool is available for advanced users).

```sh
 sudo apt update
 sudo apt install curl
 curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
 sudo apt install nodejs
 ```
 
```sh
 curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
```
![image](https://user-images.githubusercontent.com/74201353/221557435-8ba6f3c0-e474-41b4-ba2d-5f03b760d3dc.png)

### 4. Creating TUN Device 
A TUN interface is created named ogstun.

```sh
sudo ip tuntap add name ogstun mode tun
sudo ip addr add 10.45.0.1/16 dev ogstun
sudo ip addr add 2001:db8:cafe::1/48 dev ogstun
sudo ip link set ogstun up
```
### 5. Creating a bridge between the Internet and 5G core 
To have a bridge between the Internet (Data network) and 5G core UPF, NAT port forwarding is enabled. After forwarding NAT rule to the IP Tables is added.  
```sh
 sudo sysctl -w net.ipv4.ip_forward=1
 sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
 sudo iptables -I INPUT -i ogstun -j ACCEPT
 ```
 ### 6. Register Subscriber Information
 There are two ways of registering subscriber information. One is from the WebUI and other is directly from the terminal. In this project subscriber information is added directly from the terminal.
 #### i.Register Subscriber Information (WebUI)

Connect to
```sh 
http://127.0.0.1:3000
``` 

and login with admin account.

Username : admin
Password : 1423

To add subscriber information, you can do WebUI operations in the following order:

Go to Subscriber Menu.
Click + Button to add a new subscriber.
Fill the IMSI, security context(K, OPc, AMF), and APN of the subscriber.
Click SAVE Button

##### Note: Subscribers added with this tool immediately register in the Open5GS HSS/UDR without the need to restart any daemon. However, if you use the WebUI to change subscriber profile, you must restart the Open5GS AMF/MME daemon for the changes to take effect.

 #### (ii) Register Subscriber Information (Terminal)

Open another terminal and add Subscribers

```sh
./open5gs-dbctl add 901700123456789 00112233445566778899aabbccddeeff 000102030405060708090a0b0c0d0e0f 
```
![image](https://user-images.githubusercontent.com/74201353/221562109-92b20c2c-ab40-4658-b8bf-6adb52bc2107.png)
![image](https://user-images.githubusercontent.com/74201353/221562191-30e08969-729c-4359-ad54-004b0fdbae9f.png)
![image](https://user-images.githubusercontent.com/74201353/221562257-04b67031-43fd-4581-8559-731043ffbde3.png)

 ### 7. Configuring Open5gs
 sample.yaml is configured to configure the AMF and UPF of 5G core. AMF is configured to bind to the IP of the machine it’s running on. UPF is configured by setting the PLMN (MCC & MNC) and MME address according to the requirement of srsenb config.

![image](https://user-images.githubusercontent.com/74201353/221997079-c1a4e4ac-fb3b-4449-ba7e-5e6b94907fbb.png)
![image](https://user-images.githubusercontent.com/74201353/221996546-fb5351ff-e70f-40ca-89d2-055786186d86.png)
![image](https://user-images.githubusercontent.com/74201353/221996792-848f1514-4491-4f05-b6d7-54848369c499.png)
![image](https://user-images.githubusercontent.com/74201353/221997016-91f7fac4-c231-4617-9984-71115c907b60.png)


## 3. Installing srsRAN with ZMQ Virtual Radios
srsRAN is a 4G and 5g software radio suite. The 4G network consists of a core network, an eNodeB, and a UE implementation. Usually eNodeB and UE are used with physical radios for over-the-air transmissions. However, the srsRAN software suite also includes a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications. This approach is very useful for development, testing, debugging, CI/CD or for teaching and demonstrating.

### 1. ZeroMQ installation
1. First thing is to install ZeroMQ and build srsRAN. Before that, the required library for srsRAN on Ubuntu is installed by the following command
```sh
sudo apt-get install build-essential cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev
```
After that install ZMQ
```sh
sudo apt-get install libzmq3-dev
```
2. libzmq installation
```sh
git clone https://github.com/zeromq/libzmq.git
cd libzmq
./autogen.sh
./configure
make
sudo make install
sudo ldconfig
```
![image](https://user-images.githubusercontent.com/74201353/221567559-af42e75b-8dae-4fae-b426-ba9fe3882409.png)

3. czmq installation
```sh
git clone https://github.com/zeromq/czmq.git
cd czmq
./autogen.sh
./configure
make
sudo make install
sudo ldconfig
```
![image](https://user-images.githubusercontent.com/74201353/221567624-7528f4f6-d756-4843-a511-bef01b455d52.png)

### 2. srsRAN building and Installation
1. Finally download and build srsRAN
```sh
git clone https://github.com/srsRAN/srsRAN.git
cd srsRAN
mkdir build
cd build
cmake ../
make
make test
```
![image](https://user-images.githubusercontent.com/74201353/217069131-b5e6c62a-8a14-4b98-a3fa-0c62d3198ff2.png)

2. srsRan installation
```sh
sudo make install
srsran_install_configs.sh user
```

## 4. Configuring srsRAN
After configuring open5gs, srsUE and srsENB need to be configured.
### 1. srsUE configuration
The Following steps need to be taken to modify the srsUE config file to enable 5G SA features:

Enable ZMQ
Enable NR features
Configure USIM credentials & APN
#### 1. Enable ZMQ
1. Firstly, the ZMQ is enabled by configuring the ue.conf . While configuring the ZMQ sample rate, it needs to be ensured that same sample rate is used for 5G SA too.
```sh
[rf]
freq_offset = 0
tx_gain = 80
srate = 11.52e6

device_name = zmq
device_args = tx_port=tcp://*:2001,rx_port=tcp://localhost:2000,id=ue,base_srate=11.52e6
```
![image](https://user-images.githubusercontent.com/74201353/221575789-08e0d071-5b7a-495b-899a-2c842a66bb25.png)
![image](https://user-images.githubusercontent.com/74201353/221575869-930d506f-b541-49ab-8a41-3c98305d7140.png)
2. Configuring the network namespace 
```sh
[gw]
netns = ue1
```
![image](https://user-images.githubusercontent.com/74201353/221576247-ab906ca1-733d-41a5-ac4d-33d9778e33cb.png)

#### 2. Enable NR features
1. Firstly disable the LTE carrier
```sh
[rat.eutra]
dl_earfcn = 2850
nof_carriers = 0
```
![image](https://user-images.githubusercontent.com/74201353/221577162-0a44fc34-5589-45f7-b2e9-368dba67d99e.png)

2. Enable the NR band and carrier
```sh
[rat.nr]
bands = 3,78
nof_carriers = 1
```
![image](https://user-images.githubusercontent.com/74201353/221577445-dfb5a9ce-a609-4fa4-892c-895a1d40385f.png)
3. Set the release to release-15
```sh
[rrc]
release = 15
```
![image](https://user-images.githubusercontent.com/74201353/221577670-9a04721b-2c7f-45d9-b10d-7774ed0c66c0.png)

#### 3. Configure USIM credentials & APN
1. The USIM credentials are changed according to the UE credential used to register UE on 5G core and then APN is enabled.
```sh
[usim]
mode = soft
algo = milenage
opc  = 63BFA50EE6523365FF14C1F45F88737D
k    = 00112233445566778899aabbccddeeff
imsi = 901700123456780
imei = 353490069873319
```
![image](https://user-images.githubusercontent.com/74201353/221578479-91113572-7aff-49a2-9a68-d2d29e8985bf.png)

### 2. Creating Network Namespace
It is important to create to appropriate network namespace for the UE when using ZMQ.
```sh
sudo ip netns add ue1
```
To verify
```sh
sudo ip netns list
```
![image](https://user-images.githubusercontent.com/74201353/221643927-d7c8f20b-742d-477b-a6d1-94450f43f741.png)

### 3. srsENB configuration
The Following steps need to be taken to modify the srsENB config and associated config files to enable 5G SA features:

#### 1. Configuring enb.conf
1. Set correct PLMN and change MME Address to match Open5GS GTPU and NGAP address
```sh
[enb]
enb_id = 0x19B
mcc = 901
mnc = 70
mme_addr = 127.0.0.2
gtp_bind_addr = 127.0.1.1
s1c_bind_addr = 127.0.1.1
s1c_bind_port = 0
n_prb = 50
```
2. Enable ZMQ
```sh
[rf]
rx_gain = 40
tx_gain = 80

# Example for ZMQ-based operation with TCP transport for I/Q samples
device_name = zmq
device_args = fail_on_disconnect=true,tx_port=tcp://*:2000,rx_port=tcp://localhost:2001,id=enb,base_srate=11.52e6
```
![image](https://user-images.githubusercontent.com/74201353/221645535-9c05398d-1b44-44a9-8320-2b83c51aa8e4.png)


#### 2. Configuring rr.conf
Remove LTE cells by commenting out and add 5G cell to cell list
```sh
nr_cell_list =
(
  {
    rf_port = 0;
    cell_id = 1;
    root_seq_idx = 1;
    tac = 7;
    pci = 500;
    dl_arfcn = 368500;
    coreset0_idx = 6;
    band = 3;
  }
);
```
![image](https://user-images.githubusercontent.com/74201353/221647558-c39f1608-b170-4cf1-8f51-f3fcf31d5ec6.png)

## 5. Setup the Network
### 1. Running the core
Open5gs has alredy been configured. Before starting UE and gNB, 5G core must be run in the background.To run the core, net.sh is configured as following
![image](https://user-images.githubusercontent.com/74201353/221648626-b8389492-fbdf-4bd3-8c26-13fa857da6c7.png)
Then run the core in background
![image](https://user-images.githubusercontent.com/74201353/221649039-759e196e-ba42-4810-ae66-768b87fa53f5.png)
![image](https://user-images.githubusercontent.com/74201353/221649160-c1c8da32-3193-41d1-82d3-67ab2fde2bdb.png)

### 2. Running srsENB
If srsENB run successfully, the following output can be found. The NG connection successful message confirms that srsENB has connected to the core.
![image](https://user-images.githubusercontent.com/74201353/221649670-a49b63a7-2d15-4ae2-8adf-5fa379672a3a.png)

### 3. Running srsUE
It is clear that the connection has been made successfully once the UE has been assigned an IP, this is seen in PDU Session Establishment successful. IP: 10.45.0.2. The NR connection is then confirmed with the RRC NR reconfiguration successful. message.
![image](https://user-images.githubusercontent.com/74201353/221650143-b8a25c94-9ac1-4c76-bb17-b1c37b83c15d.png)

## 6. Testing the Network
This is the simplest way to test the network. This will test whether or not the UE and core can successfully communicate.
To test the connection in the uplink direction 
```sh
sudo ip netns exec ue1 ping 10.45.0.1
```
To run traffic in the downlink direction
```sh
ping 10.45.0.2
```
The IP for the UE can be taken from the UE console output. This will change each time a UE reconnects to the network, so it is best practice to always double check the latest IP assigned by reading it from the console before running the downlink traffic.
![image](https://user-images.githubusercontent.com/74201353/221651631-b65f6a4b-57eb-40e8-a5a2-944975e329a0.png)
![image](https://user-images.githubusercontent.com/74201353/221651592-fd18f689-a469-40b9-802a-caeed40c1dd7.png)
![image](https://user-images.githubusercontent.com/74201353/221651728-e21bfe90-8aab-4538-b8d0-91bab6122e66.png)
![image](https://user-images.githubusercontent.com/74201353/221651786-85b58e69-ab10-4532-a36b-99896749835c.png)
Ping to google.com to verify the connectivity to the internet.
![image](https://user-images.githubusercontent.com/74201353/221652032-f97dfc72-c6d8-4b1e-a9d0-1ec4393c775d.png)

## 7. Capturing the Traffic
To capture network traffic, Wireshark is installed
### 1.Wireshark Installation
Open Terminal Add the Wireshark stable official PPA:

``sh
sudo add-apt-repository ppa:wireshark-dev/stable
``

Update the repository:

``sh
sudo apt update
``

Install Wireshark

``sh
sudo apt install wireshark
``

After Installation, Execute the command below so that non-root users can also capture the live packets.

``sh
 sudo chmod +x /usr/bin/dumpcap
``

### 2.Captured frame of the Uplink Traffic
![image](https://user-images.githubusercontent.com/74201353/221652931-cd0291a0-f462-4413-85e6-b48369d62527.png)

### 3.Captured frame of the Downlink Traffic
![image](https://user-images.githubusercontent.com/74201353/221653108-74715f2e-1b1b-45b8-9872-ecd74cc40b35.png)

## References
https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
https://open5gs.org/open5gs/docs/guide/01-quickstart/
https://linuxize.com/post/how-to-install-mongodb-on-ubuntu-18-04/
https://docs.srsran.com/projects/4g/en/latest/general/source/1_installation.html
https://docs.srsran.com/projects/4g/en/latest/app_notes/source/5g_sa_E2E/source/index.html
https://docs.srsran.com/projects/4g/en/latest/app_notes/source/zeromq/source/index.html
































