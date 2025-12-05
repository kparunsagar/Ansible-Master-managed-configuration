
# Login
az login

# Create a resource group
az group create --name ansible-demo-rg --location centralindia

# Create the master VM
az vm create \
  --resource-group ansible-demo-rg \
  --name ansible-master \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys

# Create Managed VMs (repeat or loop for more)
az vm create \
  --resource-group ansible-demo-rg \
  --name ansible-machine1 \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys

az vm create \
  --resource-group ansible-demo-rg \
  --name ansible-machine2 \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys

az vm create \
  --resource-group ansible-demo-rg \
  --name ansible-machine3 \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys

# Grab the public IPs to use later
az vm list-ip-addresses -g ansible-demo-rg -o table

# From your local machine, SSH into master
ssh azureuser@<master-public-ip>

# Ensure you have a keypair (created by Azure); if not, create one
ls ~/.ssh/id_rsa || ssh-keygen -t rsa -b 4096

# master

ls -l ~/.ssh/id_rsa.pub  (copy key)

# Managed node
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys  (paste)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Test from Master

ssh -i ~/.ssh/id_rsa azureuser@managedvm

# On the Master
sudo apt update
sudo apt install -y ansible python3-pip
ansible --version
mkdir -p ~/ansible-demo && cd ~/ansible-demo

# create inventory
# ~/ansible-demo/inventory.ini
[Managed Nodes]
machine1 ansible_host=<machine1-public-ip> ansible_user=azureuser
machine2 ansible_host=<machine2-public-ip> ansible_user=azureuser
machine3 ansible_host=<machine3-public-ip> ansible_user=azureuser

# Test ansible connectivity
ansible -i inventory.ini all -m ping

----------------------------

##########
ansible-demo/
├── inventory.ini
├── site.yml             # Main playbook (calls roles)
├── roles/
│   ├── git/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── vars/
│   │       └── main.yml
│   └── docker/
│       ├── tasks/
│       │   └── main.yml
│       └── vars/
│           └── main.yml
├── group_vars/
│   └── managedvars.yml       # Variables for all managed server hosts

###############
-------------
# run

ansible-playbook -i inventory.ini site.yml

