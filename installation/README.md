# Video Analytics Ansible Installer
This folder contains the ansible playbooks for installing/removing/managing the IBM Video Analytics software.

Unlike the provided installation scripts, this installer deploys a complete IVA system from a single control node. The Video Analytics software packages have to be downloaded once on the control node. The installer will copy these packages on the remote nodes during the installation process.
The installer will install all the pre-requisite system packages prior to the IVA deployment including the NVIDIA drivers and container runtime. It will also install useful monitoring tools like [nvtop](https://github.com/Syllo/nvtop).
A control node could be the managed node as well, an example of an inventory for a local installation is provided [here](inventories/sample-local.ini). In that case, the ansible script will stop at a certain point, you will have to reboot manually, and re-run the same playbook after the reboot in order to complete the installation of the NVIDIA driver.

Control Node Requirements
-------------------------
* **Ansible v2.9+**, **python3** and **pip** see [Ansible Documentation](https://docs.ansible.com/ansible/2.9/installation_guide/intro_installation.html#prerequisites)
* **pip3** and **netaddr** Python packages ( pip3 install netaddr )
* a folder with the downloaded IVA software

Managed Node Requirements
-------------------------
* Ubuntu 18.04-20.04 (x86_64 only) + sudo apt install net-tools openssh-server
* RedHat 7 (x86_64 and ppc64le): the managed node operating system must be registered in order to use yum package manager
* Python 2 or 3 must be installed on these OS but this is the case by default

## Inventory
The inventory file contains the list of nodes on which the installer will deploy IVA and also some configuration parameters. Sample inventory files are provided in the inventory folder. Inventory files come in 2 flavors, ini and yml files, you just need one of those.
It is recommended to have one inventory file per IVA environment. So it should never happen that we get several managed nodes in the [mils] section of an inventory file.
The installation playbook will discover the network topology from the inventory. By default, all SSEs will connect to the single MILS and all SSEs will use a single DLE.
If there are multiple DLEs, then each SSE should indicate which **dle_node** must be used. If there is no DLE in the inventory, the SSEs will not be configured to use a DLE system.
The installer will ping the MILS and DLE nodes from the SSEs and will ping the SSEs from the MILS in order to ensure the consistency of the inventory configuration.

We could have several systems in the [allinone] section for example if we install IVA on different computers in a classroom. 
Let's explore the content of the [sample.ini](https://github.com/IBM/video-analytics/installation/sample.ini) inventory file.

[all:vars]
----------
This section contains the variables that are used by the ansible playbooks

**software_dir** is the folder on the control node where the IVA software was downloaded. The playbook will copy and extract these archives on the managed nodes when needed

**watvid_dir** is the folder on the managed node where the IVA software will be deployed. For example, if watvid_dir = /opt, the IVA software will be deployed on /opt/watvid/...

**temp_dir** is the folder on the managed node where the installer will copy the software packages and will store temporary installation files. The installer might need up to 15 GB in the temporary folder.

[allinone]
----------
This section contains the managed "standalone" nodes like demo and POC systems where all components (mils,sse,dle) are installed on one system.
The IVA configuration on these nodes is not using the external IP address of the node but an internal docker0 bridge address. It means the system will still work when disconnected from the network or with a different IP address.

[mils]
------
This section contains the managed nodes where the mils component is installed.

[sse]
-----
This section contains the managed nodes where the sse component is installed.
Supported parameters are:
* **vfi**  yes/no, by default: no, use yes if the node is dedicated to Video Files Ingestion
* **dle_node** optional ip address of the dle node that is used by this sse

[dle]
-----
This section contains the managed nodes where the dle component is installed.
Supported parameters are:
* **dle_service**  to select a specific service file (low-memory, objects-only, people-only)


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

and "utility" playbooks:
* ansible-playbook -i <inventory file> va-cleanup.yml (to remove installation files on the managed nodes)
* ansible-playbook -i <inventory file> va-check-inventory.yml (to check the topology and make sure all nodes are reachable)

All playbooks support **mils**, **sse** and **dle** tags.
ansible-playbook -i <inventory file> va-restart.yml **--tags** "sse,dle" 
is equivalent to
ansible-playbook -i <inventory file> va-restart.yml **--skip-tags** "mils"

Installation playbooks have additional tags for installing/reinstalling specific packages or services.
  * **tools** (nvtop,arp-scan,...)
  * **nvidiadriver** (nvidia-driver)
  * **nvidiaruntime** (nvidia container runtime)
  * **docker** (docker service)
  * **sys** (system packages and repositories like epel, python3,..)

Supported playbooks/tags matrix
-------------------------------
Playbook/Tag | sys | docker | nvidiadriver | nvidiaruntime | mils | sse | dle | tools
---------------- | --- | ------ | ------------ | ------------- | ---- | --- | --- | -----
va-install | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes
va-reinstall | No | No | No | No | Yes | Yes | Yes | No
va-remove | No | No | No | No | Yes | Yes | Yes | No
va-start | - | Yes | - | - | Yes | Yes | Yes | -
va-restart | - | Yes | - | - | Yes | Yes | Yes | -
va-stop | - | Yes | - | - | Yes | Yes | Yes | -
 
Examples:
 * **ansible-playbook -i <inventory file> va-install.yml** will install all IVA components
 * **ansible-playbook -i <inventory file> va-restart.yml --tags "mils,sse"** will restart mils and sse components
 * **ansible-playbook -i <inventory file> va-reinstall.yml --tags "dle"** will re-install the dle component
 
Troubleshooting
---------------
* the version of the dependencies is determined from the VA software package names, so if you rename the packages after downloading them, you might end-up with incorrect versions of underlying components like the nvidia container runtime.
* the different nodes will ping each other before the installation in order to ensure proper connectivity, so all the nodes must be up and running. If that's not the case, then comment out the stopped nodes in the inventory, you will run the installation again later when thoset nodes will be available
* the installers are checking if the software is already present on the file system. If you want to reinstall something, then use va-reinstall and not va-install

 
