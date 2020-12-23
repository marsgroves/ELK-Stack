# ELK-Stack Demo
This <b>ELK Stack</b> demo was created and deployed in an <a href="https://portal.azure.com">Azure</a> Cloud environment.  

The purpose of this demo is to showcase SIEM servers, monitoring log events, and alerting. It is also to allow you to emulate the step-by-step guideline below, so you can build and deploy an ELK Stack in an Azure Cloud Environment.

This <b>ELK Stack</b> monitored for a DVWA app. Filebeat monitored and collected log events of ongoing processes in realtime. Metricbeat collected metrics such as CPU and memory. The <a href="https://www.elastic.co/guide/en/siem/guide/current/index.html">SIEM</a> alerted for potential security threats e.g., authentication failures due to unauthorized access.

🙌 Hooray! Let's get started! 🙌
# Building the ELK-Stack #
Open the Azure Cloud environment to get started ☕ 

ツ In the following order, we will begin by building our basic foundation in Azure starting with creating the following: Resource Group, Virtual Network, Network Security Group & Inbound Security Rules, SSH-Key. 

Afterward, we will build our Virtual Machines including a <a href="https://www.docker.com/">Docker</a> container in which <a href="https://www.ansible.com/hubfs/pdfs/Ansible-InDepth-WhitePaper.pdf">Ansible</a> will be deployed inside of. 

Lastly, we will create our Ansible and <a href="https://www.gnu.org/software/bash/">Bash</a> scripts that we will run in order to access the ELK server landing page (<a href="https://www.elastic.co/what-is/kibana">Kibana</a>). This is where the magic happens when we can utilize Filebeat and Metricbeat for monitoring logs, as well as the utilization of SIEM for detecting security threats alerting for anomalous activities.

# Azure Cloud Environment Setup 
## Resource Group Creation ##

1. Create the Resource Group and name it 'Red-Team'
2. Select Region: US West 2
## Virtual Network (VNet) Creation ##

1. Create the VNet and name it 'RedTeamNet'
2. Select the Resource Group 'Red-Team' just created
3. Create the VNet with network IP range 10.1.0.0/16
4. Create default subnet with IP range 10.1.0.0/24

## Establish Network Security Group ##

1. Create a Network Security Group 'Red-Team-NSG'
2. Select Region: US West 2
3. Create the following Inbound Security Rule to block ALL traffic below. ⚠️⚠️⚠️ IMPORTANT! ⚠️⚠️⚠️ <b><u>DO NOT SKIP THIS STEP</b></u> in order to protect the environment as you are building it!!</b> The NSG is subsequently modified to allow specific traffic as the infrastructure is built. 
```

    1. Source: Any
    2. Source Port: */Any
    3. Destination: Any
    4. Destination Port: */0-65535
    5. Protocol: Any
    6. Action: Block
    7. Priority: 4096
    8. Name: BLOCK_ALL
    9. Description: Block all inbound traffic
```
![RedTeam](images/Red-Team-NSG_BlockAll.png)
## 🔑🔑  SSH Key Creation  ##

1. Create an SSH key on your local machine to allow SSH access to the virtual machines which we will be creating soon. 

Open up your terminal and run the following command:

```bash
$ ssh-keygen -f ~/.ssh/elkstack_key
```
☝ This command will automatically generate both a <b>private</b> <i>and</i> <b>public</b> SSH key on your machine in your .ssh folder in a file named elkstack_demo <i>(or any file name of your choice)</i>.

    *NOTE: Make sure to have an empty passphrase. DO NOT type a passphrase! Doing so may create conflicts when running Ansible Playbook scripts later, and you will have to regenerate your key again.

👉 You will notice there are two SSH key files created: 

~/.ssh/elkstack_key
<b>and</b>
~/.ssh/elkstack_key.pub

⚠️ Be mindful that the key file ending in .pub is your <b>PUBLIC</b> key. The other file contains your private key. ⚠️⚠️⚠️ <u><b>DO NOT PASTE YOUR PRIVATE KEY ANYWHERE IN PUBLIC!</b></u> ⚠️⚠️⚠️

