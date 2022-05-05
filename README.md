## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted in the diagram below.

![Network Diagram](images/Network_Diagram.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above, or select playbooks in the `ansible` folder may be used to install only certain pieces of it - such as only Filebeat or the DVWA.

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

`Load balancing` ensures that the application will be highly available, in addition to restricting inbound access to the network.
Load balancers can help protect against DDoS attacks and help keep the web server available if one of the machines goes down.

Integrating an `ELK server` allows system admins to easily monitor the vulnerable VMs for changes to the file system of the machines on the network and allows us to monitor a suite of system metrics. 

`Filebeat` logs and visualizes information about the file system including which files have changed and when. 
`Metricbeat` logs and visualizes data from the systems and services running on the server such as uptime, permission escalation attempts, and more.

The configuration details of each machine may be found below.

| Name                 | Function   | IP Address | Operating System | Public IP               |
|----------------------|------------|------------|------------------|-------------------------|
| Jump Box Provisioner | Gateway    | 10.0.0.4   | Linux            | Public Jump Box IP      |
| Web 1                | Web Server | 10.0.0.5   | Linux            | Load Balancer Public IP |
| Web 2                | Web Server | 10.0.0.6   | Linux            | Load Balancer Public IP |
| ELK                  | Monitor    | 10.1.0.4   | Linux            | ELK Public IP           |

**Note** We have provisioned an Azure load balancer in front of Web 1 and Web 2, they both share the load balancer's front end IP.

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box Provisioner can accept a remote connection. Access to this machine is only allowed from the public IP address of the system administrator.

Machines within the network can only be accessed by the Jump Box Provisioner.

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses | Ports Open            |
|----------|---------------------|----------------------|-----------------------|
| Jump Box | Yes                 | sysadmin's ip        |                       |
| Web 1    | No                  | 10.0.0.4             | 80 (HTTP)             |
| Web 2    | No                  | 10.0.0.4             | 80 (HTTP)             |
| ELK      | No                  | 10.0.0.4             | 5601 to sysadmin's ip |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which allows us to rapidly and securely deploy this ELK configuration on any machine to monitor its network traffic. Once the playbook has ran, logs & traffic can be monitored remotely. 

The `ansible/install-elk.yml` playbook implements the following tasks:

- Installs Docker & PIP3
- Downloads and launches an ELK Docker image
- Enables the ELK service to start on boot

The Filebeat & Metricbeat playbooks are similar:

- Downloads and installs Metricbeat onto the _target machines_
- Creates config file & enables Docker module
- Starts Metricbeat service, enables service to start on boot

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance. *Note* You must SSH into the ELK machine from your Ansible container, for example `ssh@RedAdmin10.1.0.4`, before running `docker ps`!

![Docker Output](images/docker_ps_ELK.png)

### Target Machines & Beats
This ELK server is configured to monitor the Web 1 and Web 2 machines (10.0.0.5, 10.0.0.6). We can change what machines are monitored by changing these IP addresses in the `hosts` files under `[webservers]`.

We can install Metricbeat and Filebeat on these machines by running `filebeat-playbook` and `metricbeat-playbook`.

These Beats allow us to collect a suite of data from the web servers. With Metricbeat we can view CPU usage, memory usage, inbout & outbound traffic metrics, spikes in web traffic, and more. Filebeat allows us to read and sort logs from our web machines, including changes to the authentication logs and the file system.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured.

To configure the Ansible container first make sure Docker is installed on the Jumpbox.
- Run `sudo apt update`
      `sudo apt install docker.io` to isntall Docker
- We can check if docker is up and running with `sudo systemctl status docker`

To install Ansible, we will first pull it from the cyberxsecurity repo and then install the container:
- `sudo docker pull cyberxsecurity/ansible`
- `sudo docker run -ti cyberxsecurity/ansible bash`
The Ansible control node should now be installed. `exit` the container.

When we are ready to run one of the playbooks to configure a DVWA, ELK, Metricbeat, or Filebeat we must first ssh into our Jumpbox and start the docker container.
- `sudo docker container list -a` shows us the name of our Ansible container
- `sudo docker start [name of container]` starts Ansible.
- `sudo docker attach [name of container]` logs us into Ansible
**Note** You must change the hosts file inside the Ansible container at `/etc/ansible` to match the IP addresses of your webservers and ELK machines. Your virtual network may be slightly different than the network used in this project. Ansible's connection to the other machines can be tested with `ansible all -m ping`.

Once we have configured the hosts file and copied the `config` and `playbook` files from this repo into the Ansible container at `/etc/ansible/` we can run our playbooks to configure the targeted `hosts` with the `ansible-playbook` command.
- `ansible-playbook pentest.yml` will install the D*mn Vulnerable Web Application on our webservers.
      We can test if the installation worked as expected and access the DVWA by navigating to `http://[public IP of the loadbalancer]/setup.php` in a web browser from sysadmin's machine.

- `ansible-playbook install-elk.yml` will install the ELK stack on our ELK server.
      We can test if the installation worked as expected and access Kibana by navigating to `http://[ELK server IP]:5601/app/kibana#/home` in a web browser from sysadmin's machine.

- `ansible-playbook filebeat-playbook.yml` will install Filebeat monitoring on our webservers.
      **NOTE** You may need to change the "hosts" IP address in your `filebeat-config` file to the IP of your ELK server if it is different than the machine depicted in this repo!

- `ansible-playbook metricbeat-playbook.yml` will install Metricbeat monitoring on our webservers.
      **NOTE** You may need to change the "hosts" IP address in your `metricbeat-config` file to the IP of your ELK server if it is different than the machine depicted in this repo!