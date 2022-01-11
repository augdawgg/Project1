## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![](C:\Users\augus\Desktop\project1\Images\topography.png.png)

This document contains the following details:
- Description of the Topology!
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly usable, while restricting unwanted access to the network. They ensure equal traffic distribution to all machines in the network.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the _____ and system _____.
- Filebeat watches for access to system logs or specific system files.
- Metricbeat collects metrics from machines on the network.

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function       | IP Address | Operating System |
| -------- | -------------- | ---------- | ---------------- |
| Jump Box | Gateway        | 10.0.0.4   | Linux            |
| Web-1    | Server         | 10.0.0.5   | Linux            |
| Web-2    | Server         | 10.0.0.6   | Linux            |
| elkbox   | Logging server | 10.1.0.5   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the _____ machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- My IP address

Machines within the network can only be accessed from the Jump Box, and the elkbox can be used from my personal machine (only on port 5601).

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
| -------- | ------------------- | -------------------- |
| Jump Box | Yes                 | Personal             |
| Web-1    | No                  | 10.0.0.4             |
| Web-2    | No                  | 10.0.0.5             |
| elkbox   | Yes                 | Personal             |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because installation was made easier by the fact that the machine only needed to configure itself with user guidance.

The playbook is responsible for downloading and installing docker to all the machines, as well as its python wrapper and docker, the command line tool.

```
---

- name: Config Web VM with Docker
  hosts: elk
  become: true
  tasks:

  - name: Use more memory
    sysctl:
      name: vm.max_map_count
      value: '262144'
      state: present

  - name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present

  - name: download and launch a docker elk container
    docker_container:
      name: elk
      image: sebp/elk:761
      state: started
      restart_policy: always
      published_ports:
        - 5601:5601
        - 9200:9200
        - 5044:5044
```



### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1 at 10.0.0.4
- Web-2 at 10.0.0.5

We have installed the following Beats on these machines:
- FileBeat
- MetricBeat

These Beats allow us to collect the following information from each machine:
- FileBeat - Collects log files from across our network
- MetricBeat - Collects statistics on the network

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the configuration file to the affected VM's.
- Update the hosts file with the IP of the ELK server as well as the webservers it will be monitoring.
- Run the playbook, and navigate to the Kibana web app to check that the installation worked as expected.

```
---
- name: install filebeat on dvwa boxes
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: install filebeat
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: move filebeat.yml
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: config system module
    command: filebeat modules enable system

  - name: set up filebeat
    command: filebeat setup

  - name: start filebeat
    command: service filebeat start

  - name: systemd make filebeat start on machine boot
    systemd:
      name: filebeat
      enabled: yes
```



```
---
- name: install metricbeat
  hosts: webservers
  become: yes
  tasks:

  - name: download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

  - name: move metricbeat play
    copy:
      src: /etc/ansible/metricbeatconfig.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: configure system
    command: metricbeat modules enable docker

  - name: setup metric beat
    command: metricbeat setup

  - name: start metricbeat
    command: service metricbeat start

  - name: enable service metricbeat on machine boot
    systemd:
      name: metricbeat
      enabled: yes
```