2. Run the following command to ensure your public key was generated:

    ```bash
    $ cat ~/.ssh/elkstack_demo.pub
    ```

3. Copy your public key and paste it somewhere safe because we will be using it later.

# Virtual Machines #
We will create a total of four virtual machines in our Azure cloud environment.

The <i>first</i> VM we will create is the <b>Jump Box Provisioner</b>. It is the first VM we will SSH into from our local machine. The Jump Box is where we will deploy the Docker container and run Ansible playbooks inside of it — allowing us to gain access to Kibana, and to utilize Filebeat and Metricbeat for monitoring.

The <i>second</i> and <i>third</i> VMs we will create will be <b>Web-1</b> and <b>Web-2</b>. They will run the Docker container, and they may also be configured to allow access to the Damn Vulnerable Web App (DVWA) app.

Last but not least, the <i>fourth</i> VM will be the <b>ELK-Server</b> which will not be created until <u>AFTER</u> we have set up the ELK-Server environment to include: a new VNet, a new Network Security Group and Inbound Rules, Peerings, etc. The ELK-Server is where we will run our Ansible Playbook scripts to gain entry to the ELK Server landing page, also known as Kibana, where we can utilize Filebeat and Metricbeat to monitor log events on the VMs <b>Web-1</b> and <b>Web-2</b> which may run the Docker container DVWA.

🤖🤖🤖🤖🤖 Let's begin building our virtual machines! 🤖🤖🤖🤖🤖


## 👾 VM 1 - Jump Box Provisioner 

The first virtual machine we will create is the Jump Box which we will access from our local machine via SSH login on our terminal. We will also use SSH login to access the <i>other</i> three VMs in our Azure cloud environment from our Jump Box as well.  

Follow these steps in Azure to create the first VM.

```bash
1. Create a Virtual Machine 
2. Name: 'Jump-Box-Provisioner'
3. Select Resource Group: 'Red-Team'
4. Region: US West 2
5. Availability Options: No infrastructure redundancy required
6. Ubuntu Server 18.04 LTS
7. VM Options:
	1. Standard B1s
		1. 1 CPU
		2. 1 GB RAM
8. Authentication: SSH Public Key (<b>Use existing public key</b>) 
9. Username: 'azureuser' (or any name of choice)
10. Paste key generated from above
11. Paste the Public key from above.
12. Public Inbound Ports: Allow selected ports
13. Select Inbound Ports: SSH (22)
14. Disks - No changes
15. Choose Subnet: default 10.1.0.0/24
16. Select Virtual Network: 'RedTeamNet'
17. Create New Public IP
    1. Name Public IP 'Jump-Box-Provisioner-ip'
    2. SKU: Basic
    3. Assignment: Static
18. NIC Security Group: Advanced
19. Network Security Group: 'Red-Team-NSG'
20. Accelerated networking: Off
21. Management:
    1. Boot diagnostics: Disable
    2. Leave the rest as default
22. Advanced: All defaults
```
## 👾 VMs 2 & 3 - Web 1, Web 2

1. Create VMS 2 & 3 <b><i>one at a time</i></b> following the same exact order below. The only difference between them will be their VM name e.g., Web-1 and Web-2.

```bash
    1. Select Resource Group: 'Red-Team'
    2. Name: Web-1 (VM 2) and Web-2 (VM 3)
    3. Select Region: US West 2
    4. Availability Options: Availability Set
    5. Availability Set:
        1. Create one (if it does not exist)
        2. Name: 'Red-Team-as'
    6. Image: Ubuntu Server 18.04 LTS
	7. Size: B1ms
		1. 1 CPU
		2. 2 GB RAM
	8. SSH Public Key
		1. Username: sysadmin
		2. SSH Public Key: Use existing public key from above
		3. Public Inbound Ports: Allow selected ports
		4. Select Inbound Ports: SSH (22)
	9. Disks - No changes
	10. Networking
	1. Virtual Network: 'RedTeamNet'
	2. Subnet: default
	3. Public IP: None
	4. NIC Network Security Group: Advanced
	5. Configure Network Security Group: 'Read-Team-NSG'
	6. Accelerated Networking: Off
	7. Load Balancing: No
	8. Place this virtual machine behind an existing load balancing solution?: No
	9. Management
		1. Boot diagnostics: Disable
		2. Leave the rest alone
	10. Advanced: All Defaults
    11. Repeat steps to create Web-2 (VM 3)
```
# Establish Network Security Group Rules for Jump Box Provisioner (VM 1)

