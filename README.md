# DELTA: A Penetration Testing Framework for Software-Defined Networks

## What is DELTA?
DELTA is a penetration testing framework that regenerates known attack scenarios for diverse test cases. This framework also provides the capability of discovering unknown security problems in SDN by employing a fuzzing technique.

+ Agent-Manager is the control tower. It takes full control over all the agents deployed to the target SDN network.
+ Application-Agent is a legitimate SDN application that conducts attack procedures and is controller-dependent. The known malicious functions are implemented as application-agent functions.
+ Channel-Agent is deployed between the controller and the OpenFlow-enabled switch. The agent sniffs and modifies the unencrypted control messages. It is controller-independent.
+ Host-Agent behaves as if it was a legitimate host participating in the target SDN network. The agent demonstrates an attack in which a host attempts to compromise the control plane.

![Delta architecture](http://143.248.53.145/research/arch.png)

## Prerequisites
In order to build and run DELTA, the following are required:
+ A host machine based on Ubuntu 14.04 LTS 64 bit (agent manager)
+ Three virtual machines based on Ubuntu 14.04 LTS 64 bit.
 + VM-1: Target controller + Application agent
 + VM-2: Channel agent
 + VM-3: Host agent
+ Target Controller (in VM-1)
 + [Floodlight](http://www.projectfloodlight.org/download/): 0.91, 1.2
 + [ONOS](https://wiki.onosproject.org/display/ONOS/Downloads): 1.1, 1.6
 + [OpenDaylight](https://www.opendaylight.org/downloads): Helium-sr3
+ [Cbench](https://floodlight.atlassian.net/wiki/display/floodlightcontroller/Cbench) (in VM-2)
+ [Mininet 2.1+](http://mininet.org/download/) (in VM-3)
+ Ant build system
+ Maven build system
+ Vagrant system
+ JDK 1.7+

## Installing DELTA
DELTA installation depends on maven and ant build systems. The mvn command is used to install the agent-manager and the sub-agents.

+ STEP 0. Get the source
```
$ git clone https://github.com/OpenNetworkingFoundation/DELTA.git
```

+ STEP 1. Install DELTA dependencies on the host machine.

```
$ cd <DELTA>/tools/dev/delta-setup/
$ ./delta-setup-devenv-ubuntu
```

+ STEP 2. Install 3 virtual machines using the vagrant system.

```
$ cd <DELTA>/tools/dev/vagrant/
$ vagrant up
```

+ STEP 3. Install DELTA using maven build.

```
$ cd <DELTA>
$ source ./tools/dev/delta-setup/bash_profile
$ mvn clean install
```

+ STEP 4. Add NAT to VM3 (mininet)
![NAT](http://143.248.53.145/research/nat1.png)

+ After installing DELTA, the test environment is automatically setup as below:
![Env](http://143.248.53.145/research/env.png)

## Configuring your own experiments
+ Execute sudo without Password
```
$ sudo visudo
In the bottom of the file, type the follow:
username ALL=(ALL) NOPASSWD: ALL
```
+ Configure passwd-less ssh login for the VMs.

```
$ cd ~
$ ssh-keygen -t rsa

Generating public/private rsa key pair.
Enter file in which to save the key (/home/sk/.ssh/id_rsa): ## Press Enter
Enter passphrase (empty for no passphrase): ## Enter Passphrase 
Enter same passphrase again: ## Re-enter Passphrase
Your identification has been saved in /home/sk/.ssh/id_rsa.
Your public key has been saved in /home/sk/.ssh/id_rsa.pub.
The key fingerprint is:
e4:6d:fc:7b:6b:d4:0c:04:72:7e:ae:c4:16:f3:13:d1 sk@sk
The key's randomart image is:
+--[ RSA 2048]----+
|          . o... |
|           +  ..E|
|        .   +.o  |
|       o o . *.. |
|        S + + ++ |
|         . + ...o|
|            o.   |
|             .o  |
|            .o.. |
+-----------------+

$ ssh-copy-id -i ~/.ssh/id_rsa.pub vagrant@10.100.100.11
$ ssh-copy-id -i ~/.ssh/id_rsa.pub vagrant@10.100.100.12
$ ssh-copy-id -i ~/.ssh/id_rsa.pub vagrant@10.100.100.13

Check if you will be able to access the VMs without having to enter the password.
```

+ The Agent-Manager automatically reads your configuration file and sets up the environment based on the configuration file settings. [DELTA_ROOT]/tools/config/manager.cfg contains sample configurations. You can specify your own config file by passing its path:
```
CONTROLLER_SSH=vagrant@10.100.100.11
CHANNEL_SSH=vagrant@10.100.100.12
HOST_SSH=vagrant@10.100.100.13
TARGET_HOST=10.0.0.2
ONOS_ROOT=/home/vagrant/onos-1.6.0
CBENCH_ROOT=/home/vagrant/oflops/cbench/
TARGET_CONTROLLER=Floodlight
TARGET_VERSION=0.91
OF_PORT=6633
OF_VER=1.3
MITM_NIC=eth1
CONTROLLER_IP=10.100.100.11
SWITCH_IP=10.100.100.13,10.100.100.13,10.100.100.13
DUMMY_CONT_IP=10.0.2.2
DUMMY_CONT_PORT=6633
AM_IP=10.0.2.2
AM_PORT=3366
```

+ Configuring Target Controllers to VM-1
+ 1) Floodlight
```
$ cd <DELTA>/tools/dev/floodlight-setup
$ ./floodlight-scp
```
 + 2) ONOS
```
$ cd <DELTA>/tools/dev/onos-setup
$ ./onos-<version>-scp
(in VM-1) $ ./onos-<version>-setup
```
+ 3) OpenDaylight: (only JDK 1.7-supported)
```
$ cd <DELTA>/tools/dev/odl-setup
$ ./odl-helium-sr3-scp
(in VM-1) $ ./odl-helium-sr3-scp
```
+ The AppAgent (in VM-1) needs agent.cfg file in order to connect to agent-manager.
```
MANAGER_IP=10.0.2.2
MANAGER_PORT=3366
```

## Running DELTA
+ STEP 1. Distribute the executable files to VMs

```
$ cd <DELTA>
$ source ./tools/dev/delta-setup/bash_profile
$ ./tools/dev/delta-setup/delta-agents-scp
```


+ STEP 2. Execute Agent-Manager first
```
$ cd <DELTA>/manager
$ java -jar target/delta-manager-1.0-SNAPSHOT-jar-with-dependencies.jar ../tools/config/manager.cfg

 DELTA: A Penetration Testing Framework for Software-Defined Networks

 [pP]	- Show all known attacks
 [cC]	- Show configuration info
 [kK]	- Replaying known attack(s)
 [uU]	- Finding an unknown attack
 [qQ]	- Quit

Command>_
```

+ STEP 3. Connect Web-based UI (port number is 7070)

## Main Contributors
+ Seungsoo Lee (KAIST)
+ Changhoon Yoon (KAIST)
+ Sandra Scott-Hayward (Queen's University Belfast)
+ Seungwon Shin (KAIST)

## Collaborators
+ Phil Porras, Vinod Yegneswaran (SRI International) 
+ Kyuho Hwang, Daewon Jung (National Security Research Institute)
+ [Atto Research](http://www.atto-research.com/index.php/en/home/)
+ ![collabo](http://143.248.53.145/research/collabo2.png)


## Questions?
Send questions or feedback to: lss365@kaist.ac.kr or chyoon87@kaist.ac.kr
