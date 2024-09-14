## Cluster Starter

### Description

These Ansible playbooks are intended to create, power on, power off and delete Kubernetes clusters on Proxmox VE and VMware vSphere. They're compatible with Kubespray inventories.

### Important Notes

* This is a new project, so you may expect some bugs and future changes that may break stuff
* You can optionally have a different network for communication among nodes
* These playbooks are not idempotent by now, so you can expect some errors if you run them more than once
* If you find any error, please [create a new issue](https://github.com/burbuja/cluster-starter/issues)
* Be careful if you already have virtual machines named ```node1```, ```node2```, ```node3``` and so on

### Example of Usage

```ShellSession
# Create a working directory
mkdir myproject
cd myproject

# Create a virtual environment and activate it
python3 -m venv venv
source venv/bin/activate

# Clone Cluster Starter and Kubespray repositories
git clone --single-branch --branch v0.2.1 https://github.com/burbuja/cluster-starter.git
git clone --single-branch --branch v2.24.0 https://github.com/kubernetes-sigs/kubespray.git

# Install requirements
pip install -U -r kubespray/requirements.txt
pip install -r cluster-starter/requirements.txt
ansible-galaxy collection install community.vmware

# Change directory
cd kubespray

# Copy inventory/sample as inventory/mycluster
cp -rfp inventory/sample inventory/mycluster

# Update Ansible inventory file with inventory builder
declare -a IPS=(10.10.1.3 10.10.1.4 10.10.1.5)
CONFIG_FILE=inventory/mycluster/hosts.yml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

# Optional:
# * Edit the node names
# * If you intend to use an additional network for communication among nodes, edit each
# "ip" and commment each "access_ip"
nano inventory/mycluster/hosts.yml

# If intend to use VMware vSphere, you should set at least "external_vsphere_vcenter_ip",
# "external_vsphere_vcenter_port", "external_vsphere_insecure", "external_vsphere_user",
# "external_vsphere_password", "external_vsphere_datacenter" (e.g.: ha-datacenter) and
# "external_vsphere_kubernetes_cluster_id" (if necessary)
nano inventory/mycluster/group_vars/all/vsphere.yml

# Create this new file as the example below
nano inventory/mycluster/group_vars/all/custom.yml

# Change directory
cd ..
cd cluster-starter

# Create a new cluster for Kubespray
ansible-playbook -i ../kubespray/inventory/mycluster/hosts.yml create.yml

# Verify whether the new cluster is already powered on
ansible -i ../kubespray/inventory/mycluster/hosts.yml -m ping all

# Change directory
cd ..
cd kubespray

# Deploy Kubernetes
ansible-playbook -i inventory/mycluster/hosts.yml cluster.yml --become

# Change directory
cd ..
cd cluster-starter

# Examples to power off, power on and delete cluster
ansible-playbook -i ../kubespray/inventory/mycluster/hosts.yml poweroff.yml
ansible-playbook -i ../kubespray/inventory/mycluster/hosts.yml poweron.yml
ansible-playbook -i ../kubespray/inventory/mycluster/hosts.yml delete.yml

# Enable the RBD module in Ubuntu Noble, for Rook
ansible-playbook -i ../kubespray/inventory/mycluster/hosts.yml rook-patch.yml --become
```

### Example for ```kubespray/inventory/mycluster/group_vars/all/custom.yml```

```yaml
### Hypervisor
cc_hypervisor: pve_ssh # pve_ssh or vsphere

### Uncomment and fill the following lines in case of cc_hypervisor: pve_ssh
# pve_host: 
# pve_user: root
# pve_pass: 
# vm_datastore: local-lvm
# vm_network_0: vmbr0
# # vm_network_1: vmbr1 # (optional)

### Uncomment and fill the following lines in case of cc_hypervisor: vsphere
### Don't forget to also edit the file 'kubespray/inventory/mycluster/group_vars/all/vsphere.yml'
# vm_datastore: datastore1
# vm_network_0: VM Network
# # vm_network_1: Kubernetes Network # (optional)
# vm_hardware_version: latest

### Ubuntu
ubuntu_version: noble # jammy or noble

### Virtual Machine
vm_num_cpus: 2
vm_memory_mb: 2048
vm_disk_size_gb_0: 40
# vm_disk_size_gb_1: 100 # (optional, useful for Rook)
vm_mask_0: 24
# vm_mask_1: 24 # (optional) 
vm_gateway_0: 10.10.1.1
vm_dns_0: 8.8.8.8
vm_keyboard_layout: en # en, es, latam...
vm_timezone: Etc/UTC # Etc/UTC, America/New_York, Europe/Madrid, America/Santiago...
vm_gecos: User
ansible_user: user
ansible_ssh_pass: changeme
```

### License

[Apache License version 2.0](https://github.com/burbuja/cluster-starter/blob/master/LICENSE)