The purpose of establish NSG rules for the Jump-Box-Provisioner (VM 1) is to grant us SSH login access into VM 1 via the terminal on our local machine.


1. Determine your IPv4 address on Google by searching 'What's my ip address?' <b>OR</b> run the following command in your terminal:

```bash
$ curl icanhazip.com
```

2. Go back to your Network Security Group 'Red-Team-NSG' to update its inbound rules
```bash
        1. Add Inbound Rule
        2. Source IP Address: <your-home-ipv4/32>
        3. Source Port: */Any
		3. Destination: VirtualNetwork
		4. Destination Port: 22
		5. Destination Protocol: TCP
		6. Action: Allow
		7. Priority: 4090
		8. Name: SSH
		9. Description: Allow my home ip to Jump Box
```
![RedTeam](images/Red-Team-NSG_SSHfromHome.png)

3. Copy the <b>public</b> IP address of your Jump Box VM and paste it in your secured notes (you may want to do the same for its private IP address). 

4. SSH login into your Jump Box VM from the terminal on your local machine. Run the following command:

```bash
ssh -i ~/.ssh/elkstack_key azureuser@<Jump-Box-public-ip>
```

You may also try this command as well:
```bash
ssh -v -v -v -i ~/.ssh/elkstack_key azureuser@<Jump-Box-public-ip>
```

🙌 After successfully creating VM 1, VM 2 & 3, we will be creating VM 4 later which will be named ELK-Server.
## Update Network Security Group (NSG) Rules

We will update the NSG rules for <b>Red-Team-NSG</b> to allow SSH from the Jump Box Provisioner.

1. Go back to your NSG rules for 'Red-Team-NSG' 
2. Add Inbound Security Rule:

```bash
    1. Add Inbound Rule
    2. Source IP: <Jump-Box-ip/32)
	3. Source Port: * / Any
	4. Destination: VirtualNetwork
	5. Destination Port: 22
	6. Protocol: TCP
	7. Action: Allow
	8. Priority: 4080
	9. Name: SSH_From_Jumpbox
	10. Description: Allow SSH from the jump box IP to Web Servers
```
![RedTeam](images/Red-Team-NSG_AllowJumpBoxSSHtoWebServers.png)

3. Add Inbound Security Rule:

```bash
    1. Add Inbound Rule
    2. Source IP: <Jump Box-private IP/32)
	3. Source Port: * / Any
	4. Destination: VirtualNetwork
	5. Destination Port: 80
	6. Protocol: TCP
	7. Action: Allow
	8. Priority: 4070
	9. Name: SSH_From_Jumpbox
	10. Description: Allow Port 80 From Jump Box to Web
```
![RedTeam](images/Red-Team-NSG_AllowPort80-FromJumpBoxtoWeb.png)


# Containers

We will be installing <b>Docker</b> in the <b>Jump Box Provisioner (VM 1)</b> and will use Docker to pull a container image and run the container. By doing so, we can run <b>Ansible</b> inside of the container and can run Ansible Playbook scripts such as YML, which will be <i>necessary</i> for running <b>Filebeat</b> and <b>Metricbeat</b> in our <b>ELK Stack</b>. 

## 👉 SSH to the Jump-Box-Provisioner (VM 1)

1. Make sure you are in <b>Jump-Box-Provisioner (VM 1)</b> 

2. Run these commands sequentially in the command line:

```bash
$ sudo apt update
$ sudo apt install -y docker.io
$ sudo systemctl status docker
$ sudo systemctl start docker
$ sudo docker pull cyberxsecurity/ansible
$ sudo docker run -it --name ansible cyberxsecurity/ansible:latest bash 
```

Once the Ansible container is running inside of Docker, you should be in the root of the container which may look similar to this:

