BOSH Networks
====================================
A BOSH network is an IaaS-agnostic representation of the networking layer. The Director is responsible for configuring each deployment job’s networks with the help of the BOSH Agent and the IaaS. Networking configuration is usually assigned at the boot of the VM and/or when network configuration changes in the deployment manifest for already-running deployment jobs.

There are three types of networks that BOSH supports:
- manual - require that users specify one or more subnets and BOSH determines how to assign IPs to each job instance
- dynamic - BOSH defers IP selection to the IaaS
- vip - BOSH allows users to assign IPs to VMs.

Each type of network supports one or both IP reservation types:
- static - IP is explicitly requested by the user in the deployment manifest
- automatic - IP is selected automatically based on the network type

| | manual network | dynamic network | vip network |
| ------------ | ------------ |--------|-------|
| static IP assignment | Supported | Not supported | Supported |
| automatic IP assignment | Supported, default | Supported | Not supported |

General Structure
-------------------------
Networking configuration is usually done in three steps:
- Configuring the IaaS: Outside of BOSH’s responsibility
	- Example on AWS: User creates a VPC and subnets with routing tables.
- Adding networks section to the deployment manifest to define networks used in this deployment
	- Example: User adds a manual network with a subnet and adds AWS subnet ID into the subnet’s cloud properties.
- Adding network associations for one or more networks to each deployment job

All deployment manifests have a similiar structure in terms of network definitins and associations:
```
# Network definitions
networks:
- name: my-network
  ...
- name: my-other-network
  type: ...
  # IaaS specific attributes
  cloud_properties: { ... }

jobs:
- name: my-job
  # Network associations for `my-job`
  networks:
  - name: my-network
  ...

- name: my-multi-homed-job
  networks:
  - name: my-network
  - name: my-other-network
  ...

- name: my-static-job
  networks:
  - name: my-network
    # Static IP reservations for `my-job`
    static_ips: [IP1]
  ...
```

Manual Networks
-------------------
Manual networking allows you to specify one or more subnets and let the Director choose available IPs from one of the subnet ranges.
A subnet definition specifies the CIDR range and, optionally, the gateway and DNS servers. In addition, certain IPs can be blacklisted(The Director will not use thses IPs) via the **reserved** property.
Each manual network attached to a job instance is typically represented as its own NIC in the IaaS layer.
Schema for manual network definition:
- name [String, required]: Name used to reference this network configuration
- type [String, required]: Value should be manual
- subnets [Array, required]: Lists subnets in this network
	- range [String, required]: Subnet IP range that includes all IPs from this subnet
	- gateway [String, required]: Subnet gateway IP
	- dns [Array, optional]: DNS IP addresses for this subnet
	- reserved [Array, optional]: Array of reserved IPs and/or IP ranges. BOSH does not assign IPs from this range to any VM
	- static [Array, optional]: Array of static IPs and/or IP ranges. BOSH assigns IPs from this range to jobs requesting static IPs. Only IPs specified here can be used for static IP reservations.
	- cloud_properties [Hash, optional]: Describes any IaaS-specific properties for the subnet. Default is {} (empty Hash).

```
networks:
- name: my-network
  type: manual
  subnets:
  - range:    10.10.0.0/24
    gateway:  10.10.0.1
    dns:      [10.10.0.2]

    # IPs that will not be used for anything
    reserved: [10.10.0.2-10.10.0.10]
    cloud_properties: {subnet: subnet-9be6c3f7}

  - range:   10.10.1.0/24
    gateway: 10.10.1.1
    dns:     [10.10.1.2]

    # IPs that can only be used for static IP reservations within this subnet
    static: [10.10.1.11-10.10.1.20]

    cloud_properties: {subnet: subnet-9be6c6gh}
```
Manual networks use automatic IP reservation by default. They also support static IP reservation. To assign specific IPs to instances of the deployment job, they must be specified in deployment job's **networks** section, in the **static_ips** property for the associated network. That network's subnet definition must also specify them in its **static** property:
```
jobs:
- name: my-job-with-static-ip
  instances: 2
  ...
  networks:
  - name: my-network

    # IPs associated with 2 instances of `my-job-with-static-ip` job
    static_ips: [10.10.1.11, 10.10.1.12]
```

common problem that you may run into is configuring multiple deployments to use overlapping IP ranges. The Director does not consider an IP to be “used” even if the Director used that IP in a different deployment. There are two possible solutions for this problem:
- reconfigure one of the deployments to use a different IP range
- use the same IP range but configure each deployment such that reserved IPs exclude the deployment from each other.

Dynamic Networks
-----------------------
Dynamic networking defers IP selection to the IaaS. For example, AWS assigns a private IP to each instance in the VPC by default. By associating a deployment job to a dynamic network, BOSH will pick up AWS-assigned private IP addresses.

Schema for dynamic network definition:
- name [String, required]: Name used to reference this network configuration
- type [String, required]: Value should be dynamic
- dns [Array, optional]: DNS IP addresses for this network
- cloud_properties [Hash, optional]: Describes any IaaS-specific properties for the network. Default is {} (empty Hash).

```
networks:
- name: my-network
  type: dynamic
  dns:  [10.10.0.2]
  cloud_properties: {subnet: subnet-9be6c3f7}
```

VIP(Virtual IP) Networks
-----------------------
VIP networking enables the association of an IP address that is not backed by any particular NIC. This flexibility enables users to remap a virtual IP to a different instance in cases of a failure.

VIP network attachment is not represented as a NIC in the IaaS layer. In the AWS CPI, it is implemented with elastic IPs. In OpenStack CPI, it is implemented with floating IPs.

VIP networking only supports static IP reservations.

Schema for VIP network definition:
- name [String, required]: Name used to reference this network configuration
- type [String, required]: Value should be vip
- cloud_properties [Hash, optional]: Describes any IaaS-specific properties for the network. Default is {} (empty Hash).

```
networks:
- name: my-network
  type: vip

jobs:
- name: my-job
  ...
  networks:
  - name: my-network
    static_ips: [54.47.189.8]
```

Multi-homed VMs
--------------
A deployment job can be configured to have multiple IP addresses (multiple NICs) by being on multiple networks. Given that there are multiple network settings available for a deployment job, the Agent needs to decide which network’s DNS settings to use and which network’s gateway should be the default gateway on the VM. Agent performs such selection based on the network’s default property specified in the deployment job.

Schema for default property:

- default [Array, optional]: Configures this network to provide its settings for specific category as a default. Possible values are: dns and gateway. Both values can be specified together.

```
networks:
- name: my-network-1
  type: dynamic
  dns: [8.8.8.8]

- name: my-network-2
  type: dynamic
  dns: [4.4.4.4]

jobs:
- name: my-multi-homed-job
  ...
  networks:
  - name: my-network-1
    default: [dns, gateway]
  - name: my-network-2

- name: my-other-multi-homed-job
  ...
  networks:
  - name: my-network-1
    default: [dns]
  - name: my-network-2
    default: [gateway]
```


