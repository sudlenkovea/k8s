# RKE2 Kubernetes Cluster Automation with Ansible

This project speeds up and simplifies the deployment, configuration, and expansion of an RKE2 Kubernetes cluster using Ansible. It simplifies the management of your Kubernetes infrastructure by encapsulating complex tasks into Ansible roles and playbooks that can be executed with simple commands.

## Requirements

- **Ansible 2.9+**: Ensure you have Ansible installed on your control machine.
- **SSH Access**: SSH access and sudo privileges on all target nodes.
- **RKE2 Requirements**: Target nodes must meet the [RKE2 requirements](https://docs.rke2.io/install/requirements/).

## Project Structure
```sh
.
├── rke2-inventory-example.yaml # Inventory file for defining hosts and variables
├── rke2_setup/ # Roles directory containing reusable roles
│ └── tasks/ # Role for RKE2 setup and configuration
│ └── handlers/
│ └── templates/
│ └── vars/
└── rke2_setup.yaml # Main playbook for cluster setup and expansion
```

## Usage

### Cluster Setup

To create or reconfigure your RKE2 cluster, use the `cluster_setup` tag with the `rke2_setup.yaml` playbook. This operation will configure the cluster according to the definitions in your `rke2-inventory-example.yaml` and roles.

```sh
ansible-playbook -i rke2-inventory-example.yaml rke2_setup.yaml --become --tag cluster_setup -u sudo_enabled_or_root_user
```
This command initializes the RKE2 servers and agents defined in your inventory, applying all necessary configurations for a functional Kubernetes cluster.

### Cluster Expansion
To add new nodes to your existing RKE2 cluster (expansion), use the cluster_expand tag with the rke2_setup.yaml playbook. Before running, ensure the new nodes are defined in your inventory.yaml.

```sh
ansible-playbook -i inventory.yaml rke2_setup.yaml --become --tag cluster_expand -u sudo_enabled_or_root_user
```
This command applies configurations only necessary for integrating new nodes into the cluster, expanding your cluster's capacity.

### Customizing Your Configuration
You can customize your cluster configuration by editing the inventory.yaml file to specify the nodes, roles, and any variable overrides needed for your specific setup. Role-specific variables and templates can also be modified to fine-tune the behavior of your cluster deployment and management.

## Additional Notes
Ensure all target nodes have network connectivity and can be accessed via SSH from the control machine.
Review and adjust role variables and templates to match your cluster requirements and security policies.
Use Ansible's --check flag to perform a dry run of your playbooks and identify any potential issues before applying changes to your live environment.

## Authors

- [Evgenii Sudlenkov](https://github.com/sudlenkovea)