```
root@b3847j7a3de:~#
```
After testing that the container runs, you can exit the container like this:

```
root@b3847j7a3de:~# exit
```

<b>More about these commands</b>:

💡 <b>sudo apt update</b> 👇
> This will update advanced package tool (apt) which is used for installing and removing software on Debian, Ubuntu, and Linux distros.

💡 <b>sudo apt install -y docker.io</b> 👇
> This installs Docker
and -y means to assume the answer "yes" to all prompts during installation for convenience purposes

💡 <b>sudo systemctl status docker</b> 👇 
> This shows the status if docker is running (active) or if it dead (inactive)

💡 <b>sudo systemctl start docker</b> 👇
> This command will start docker if it dead (inactive) so that it is running (active) 

💡 <b>sudo docker pull cyberxsecurity/ansible</b> 👇 
> This will pull the container image that we have selected. You can search for a list of many more containers to pull on your own time if you have an account on <a href="https://hub.docker.com">Docker Hub</a>.

💡 <b>sudo docker run -it --name ansible cyberxsecurity/ansible:latest bash</b> 👇
> This command runs the actual container. The <b>--it</b> commands allocates a teletype (<b>tty</b>), a pseudo terminal for the container process. The command <b><i>--name</b></i> signifies the name of the container.
####

#####
```

3. Now that your container has been created and your NSG rules, run these commands in your Jump Box Provisioner (VM 1):

```bash
$ sudo docker ps -a
$ sudo docker start <image_name>
$ sudo docker attach <image_name>
```
💡 <b>sudo docker ps -a </b>
> This command will show you a list of all the containers on your VM. Each container will normally have a unique image name similar to these examples: uniquely_sunshine, lamborghini_essence, cigar_lambert etc. Sometimes, it may as simple as one word such as ansible.</b>

💡 <b>sudo docker start <image_name> </b>
> This command is like pull the container image

💡 <b>sudo docker attach <image_name> </b>
> This command actually runs the container. Once it is successfully attached, you will be in the root of the container.

## 🔑🔑 Create New SSH Key Inside Container

After running our Ansible container from the Docker, we should be in the root where we will create a new SSH-key.

1. Make sure you are in the root of the Ansible container e.g., root@38947a38be:~#

2. Run the following command while in the root of our container to create a new SSH-key in ~/.ssh/id_rsa

```
# ssh-keygen
```
3. <b><i>IMPORTANT:</b></i> Make sure to have an empty passphrase! <u><b>DO NOT</b></u> write a passphrase to prevent any conflicts when Ansible runs Playbook YML scripts inside of the container later.

4. Run the following command to view the contents of the SSH public key just created:

```
# cat ~/.ssh/id_rsa.pub
```

5. Copy and paste the <b>public</b> SSH key somewhere in your secure notes since we will be using this same key for the ELK Server (VM 4) that we will be creating next.

6. You can use this same <b>public</b> SSH key for all your VMs (Jump Box, Web 1 & 2, ELK Server), and even change it on your local machine for ease of access. <b>Configure the private and public key file on each VM and change your public key under 'Reset Password' in Azure when you open up each VM one at a time -- this will allow you to use one key for easy access</b>

# Create Load Balancer, Health Probe, & Backend Pools

We will create a load balancer, health probe, and backend pools for the <b>Web Servers (Web 1 & Web 2 VMs).</b> This is to prevent a Denial of Service (DoS) due to the Web Servers receiving too much traffic from the Red Team. We will install a load balancer in front of the Web Servers to distribute the traffic across more than one VM.

## 👉 Create Load Balancer

1. Create a load balancer in your Azure environment and and name it 'Red-Team-LB' and do the following:
```
  1. Resource group: Red-Team
  2. Name: Red-Team-LB
  3. Public IP address name: Red-Team-LB
  4. Type: Public
  5. SKU: Basic
  6. Public IP name: Red-Team-LB (also)
  7. Public IP assignment: Static
  8. Add a public IPv6 address: No
```
## 👉 Add Health Probe to Load Balancer 

1. Add a health probe to the load balancer just created
```
  1. Name: RedTeamProbe
  2. Protocol: TCP
  3. Port: 80
  4. Interval: 5
  5. Threshold: 2
