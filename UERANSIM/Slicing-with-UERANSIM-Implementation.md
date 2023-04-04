# Slicing configuration with Open5GC and UERANSIM - with Single SMF.
This describes the configuration that uses Open5GS and UERANSIM for connecting multiple UE's with seperate slicing associated.

## Contents/Steps

- [Overview of Open5GS 5GC Simulation Mobile Network](#Overview)
- [Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN](#Open5GS)
- [Changes in configuration files of Open5GS 5GC C-Plane and U-Plane](#CPlane)
- [Changes in configuration files of UERANSIM UE / RAN](#UE)
- [Network settings of Open5GS 5GC and UERANSIM UE / RAN](#Network)
- [Build Open5GS and UERANSIM](#Build)
- [Run UERANSIM (gNodeB)](#Run)
- [PING google.com](#PING)
- [References](#References)

<h2 id="Overview">Overview of Open5GS 5GC Simulation Mobile Network</h2>

![image](https://user-images.githubusercontent.com/93492067/221964645-baff1697-3aec-4c14-b63f-f06d8bc1200f.png)

| SST | ST | APN/DNN | Description | IP Address |
| --- | --- | --- | --- | --- |
| 1 | 000001 | Internet | Access to Internet | 10.45.0.1/16 |
| 1 | 000002 | Internet2 | Access to Internet | 10.46.0.1/16 |
| 2 | 000001 | voip | Access to Internet | 10.47.0.1/16 |
| 2 | 000002 | voip2 | Access to Internet | 10.48.0.1/16 |

<h2 id="Open5GS">Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN</h2>

### Please refer to the following for building Open5GS and UERANSIM respectively.
[Open5GS with UERANSIM Implementation](https://github.com/FRA-UAS/mobcom-project-noobies/blob/main/UERANSIM-Implementation-Guide.md)

<h3 id="CPlane">Changes in configuration files of Open5GS 5GC C-Plane and U-Plane</h3>

### AMF Configuration:

Configure as shown in the images:
- `/etc/open5gs/amf.yaml`

![image](https://user-images.githubusercontent.com/93492067/221886467-291f17af-2ae2-4584-99a3-6788e92ed464.PNG)

### SMF Configuration:

- `/etc/open5gs/smf.yaml`
 
![image](https://user-images.githubusercontent.com/93492067/221886536-a4424032-739e-4c72-a9a0-9c2a9c8510bf.PNG)

![image](https://user-images.githubusercontent.com/93492067/221886578-42041d44-1f91-4609-996d-56ee6fb0df39.PNG)

![image](https://user-images.githubusercontent.com/93492067/221886639-00483c74-4bf5-4ad4-9c56-f033410c87b9.PNG)

### NSSF Configuration:

- `/etc/open5gs/nssf.yaml`

![image](https://user-images.githubusercontent.com/93492067/221886679-3fd19b77-a457-4d89-8b95-a058f22e067b.PNG)

### UPF Configuration:

- `/etc/open5gs/upf.yaml`

![image](https://user-images.githubusercontent.com/93492067/221886738-c6245c15-0d9b-4795-a4b5-abcd8553ab27.PNG)
 
<h3 id="UE">Changes in configuration files of UERANSIM UE / RAN</h3>

### Changes in configuration files of RAN (gNodeB):

- `UERANSIM/config/open5gs-gnb.yaml`
 
![image](https://user-images.githubusercontent.com/93492067/221886799-d5b59a84-915a-4f1b-ae8f-d3911b137807.PNG)

### Changes in configuration files of UE set to SST:1 and SD: 000001 (IMSI-99970000000001)

- `UERANSIM/config/open5gs-ue1.yaml`

![image](https://user-images.githubusercontent.com/93492067/221872032-83fdb0a4-3f28-40f7-87ab-83719cf98734.PNG)

### Changes in configuration files of UE set to SST:1 and SD: 000002 (IMSI-99970000000002)

- `UERANSIM/config/open5gs-ue2.yaml`

![image](https://user-images.githubusercontent.com/93492067/221872443-3ffdf61d-e992-4b80-885e-d5d7130bee2c.PNG)

### Changes in configuration files of UE set to SST:2 and SD: 000001 (IMSI-99970000000003)

- `UERANSIM/config/open5gs-ue3.yaml`
 
![image](https://user-images.githubusercontent.com/93492067/221887509-53981ed8-5f00-420a-8500-7d8418e560b5.PNG)

### Changes in configuration files of UE set to SST:2 and SD: 000002 (IMSI-99970000000004)

- `UERANSIM/config/open5gs-ue4.yaml`

![image](https://user-images.githubusercontent.com/93492067/221887595-5c99fe1a-1715-4bce-9cec-eeb8c6443f38.PNG)

<h3 id="Network">Network settings of Open5GS 5GC and UERANSIM UE / RAN</h3>

```
ip tuntap add name ogstun mode tun
ip addr add 10.45.0.1/16 dev ogstun
ip addr add 10.46.0.1/16 dev ogstun
ip addr add 10.47.0.1/16 dev ogstun
ip addr add 10.48.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
iptables -t nat -A POSTROUTING -s 10.46.0.0/16 ! -o ogstun -j MASQUERADE
iptables -t nat -A POSTROUTING -s 10.47.0.0/16 ! -o ogstun -j MASQUERADE
iptables -t nat -A POSTROUTING -s 10.48.0.0/16 ! -o ogstun -j MASQUERADE
```

<h3 id="Build">Build Open5GS and UERANSIM</h3>

Please Refer to the below guide:
[Open5GS with UERANSIM Implementation](https://github.com/FRA-UAS/mobcom-project-noobies/blob/5dc86f42ad01a25c5c76e9e7ea979e87e99f41c0/UERANSIM-Implementation-Guide.md)

<h3 id="Run">Run UERANSIM (gNodeB)</h3>

```
^Croot@noobies:~/UERANSIM# ./build/nr-gnb -c config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2023-02-28 14:51:57.292] [sctp] [info] Trying to establish SCTP connection... (192.168.10.17:38412)
[2023-02-28 14:51:57.300] [sctp] [info] SCTP connection established (192.168.10.17:38412)
[2023-02-28 14:51:57.302] [sctp] [debug] SCTP association setup ascId[14]
[2023-02-28 14:51:57.303] [ngap] [debug] Sending NG Setup Request
[2023-02-28 14:51:57.307] [ngap] [debug] NG Setup Response received
[2023-02-28 14:51:57.307] [ngap] [info] NG Setup procedure is successful
```
- NG Setup in Wireshark

![image](https://user-images.githubusercontent.com/93492067/221899620-263de73e-4c77-422a-8744-edff501939fb.PNG)

![image](https://user-images.githubusercontent.com/93492067/221899644-1b544847-ef28-4fe3-9253-6086bef79a89.PNG)

### Run UERANSIM (UE set to SST:1 and SD:000001)

```
^Croot@noobies:~/UERANSIM# sudo ./build/nr-ue -c config/open5gs-ue1.yaml
UERANSIM v3.2.6
[2023-02-28 15:01:40.753] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-02-28 15:01:40.754] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-02-28 15:01:42.953] [nas] [error] PLMN selection failure, no cells in coverage
[2023-02-28 15:01:43.341] [nas] [info] Selected plmn[999/70]
[2023-02-28 15:01:43.343] [rrc] [info] Selected cell plmn[999/70] tac[1] category[SUITABLE]
[2023-02-28 15:01:43.343] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-02-28 15:01:43.343] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-02-28 15:01:43.344] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-02-28 15:01:43.356] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-02-28 15:01:43.359] [nas] [debug] Sending Initial Registration
[2023-02-28 15:01:43.361] [rrc] [debug] Sending RRC Setup Request
[2023-02-28 15:01:43.366] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-02-28 15:01:43.369] [rrc] [info] RRC connection established
[2023-02-28 15:01:43.369] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-02-28 15:01:43.369] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-02-28 15:01:43.412] [nas] [debug] Authentication Request received
[2023-02-28 15:01:43.426] [nas] [debug] Security Mode Command received
[2023-02-28 15:01:43.427] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-02-28 15:01:43.447] [nas] [debug] Registration accept received
[2023-02-28 15:01:43.447] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-02-28 15:01:43.447] [nas] [debug] Sending Registration Complete
[2023-02-28 15:01:43.447] [nas] [info] Initial Registration is successful
[2023-02-28 15:01:43.447] [nas] [debug] Sending PDU Session Establishment Request
[2023-02-28 15:01:43.447] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-02-28 15:01:43.659] [nas] [debug] Configuration Update Command received
[2023-02-28 15:01:43.692] [nas] [debug] PDU Session Establishment Accept received
[2023-02-28 15:01:43.724] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-02-28 15:01:43.829] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.4] is up.
```
### Run UERANSIM (UE set to SST:1 and SD:000002)

```
^Croot@noobies:~/UERANSIM# sudo ./build/nr-ue -c config/open5gs-ue2.yaml
UERANSIM v3.2.6
[2023-02-28 15:02:11.094] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-02-28 15:02:11.096] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-02-28 15:02:11.098] [nas] [info] Selected plmn[999/70]
[2023-02-28 15:02:11.098] [rrc] [info] Selected cell plmn[999/70] tac[1] category[SUITABLE]
[2023-02-28 15:02:11.098] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-02-28 15:02:11.098] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-02-28 15:02:11.099] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-02-28 15:02:11.112] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-02-28 15:02:11.115] [nas] [debug] Sending Initial Registration
[2023-02-28 15:02:11.116] [rrc] [debug] Sending RRC Setup Request
[2023-02-28 15:02:11.119] [rrc] [info] RRC connection established
[2023-02-28 15:02:11.121] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-02-28 15:02:11.121] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-02-28 15:02:11.122] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-02-28 15:02:11.135] [nas] [debug] Authentication Request received
[2023-02-28 15:02:11.144] [nas] [debug] Security Mode Command received
[2023-02-28 15:02:11.144] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-02-28 15:02:11.172] [nas] [debug] Registration accept received
[2023-02-28 15:02:11.172] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-02-28 15:02:11.172] [nas] [debug] Sending Registration Complete
[2023-02-28 15:02:11.172] [nas] [info] Initial Registration is successful
[2023-02-28 15:02:11.172] [nas] [debug] Sending PDU Session Establishment Request
[2023-02-28 15:02:11.173] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-02-28 15:02:11.389] [nas] [debug] Configuration Update Command received
[2023-02-28 15:02:11.434] [nas] [debug] PDU Session Establishment Accept received
[2023-02-28 15:02:11.435] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-02-28 15:02:11.632] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun1, 10.46.0.3] is up.
```
Do the same for UE 3 and UE 4 as well

### The TUNnel interface uesimtun0,1,2 and 3 is created as follows.

```
10: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none
    inet 10.45.0.4/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::4dc9:9f67:eba0:3451/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
11: uesimtun1: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none
    inet 10.46.0.3/32 scope global uesimtun1
       valid_lft forever preferred_lft forever
    inet6 fe80::6f3:4a1d:bb9e:4f14/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
11: uesimtun2: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none
    inet 10.47.0.2/32 scope global uesimtun1
       valid_lft forever preferred_lft forever
    inet6 fe80::6f3:4a1d:bb9e:4f14/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
11: uesimtun3: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none
    inet 10.48.0.3/32 scope global uesimtun1
       valid_lft forever preferred_lft forever
    inet6 fe80::6f3:4a1d:bb9e:4f14/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
```

<h3 id="PING">Ping google.com going through DN=10.46.0.0/16 , 47, and 48.</h3>

```
root@noobies:~# ping -I uesimtun0 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 10.45.0.4 uesimtun0: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=148 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=14.7 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=20.3 ms
64 bytes from 8.8.8.8: icmp_seq=7 ttl=57 time=14.1 ms
64 bytes from 8.8.8.8: icmp_seq=8 ttl=57 time=16.7 ms
^C
--- 8.8.8.8 ping statistics ---
8 packets transmitted, 8 received, 0% packet loss, time 7011ms
```
```
root@noobies:~# ping -I uesimtun1 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 10.46.0.3 uesimtun1: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=53 ttl=57 time=12.8 ms
64 bytes from 8.8.8.8: icmp_seq=54 ttl=57 time=13.8 ms
64 bytes from 8.8.8.8: icmp_seq=55 ttl=57 time=20.8 ms
64 bytes from 8.8.8.8: icmp_seq=61 ttl=57 time=19.3 ms
64 bytes from 8.8.8.8: icmp_seq=62 ttl=57 time=19.3 ms
64 bytes from 8.8.8.8: icmp_seq=63 ttl=57 time=12.4 ms
```
```
root@noobies:~# ping -I uesimtun2 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 10.47.0.2 uesimtun2: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=148 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=14.7 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=20.3 ms
64 bytes from 8.8.8.8: icmp_seq=7 ttl=57 time=14.1 ms
64 bytes from 8.8.8.8: icmp_seq=8 ttl=57 time=16.7 ms
^C
```
```
root@noobies:~# ping -I uesimtun3 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 10.48.0.3 uesimtun3: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=148 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=14.7 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=20.3 ms
64 bytes from 8.8.8.8: icmp_seq=7 ttl=57 time=14.1 ms
64 bytes from 8.8.8.8: icmp_seq=8 ttl=57 time=16.7 ms
^C
```
![image](https://user-images.githubusercontent.com/93492067/221881473-b293d764-d639-4865-b697-0105e03b090d.PNG)

![image](https://user-images.githubusercontent.com/93492067/221881495-ff468314-b1df-4709-a278-79352825c47b.PNG)

![image](https://user-images.githubusercontent.com/93492067/221899178-4214a02d-f9aa-457d-bda3-58ef373e2cfe.PNG)

<h3 id="References">References:</h3>


- https://www.cyberciti.biz/faq/ubuntu-linux-install-openssh-server/
- https://computingforgeeks.com/how-to-install-latest-mongodb-on-ubuntu/
- https://nickvsnetworking.com/my-first-5g-core-open5gs-and-ueransim/
- https://open5gs.org/open5gs/docs/guide/01-quickstart/
- https://kkohls.org/guides_open5gs.html#machine-1-open5gs
- https://www.virtualbox.org/wiki/Download_Old_Builds_6_1
- https://askubuntu.com/questions/151941/how-can-you-completely-remove-a-package


