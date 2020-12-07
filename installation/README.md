# Video Analytics Ansible Installer
This folder contains the ansible playbooks, allowing you to install IBM Video Analytics using ansible.

Unlike the IBM provided installation scripts, this installer deploys a complete IVA system from a single control node. The Video Analytics software packages have to be downloaded once on the control node. The installer will copy these packages on the remote nodes during the installation process.
The installer will install all the pre-requisite system packages prior to the IVA deployment including the NVIDIA drivers and container runtime. It will also install useful monitoring tools like [nvtop](https://github.com/Syllo/nvtop).
The control node is the node where the ansible playbooks will be executed from the command_line. The control node will connect remotely to the managed nodes and will copy the software from the control node to the managed nodes. A control node could be the managed nod as well if it is a single system environment but you will have to re-run manually the playbook after system restarts in order to complete the DLE installation.

Control Node Requirements
-------------------------
* Ansible v2.9+, python3 and pip see [Ansible Documentation](https://docs.ansible.com/ansible/2.9/installation_guide/intro_installation.html#prerequisites)
* pip and netaddr  python package: pip install netaddr
* a folder with the downloaded IVA software

Managed Node Requirements
-------------------------
* Ubuntu 18.04-20.04 (x86_64 only): sudo apt install net-tools openssh-server
* RedHat 7 (x86_64 and ppc64le): system must be registered in order to use yum package manager
* Python 2 or 3 must be installed on these OS but this is the case by default


## Inventory
The inventory file contains the list of nodes on which the installer will deploy IVA and also some configuration parameters. Sample inventory files are provided in the inventory folder.
It is recommended to have one inventory file per environment. So it should never happen that we get several managed nodes in the [mils] section of an inventory file.
The installation playbooks will discover the network topology from the inventory. By default, all SSEs will connect to the single MILS and all SSEs will use a single DLE.
If there are multiple DLEs, then each SSE should indicate which **dle_node** must be used. If there is no DLE in the inventory, the SSEs will not be configured to use a DLE system.
The installer will ping the MILS and DLE nodes from the SSEs and will ping the SSEs from the MILS in order to ensure the consistency of the inventory configuration.

We could have several systems in the [allinone] section for example if we install IVA on different computers in a classroom
Let's explore the content of the [sample_inventory.ini](https://github.com/IBM/video-analytics/installation/sample_inventory.ini)

[all:vars]
----------
This section contains the variables that are used by the ansible playbooks

**software_dir** is the folder on the control node where the IVA software was downloaded. The playbook will copy and extract these archives on the managed nodes when needed

**watvid_dir** is the folder on the managed node where the IVA software will be deployed. For example, if watvid_dir = /opt, the IVA software will be deployed on /opt/watvid/...

**temp_dir** is the folder on the managed node where the installer will copy the software packages and will store temporary installation files. The installer might need up to 15 GB in the temporary folder.

[allinone]
----------
This section contains the managed "standalone" nodes like demo and POC systems where all components (mils,sse,dle) are installed on one system.
The IVA configuration on the nodes configured here is not using the external IP address of the node but an internal docker0 bridge address. It means the system will still work when disconnected from the network or with a different IP address.

[mils]
------
This section contains the managed nodes where the mils component is installed.

[sse]
-----
This section contains the managed nodes where the sse component is installed.

[dle]
-----
This section contains the managed nodes where the dle component is installed
Supported parameter:
* **dle_node**  to select a specific service file (low-memory, objects-only, people-only)


Command Line Options
--------------------
There are "installation" playbooks to deploy an environment:
* ansible-playbook -i <inventory file> va-install.yml
* ansible-playbook -i <inventory file> va-reinstall.yml
* ansible-playbook -i <inventory file> va-remove.yml
and "management" playbooks to manage a deployed environment:
* ansible-playbook -i <inventory file> va-stop.yml
* ansible-playbook -i <inventory file> va-start.yml
* ansible-playbook -i <inventory file> va-restart.yml

All playbooks support **mils**, **sse** and **dle** tags.
ansible-playbook -i <inventory file> va-restart.yml **--tags** "sse,dle" 
is equivalent to
ansible-playbook -i <inventory file> va-restart.yml **--skip-tags** "mils"

Installation playbooks have additional tags for installing/reinstalling specific packages or services.
  * tools (nvtop,arp-scan,...)
  * nvidiadriver (nvidia-driver)
  * nvidiaruntime (nvidia container-runtime)
  * docker (docker service)
  * sys (system packages and repositories like epel, python3,..)

**Supported tags/playbook matrix**
Playbook/Tag | sys | docker | nvidiadriver | nvidiaruntime | mils | sse | dle | tools
---------------- | --- | ------ | ------------ | ------------- | ---- | --- | --- | -----
va-install | Yes | Yes | Yes | Yes | Yes | No | Yes | Yes
va-reinstall | Yes | Yes | Yes | Yes | Yes | No | Yes | Yes
va-remove | No | No | No | No | Yes | No | No | No
va-start | - | Yes | - | use docker | Yes | No | Yes | -
va-restart | - | Yes | - | use docker | Yes | No | Yes | -
va-stop | - | Yes | - | use docker | Yes | No | Yes | -
 
Examples:
 * **ansible-playbook -i <inventory file> va-install.yml** will install all IVA components
 * **ansible-playbook -i <inventory file> va-restart.yml --tags "mils,sse"** will restart mils and sse components
 * **ansible-playbook -i <inventory file> va-reinstall.yml --tags "nvidia-driver"** will re-install the nvidia-driver on DLE nodes