```
## 👉 Add Backend Pools to Load Balancer

1. Add a backend pool to the load balancer. The backend pool defines the group of resources that will serve traffic for a given load-balancing rule.

```
    1. Name: Red-Team-Pool
    2. Virtual network: RedTeamNet
    3. IP version: IPv4
    4. Associated to: Virtual Machine
    5. Add the following Virtual Machines (private IP only):
        1. Web-1
        2. Web-2
```

## 👉 Add a Load Balancing Rule 

``` 
    1. Name: Red-Team-LB
    2. IP version: IPv4
    3. Frontend IP address: <load-balancer-public-ip> (LoadBalancerFrontEnd)
    4. Protocol: TCP
    5. Port: 80
    6. Backend port: 80
    7. Backend pool: Red-Team-Port (3 virtual machines)
    8. Health probe: Red-Team-Probe (TCP:80)
    9. Session persistence: Client IP and Protocol
    10. Floating IP: Disabled
```

![LBrule](images/LoadBalancer_Rule.png)
# Setup Environment for ELK Server

We will setup an environment within the Azure cloud for the <b>ELK Server (VM 4)</b> that we will create which will include: Virtual Network, Network Security Group and Inbound Rules, Peerings, etc. The Resource Group 'RedTeam' will remain the same however!

We will access the <b>ELK Server</b> so that we can run 
the Docker container DVWA, and then we can run Ansible inside of it such as Ansible Playbook YML scripts. A few of the Playbook YML scripts that we will run will allow us to utilize Filebeat & Metricbeat in the ELK Server landing page also known as Kibana. Filebeat & Metricbeat will be monitoring VMs 2 & 3 (Web 1 and Web 2) which will also be running DVWA.

This environment must be built and is critical for us to fully build our ELK Stack and utilize its capabilities.

## Create a New Virtual Network (VNet)

Create a new VNet that we will name <b>ELK-NET</b>. This VNet will be used by ELK Server (VM 4) which we will build shortly soonafter.

Follow these steps in Azure to create a new VNet:

```bash
1. Select Resource Group: 'RedTeam'
2. Create Name: 'ELK-NET'
3. Select Region: US East 2
4. IP Address: 10.2.0.0/16 (do not add IPv6 address space)
5. Subnet: default 10.2.0.0/24
6. Leave the rest alone as default
```

Now that you have created the VNet ELK-NET, we will move on to add <b>Peerings</b> for ELK-NET.

## Create a New Peering

We will create a <b>Peer</b> connection between our two VNets 'RedTeamNet' and 'ELK-NET' in order to allow traffic to pass between our VNets and regions (US West 2 and US East 2). This peer connection will make both a connection from the first VNet (RedTeamNet) to the second VNet (ELK-NET). It will also make a reverse connection from the second VNet (ELK-NET) back to the first VNet (RedTeamNet). This will allow traffic to pass in both directions.

Click the <b>+Add</b> button to create a new Peering and do the following:

```bash
1. Name of the peering from ELK-NET to RedTeamNet: 'ELK-to-RED'
2. Peer details / Virtual network deployment model: Resource manager
3. Virtual network: 'RedTeamNet (RedTeam)'
4. Name of the peering from RedTeamNet to ELK-NET: 'RED-TO-ELK'
5. Configuration: Leave everything as default
```
## 👾 VM 4 - ELK Server

We will finally create our <b><i>fourth</b></i> VM that we will name as our <b>ELK Server</b>. From the ELK Server, we can access Kibana and utilize Filebeat and Metricbeat for monitoring including the SIEM servers for detecting threats and alerting of anomalous activities.

Before we begin creating our ELK Server VM, make sure to have your public key ready that we created in the Ansible container (<b>see 🔑🔑 Create New SSH Key Inside Container above</b>).

```bash
    1. Select Resource Group: 'Red-Team'
    2. Name: 'ELK-Server'
    3. Select Region: US East 2
    4. Availability Options: No infrastructure redundancy required
    5. Image: Ubuntu Server 18.04 LTS
	6. Size: B2s
		1. 2 CPU
		2. 4 GB RAM
	8. SSH Public Key
		1. Username: sysadmin
		2. SSH Public Key: Use existing key that you just created from your Ansible container (see above)
		3. Public Inbound Ports: Allow selected ports
		4. Select Inbound Ports: SSH (22)
	9. Disks - No changes
	10. Networking
	11. Virtual Network: 'ELK-Net'
	12. Subnet: 10.2.0.0/24
	13. Public IP: ELK-Server-ip
	14. NIC Network Security Group: Basic
    15. Public inbound ports: Allow selected ports
    16. Select inbound ports: SSH (22)
	17. Accelerated Networking: Off
	18. Load Balancing: No
	19. Place this virtual machine behind an existing load balancing solution?: No
	20. Management
		1. Boot diagnostics: Disable
		2. Leave the rest alone
	21. Advanced: All Defaults
