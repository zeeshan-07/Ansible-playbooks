                                                Monitoring Infrastructure for CEPH Optimization
Purpose:
The purpose of this task is to get Monitoring tools (Grafana, Prometheus, Nod exporter) and load generator (FIO) configured through Automation. We did this automation through multiple ansible playbooks. Actually, we are using this monitoring infrastructure for benchmarking of CEPH Optimization in RHOSP environment.

Tools:

Here we are using four tools they are

i-	FIO 

FIO is a versatile IO workload generator. FIO is an I/O tool meant to be used both for benchmark and stress/hardware verification. It has support for 19 different types of I/O engines. It was flexible enough to allow detailed workload setups, and it contains the reporting tools necessary to make sense of the data after tests are complete It can work on block devices as well as files. We are using FIO for benchmarking of RHOSP CEPH storage. It will be configured on VMs which will be created on overcloud compute nodes. 

ii-	Node Exporter
Node Exporter is a Prometheus exporter for hardware and OS metrics with pluggable metric collectors. It allows to measure various machine resources such as memory, disk and CPU utilization. In our case Node exporter will be installed on overcloud nodes and will collect metrics and send these stats to Prometheus.

iii-	Prometheus
Prometheus is an open source, metrics-based monitoring system. Prometheus does one thing and it does it well. It has a simple yet powerful data model and a query language that lets you analyses how your applications and infrastructure are performing. It does not try to solve problems outside of the metrics space, leaving those to other more appropriate tools. Prometheus will be installed on Director and will communicate with node exporter and Grafana.

iv-	Grafana
Grafana is an open source solution for running data analytics, pulling up metrics that make sense of the massive amount of data & to monitor our apps with the help of cool customizable dashboards. In our case Grafana will be configured on Linux Jump Host and we are using it to see stats of overcloud nodes in graphical form. It will use Prometheus as a data source. 







Playbooks and their Working

There are Seven playbooks, and all of them are imported in master-playbook. Each playbook will perform some specific tasks, they are

•	vm_env_setup.yml
This playbook will setup network, router, ssh keys, and will copy ssh keys from director to linux jump host.

•	vms_creation.yml
will create VMs, assign floating IP, and ping from external network on Openstack 

•	fio_deployment.ym
Packages of FIO will be installed on created VMs which are on different computer nodes of Openstack. FIO is load generator tool, use for the block storage as well for file system.

•	setup_nodexporter.yml
Node exporter will be installed on overcloud nodes (controller, compute, storage)

•	prometheus_env_setup.yml
Prometheus will be installed on director VM. It is configured in such a way that it will get los from node exporters that are installed on overcloud nodes. After collecting logs from node exporter, it will send to Grafana which is running on Linux jump host.

•	grafana_setup.yml
Setup Grafana on admin node for graphical view, and it will get logs from Prometheus.

•	setup_ansible.yml

Will install ansible, and will clone https://github.com/MuhammadAsif1/Ansible-playbooks.git on director. Then it will run one of the cloned playbook (setup_nodexporter.ym) on director, because overcloud nodes have IPs from private network and they are only accessible from director.

How to run?

On Linux jump host we must clone this repository 
git clone /etc/ansible/https://github.com/MuhammadAsif1/Ansible-playbooks.git
Then run this command
ansible-playbook -i /etc/ansible/Ansible-playbooks/hosts /etc/ansible/Ansible-playbooks/master-playbook.yml --key-file "/etc/ansible/ssh-key.pem"
Log file of ansible will be in /var/log/ansible.log directory.
There is an extra playbook (vms_delete.yml), it can be used to delete created VMs and release floating IPs. 

Limitations:

There are few limitations in this automation, so they are
1-	You must install Ansible on Linux Jump host Manually, because master-playbook will run on this localhost.

2-	By default, we have 10 vm names, and floating IPs respectively in a array in vm creation playbook, so if you want to create more or less vms then you have to scale this array manually according to your requirement. Names and floating ips assigned to an array are unique.

3-	For different stamps you have to change rc file name according to your stamp in vm creation, deletion, and environment setup playbooks.  

