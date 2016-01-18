Deployment Basic
====================================================================

A [deployment](http://bosh.io/docs/deployment.html) is a collection of VMs and persistent disks. A deployment is split into smaller logical units called deployment jobs.

Deployment Jobs
-------------------
A deployment job is a logical unit, defined by the cluster operator, that represents a long running service or a short running task(an errand). Deployment jobs are defined in the deployment manifest. A job can be configured to run multiple copies of itself. The operator usually decides which jobs is needed to make the deployment do something useful. For example, when creating a Redis deployment, the operator will ad Redis server job, and might also decide to add a Redis slave job.

Each job needs to be backed by an actual software - releases. An operator, when defining a job in the manifest, has to specify which release(s) make up this job.
```
# Associates redis release with this deployment
releases:
- {name: redis, version: 12}

jobs:
- name: redis-master
  instances: 1

  # Associates redis-master with redis release
  templates:
  - {name: redis-server, release: redis}

  resource_pool: redis-servers
  networks:
  - name: default
```

Resource Pools
------------------
A resource pool is a collection of machines (VMs) that are created from a specific stemcell. Resource pools are also defined by the operator in the deployment manifest. In addition to specifying a stemcell, the resource pool definition allows the operator to specify VM size, instance type, and other IaaS-specific characteristics that are used when creating VMs.

```
resource_pools:
- name: redis-servers
  network: default

  # Association with Ubuntu Trusty stemcell for AWS
  stemcell:
    name: bosh-aws-xen-ubuntu-trusty-go_agent
    version: 2765

  # IaaS specific properties
  cloud_properties:
    instance_type: m1.medium
```

Compilation
-------------------
Most VMs in the deployment belong to a specific resource pool; however, the Director also creates compilation worker VMs for release compilation. The Director will compile each release on every necessary stemcell defined in the resource pools section. 
```
compilation:
  workers: 2
  network: default

  # IaaS-specific properties
  cloud_properties:
    instance_type: m1.medium
```

Persistent Disks
--------------------
Some deployment jobs may need to store persistent data – data that lives as long as the job instance is defined, independent of whether the job instance VM is deleted or recreated. 
```
jobs:
- name: redis-slave
  instances: 2
  templates:
  - {name: redis-server, release: redis}

  # Associated 10GB disk
  persistent_disk: 10_240

  resource_pool: redis-servers
  networks:
  - name: default
```

Networks
----------------------------
A deployment job must also reference at least one network. A BOSH network is an IaaS-agnostic representation of the networking layer. Each job instance gets an IP from its associated networks.
-	manual - require that users specify one or more subnets and BOSH determines how to assign IPs to each job instance
-	dynamic - BOSH defers IP selection to the IaaS
-	vip - BOSH allows users to assign IPs to VMs.
```
networks:
- name: default
  type: manual

  subnets:
  - range:    10.10.0.0/24
    gateway:  10.10.0.1
    dns:      [10.10.0.2]
    reserved: [10.10.0.2-10.10.0.10]

    # IaaS specific properties
    cloud_properties:
      subnet: subnet-9be6c3f7
```

