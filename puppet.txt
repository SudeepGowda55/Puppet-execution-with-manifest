Introduction
Puppet is an open-source, automation admin engine used to perform administrative tasks and server management remotely. This tool is available on Linux, Unix, and Windows.
the installation of Puppet on Ubuntu 20.04 on master and client nodes.

Prerequisites
    • Multiple systems running Ubuntu 20.04 (one for the master node and one or more for client nodes)
    • Access to an account with sudo privileges
    • Access to the terminal/command line
Step 1: Update Package List
Before you start the installation process, update the list of available packages:
sudo apt-get update -y

Step 2: Set Up Hostname Resolution
  With Puppet, master and client nodes communicate using hostnames. Before installing Puppet, you need to set up a unique hostname on each node(optional).
1. Open the hosts file on each node by using:
sudo nano /etc/hosts
2. Paste the following lines at the end of each hosts file (both):
[puppet master ip address] puppet

3. Press Ctrl + X to close the file, and then type Y and press Enter to save the changes you made.
Step 3: Install Puppet Server on Master Node
1. Download the latest Puppet version on the master node:
wget https://apt.puppetlabs.com/puppet6-release-focal.deb

2. Once the download is complete, install the package by using:
sudo dpkg -i puppet6-release-focal.deb

3. Update the package repository:
sudo apt-get update -y
4. Install the Puppet server with the following command:
sudo apt-get install puppetserver -y
5. Open the puppetserver file by using:
sudo nano /etc/default/puppetserver
6. In the puppetserver file, modify the following line to change the memory size to 1GB:
JAVA_ARGS="-Xms1g -Xmx1g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"

7. Press Ctrl + X to close the puppetserver file. Type Y and press Enter to save the changes you made.

8. Start the Puppet service and set it to launch on system boot by using:
sudo systemctl enable puppetserver
sudo systemctl start puppetserver
To Verify whether puppetserver is installed. run
apt policy puppetserver

9. Check if the Puppet service is running with:
sudo systemctl status puppetserver

Now press q to come out of window.

Step 4: Install Puppet Agent on Client Node
1. Download the latest version of Puppet on a client node:
wget https://apt.puppetlabs.com/puppet6-release-focal.deb
2. Once the download is complete, install the package by using:
sudo dpkg -i puppet6-release-focal.deb
3. Update the package repository one more time:
sudo apt-get update -y
4. Install the Puppet agent by using:
sudo apt-get install puppet-agent -y
5. Open the Puppet configuration file:
sudo nano /etc/puppetlabs/puppet/puppet.conf
6. Add the following lines to the end of the Puppet configuration file to define the Puppet master information (optional) :
[main]
certname = puppetclient
server = puppetmaster

7. Press Ctrl + X to close the Puppet configuration file, then type Y and press Enter to save the changes.
8. Start the Puppet service and set it to launch on system boot by using:
sudo systemctl enable puppet
sudo systemctl restart puppet
9. Check if the Puppet service is running with:
sudo systemctl status puppet

Now press q to come out of window.

Step 5: Sign Puppet Agent Certificate
The first time you run the Puppet agent, it generates an SSL certificate and sends a signing request to the Puppet master. After the Puppet master signs the agent's certificate, it will be able to communicate  with and control the agent node.

First list the unsigned certificates on puppet master server

1. Using the Puppet master node, list all the available certificates:
sudo /opt/puppetlabs/bin/puppetserver ca list --all

2. Sign the certificates with:
sudo /opt/puppetlabs/bin/puppetserver ca sign --all
or to sign just one certificate....

Example:
The above command will list agent ip address.
  "your_puppet_Agent_Ec2_private_dns_name"  (SHA256) 46:19:79:3F:70:19:0A:FB:DA:3D:C8:74:47:EF:C8:B0:05:8A:06:50:2B:40:B3:B9:26:35:F6:96:17:85:5E:7C

sign the Puppet agent IP address.
sudo /opt/puppetlabs/server/bin/puppetserver ca sign --certname AgentEc2_private_dnsname

3. Use the following command to test the communication between the master and client nodes:
sudo /opt/puppetlabs/bin/puppet agent --test

Step 6 - Verifying installation by creating Manifests in Puppet Master

The puppet manifest file is the actual file which contains the configuration details for the agents. This file is centrally stored at Puppet Master.

sudo nano /etc/puppetlabs/code/environments/production/manifests/site.pp
#copy the below yellow lines in the above file
    file {'/tmp/puppet_test.txt':                        # resource type file and filename
    ensure  => present,                             # make sure it exists
    mode    => '0644',                              # file permissions
  content => "Hello from Puppet master to agent on ${ipaddress_eth0}!\n",  # Print the eth0 IP fact
    }
Press Ctrl S for saving and then enter
Press Ctrl X for exit after saving.

Step 7 - Apply Manifests in Puppet Agent
apply the changes in puppet agent by executing below command:
sudo /opt/puppetlabs/bin/puppet agent --test

You should see a file being modified in /tmp/puppet_works.txt in agent(node).
You can confirm by typing this command on puppet node 

sudo cat /tmp/puppet_test.txt
Hello from Puppet master to agent on IP_address!!
That's it! you have set up Puppet Master and configured agent on the target node successfully!