```
# Configure Container

Make sure you are in your <b>Jump Box (VM 1)</b> and run the Ansible container. Once you are in the root, you will be configuring the following Ansible files: <b><i>hosts</b></i>, <b></i>ansible.cfg</b></i>, and the <b><i> playbook</b></i> that will create and run an ELK container, allowing you to load the ELK Stack server and log into Kibana. 

1. First, you will configure the Ansible hosts file and paste the Ansible hosts script (see <b>ansible hosts script in Scripts folder)</b> by running the command:

```
# nano /etc/ansible/hosts
```
2. Second, we will need to configure the <b>ansible.cfg</b> file by running the command:
```
# nano /etc/ansible/ansible.cfg
```
Paste the ansible.cfg script (<b>see Scripts folder</b>)

3. Third, you will need to configure the <b>Ansible playbook YML</b> to run the ELK container. We will run this command:

```
# nano /etc/ansible/install-elk.yml
```

Paste the <b>ansible playbook</b> install-elk.yml (see <b>Scripts</b>)

4. Next, after configuring our Ansible scripts, we will run the Ansible playbook YML we just created inside the Ansible container:

```
root@48347b34e:/etc/ansible# ansible-playbook elk-install.yml
```

5. After it is successfully installed, we will configure the hosts file to add the Web Servers and ELK Servers, so that we can SSH into each VM from the Ansible container without requiring an IP address but just name:

```
# nano /etc/hosts
```
The /etc/hosts file should look similar to below:

```
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      25bc6f626e7f

10.1.0.5        Web-1 web-1
10.1.0.6        Web-2 web-2
10.2.0.4        ELK-SERVER elk-server
```

💡 <b>Note</b>: You WILL need to configure the <b>hosts</b> file each time you restart your Jump Box (VM 1) after stopping your Jump Box. Any changes made to the /etc/hosts file will not remain permanently, unfortunately. 

Now you should be able to SSH into Web 1, Web 2, or Web 3 from the <b>Jump Box (VM 1)</b> without requiring their IP address. 👍👍

For example:

```
# ssh sysadmin@web-1
# ssh sysadmin@web-2
# ssh sysadmin@elk-server
```

<i>Rather than</i>

```
# ssh sysadmin@<web1-privateip>
# ssh sysadmin@<web2-privateip>
# ssh sysadmin@elkserver-privateip>
```

7. Run the following command to test that each VM will ping and pong successfully!

```
# ansible all -m ping
```
If successful, it will look like this 👇

```  
  10.0.0.6 | SUCCESS => {
     "changed": false, 
     "ping": "pong"
  }
  10.0.0.7 | SUCCESS => {
     "changed": false, 
      "ping": "pong"
  }
  10.0.0.5 | SUCCESS => {
      "changed": false, 
      "ping": "pong"
  }
