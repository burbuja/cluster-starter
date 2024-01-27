# Cluster Starter

## Description

These Ansible playbooks are intended to create, power on, power off and delete Kubernetes clusters on VMware. They're compatible with Kubespray inventories.

## Important Notes

* This is a new project, so you may expect some bugs and future changes that may break stuff
* You MUST have a different network for communication among nodes until we change that
* You can't repeat the creation playbook without any errors
* If you find any error, please [create a new issue](https://github.com/burbuja/cluster-starter/issues)
* Be careful if you already have virtual machines named "node1", "node2" and so on

## Example of Usage

```ShellSession
# Create a working directory
mkdir myproject
cd myproject

# Create a virtual environment and activate it
python3 -m venv venv
source venv/bin/activate

# Clone Cluster Starter and Kubespray repositories
git clone --single-branch --branch v0.1 https://github.com/burbuja/cluster-starter.git
git clone --single-branch --branch v2.24.0 https://github.com/kubernetes-sigs/kubespray.git

# Install requirements
pip install -U -r kubespray/requirements.txt
pip install -r cluster-starter/requirements.txt
ansible-galaxy collection install community.vmware

# Copy kubespray/inventory/sample as kubespray/inventory/mycluster
cp -rfp kubespray/inventory/sample kubespray/inventory/mycluster

# Update Ansible inventory file with inventory builder
declare -a IPS=(10.10.1.3 10.10.1.4 10.10.1.5)
CONFIG_FILE=kubespray/inventory/mycluster/hosts.yml python3 kubespray/contrib/inventory_builder/inventory.py ${IPS[@]}

# Edit at least "node names"
# Optional: if you want to use an additional network for communication among nodes, edit each "ip" and commment each
# "access_ip"
nano kubespray/inventory/mycluster/hosts.yml

# Edit at least "external_vsphere_vcenter_ip", "external_vsphere_vcenter_port", "external_vsphere_insecure",
# "external_vsphere_user", "external_vsphere_password", "external_vsphere_datacenter" and
# "external_vsphere_kubernetes_cluster_id"
nano kubespray/inventory/mycluster/group_vars/all/vsphere.yml

# Create this new file as the example below
nano kubespray/inventory/mycluster/group_vars/all/custom.yml

# Create a new cluster for Kubespray
ansible-playbook -i kubespray/inventory/mycluster/hosts.yml cluster-starter/create.yml

# Verify whether the new cluster is alive
ansible -i kubespray/inventory/mycluster/hosts.yml -m ping

# Deploy Kubespray
ansible-playbook -i kubespray/inventory/mycluster/hosts.yml kubespray/cluster.yml --become
```

# Example for ```kubespray/inventory/mycluster/group_vars/all/custom.yml```

```yaml
# Ubuntu
ubuntu_version: jammy
ubuntu_ova_remote_path: "https://cloud-images.ubuntu.com/{{ubuntu_version }}/current/{{ ubuntu_version }}-server-cloudimg-amd64.ova"
ubuntu_ova_local_path: "files/{{ ubuntu_version }}-server-cloudimg-amd64.ova"

# Virtual Machines
vm_datastore: datastore1
vm_network_0: VM Network
vm_network_1: Kubernetes Network
vm_memory_mb: 2048
vm_num_cpus: 2
vm_hardware_version: latest
vm_disk_size_gb_0: 40
vm_disk_size_gb_1: 100
vm_mask_0: 24
vm_mask_1: 24
vm_gateway_0: 10.10.1.1
vm_dns_0: 8.8.8.8
vm_gecos: User

# Ansible
ansible_user: user
ansible_ssh_pass: changeme
```

# License

[Apache License version 2.0](https://github.com/burbuja/cluster-starter/blob/master/LICENSE)
