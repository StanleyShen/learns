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