```

# Load the ELK Stack Server (Kibana)

We will SSH into the ELK Server (VM 4) and test that the ELK container we just created from the Ansible playbook script is running. Then we will create an incoming rule for the ELK Server's security group that will allow ELK Server (VM 4) to load the ELK Stack server from your browser at the Kibana page.

## SSH into ELK Server

1. SSH into ELK Server (VM 4) from the Ansible container.


```
# ssh sysadmin@elk-server
```

2. We will SSH into the ELK container just created from our Ansible playbook and double check that it is running.

```
sysadmin@elk-server:~$ sudo docker ps -a
```

Now that we can see it is successfully running, we will create an Inbound Security Rule for the ELK Server's security group.

## Create Inbound Security Rules in ELK Server's Security Group

Since the <b>ELK Server</b> (VM 4) runs on port 5601, we will create an incoming rule in ELK Server's security group so that we can allow TCP traffic over port 5601 from your IP address. We will also create an Inbound Security Rule that will also allow us to access the ELK Stack server that runs on port 5601, so that we can load Kibana from our web browser at home.

<b>Note</b>: It is fine to send traffic to the VNet ELK-NET because there are no other resources besides the ELK Server.

1. To create an Inbound Security Rule that will allow TCP traffic over port 5601 from our IP address, we will create the following rule:

```
1. Add Inbound Rule
2. Source: IP Addresses
3. Source IP Addresses: <ELK-Server-public-ip>
4. Source Port Ranges: */Any
5. Destination: VirtualNetwork
6. Destination Port Ranges: 80
7. Protocol: Any
8. Action: Allow
9. Priority: 300
10. Name: Port_80
```

2. We will add another Inbound Security Rule that will allow us to log into Kibana from our web browser at home:

```
1. Add Inbound Rule
2. Source: IP Addresses
3. Source IP Addresses: <your-home-ip/32>
4. Source Port Ranges: */Any
5. Destination: VirtualNetwork
6. Destination Port Ranges: 5601
7. Protocol: TCP
8. Action: Allow
9. Priority: 310
10. Name: to port 5601 from home
```

Your setup will now appear like this 👇

![CloudDiagram](images/Cloud-Diagram.png)

## Load Kibana from Web Browser

1. Make sure you SSH into the ELK Server from the Ansible container if you are not already in there.

```
# ssh sysadmin@elk-server
```

2. Once in the ELK Server, we will run the curl command that will allow us access to the Kibana page:

```
$ curl elk-server:5601/app/kibana
```

⚠️⚠️ !!! <b>IMPORTANT</b> !!! ⚠️⚠️ You <b><u>MUST</u></b> run this curl command before you access the <b>ELK Server landing page (Kibana)</b> from your web browser, and before you run the <b>Filebeat</b> & <b>Metricbeat</b> YML scripts. By skipping this step, you will not be able to access the Kibana page.

<b>ALSO</b>, make sure you ran the following Ansible playbook code from earlier in the correct dir. 

```
# ansible-playbook elk-install.yml 
``` 

Failing to run the <b>elk-install.yml</b> playbook earlier is one reason the curl command will certainly <u>NOT</u> work!


2. Go to the following url:
```
http://<ELK-Server-public-ip>:5601/app/kibana
```

Once it is successful, you should see the <b>Kibana</b> page!

![Kibana](images/Kibana_Success.png)

♣️♣️ SUCCESS!! ♣️♣️

3. Now that the Kibana front page has loaded, open a terminal and SSH into your Jump-Box then and run the Ansible container with the following commands:

```
$ sudo docker ps -a
$ sudo docker start <container_name>
$ sudo docker attach <container_name>
```
# Filebeat

We will create a configuration YML file and a playbook for Filebeat in the <b>/etc/ansible/files</b> directory. These Ansible scripts will be ran while we are in our Ansible container.


1. Make sure you are inside your Ansible container and create a 'files' directory with this command:

```
# mkdir /etc/ansible/files
```

2. Next, change directories into the 'files' dir just created:

```
root@b83437a848e:~# cd /etc/ansible/files
```

3. Run the curl command:

```
# curl https://gist.githubusercontent.com/slape/5cc350109583af6cbe577bbcc0710c93/raw/eca603b72586fbe148c11f9c87bf96a63cb25760/Filebeat > /etc/ansible/files/filebeat-config.yml
```

4. Configure the <b>filebeat-config.yml</b>:

```
# nano /etc/ansible/files/filebeat-config.yml
```

💡 IMPORTANT: <b>While still in nano in the filebeat-config.yml, press CTRL+W to search 10.0.0.12 twice and replace it with the ELK-Server private IP e.g., 10.2.0.4 (yours may be different) then SAVE filebeat-config file. You cannot run Filebeat without properly configuring the IP. </b>

5. Create a Filebeat playbook script in the files dir:

```
# nano /etc/ansible/files/filebeat-playbook.yml
```

6. Run the Filebeat playbook:

```
# ansible-playbook filebeat-playbook.yml
```
7. Go back to the Kibana front page on your web browser 
    1. Click on "Add log data" under Logs
    2. Select "System logs"
    3. Click on "DEB"
    4. Scroll down to the bottom of the Filebeat install instructions until you can see "Module Status"
    5. Click on "Check Data" under Module Status
    6. After clicking Check Data, you should read the message: "Data successfully received from this module!"
    7. Now you can click on "System logs dashboard" to explore Filebeat!


🎊🎉 Congrats! You've successfully installed Filebeat! 🎉🎊

![FilebeatSuccess](images/Filebeat_Success.png)

# Explore Filebeat

Once you are in the System logs dashboard of Filebeat, you can monitor syslogs hostnames and processes in your Web Servers. You can select the timeframe of your choosing to monitor activities in the Web Servers.

![FilebeatLogs](images/Filebeat_Logs.png)

# Metricbeat

We will create a configuration YML file and a playbook for Metricbeat in the <b>/etc/ansible/files</b> directory. These Ansible scripts will be ran while we are in our Ansible container.

1. Make sure you are in the Ansible container in your Jumpbox and cd to /etc/ansible/files 

```
# cd /etc/ansible/files
```

2. Configure the metricbeat-config file the same way you did with the filebeat-config file.

```
# nano /etc/ansible/files/metricbeat-config.yml
```

<b>Press CTRL+W to search 10.0.0.12 twice and replace it with the ELK-Server private IP then save.</b>

3. While you are in the 'files' directory, create the metricbeat-playbook yml:

```
# nano metricbeat-playbook.yml
```

4. After you copied and pasted the metricbeat-playbook.yml script, you will run it.

```
# ansible-playbook metricbeat-playbook.yml
```

5. Go back to the Kibana front page on your web browser 
    1. Click on "Metrics" 
    2. Select "Docker metrics"
    3. Click on "DEB"
    4. Scroll down to the bottom of the Metricbeat installation instructions until you can see "Module Status"
    5. Click on "Check Data" under Module Status
    6. After clicking Check Data, you should read the message: "Data successfully received from this module!"
    7. Now you can click on "Docker metrics dashboard" to explore Metricbeat!

🎊🎉 Congrats! You've successfully installed Metricbeat! 🎉🎊

# Explore Metricbeat

Once you are in the Docker metrics dashboard of Metricbeat, you can monitor metrics such as CPU and memory in the Web Servers from the dashboard as seen below. 

![Metricbeat](images/Metricbeat_Dockerlogs.png)

You can also test the Web Servers' CPU capacity with a stress test.

In order to run a stress test in one of your Web Servers, do the following:

1. SSH into Web 2 VM

```
# ssh sysadmin@Web-2
```

2. Run the stress test command to test it and watch how the CPU % increases within mins:

```
$ sudo stress --cpu 1
```

![MetricbeatStress](images/Metricbeat_StressTestWeb1-start.png)


3. Now SSH into Web 1 VM and run the stress test command to see what happens after waiting.

![MetricbeatFullStress](images/Metricbeat_StressTestWeb1-100.png)

☝️ Web-1 VM is maxed out at full CPU capacity at 100%. Metricbeat monitors these processes.

# SIEM

Now you can also click on SIEM on the menu and explore some of its capabilities. In this case, we will briefly explore how SIEM monitors hosts, authentications, and events and alerts for authentication failures.

![SIEM](images/SIEM_Authentications2.png)

![SIEM](images/SIEM_EventCount.png)

![SIEM](images/SIEM_Events.png)

![SIEM](images/SIEM_HostsAuthentications.png)


🙌 Hooray! You have created your ELK Stack! Happy exploring! 🙌





























