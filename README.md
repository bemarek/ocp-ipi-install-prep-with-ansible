# OpenShift cluster install

This repo contains the Ansible assets and can be used to install a OpenShift cluster. The Ansible assets are developed for vSphere IPI installation mode.

# Ansible
## Roles

There are two Ansible roles

- prepare - preparation of bastion host, download OpenShift binaries etc.
- install - preparation and kick off `openshift-installer`.

An example of the playbook.

  ```yaml
  - name: Prepare and install OpenShift cluster
    hosts: all
    gather_facts: true
    become: true
    roles:
      - name: prepare
        tags: prepare
      - name: install
        tags: install
  ```

## How to deploy

- Create a `./group_vars/all/auth.yml` file with credentials

  ```yaml
  vsphere_username: changeme
  vsphere_password: changeme
  pull_secret: changeme
  ```
- Prepare the inventory. These Ansible roles run on a **Bastion host** hence you can run it from your workstation or locally cloning the repo in the bastion host itself.

  ```yaml
  all:
    hosts:
    children:
      dev:
        hosts:
          bastionA:
            ansible_host: bastionA.dev
          bastionb:
            ansible_host: bastionB.dev
      qas:
        hosts:
  ```

- Provide cluster specific parameters in [group_vars](inventory/group_vars/) or [host_vars](inventory/host_vars/) if you have multiple clusters & bastion in one environment.

  ```yaml
  # Common params which are applicable to environment
  httpProxy: 
  httpsProxy: 
  noProxy: .local
  vsphere:
    vcenter: vcenter.example
    datacenter: DC
    cluster: Cluster
    datastore: WorkloadDatastore
    network: segment-sandbox-9wj4v
    folder: /Datacenter/vm/Workloads/folder"
    username: "{{ vsphere_username | default('changeme') }}"
    password: "{{ vsphere_password | default('changeme') }}"

  # Cluster list and specific params
  openshift_clusters:
  - name: bla
    baseDomain: base.example.com
    networking:
      clusterNetworkcidr: 10.128.0.0/14
      machineNetworkcidr: 192.168.25.0/24
      serviceNetworkcidr: 172.30.0.0/16
    platform:
      apiVIP: 192.168.25.201
      ingressVIP: 192.168.25.202
    controlplane:
      replicas: 3
    #   cpus:
    #   ram:
    #   disksizeGB:
    compute:
      replicas: 2
      # cpus:
      # ram:
      # disksizeGB:

  additionalTrustBundlePolicy: Proxyonly
  additionalTrustBundle: |
      -----BEGIN CERTIFICATE-----
      ...
      -----END CERTIFICATE-----

  ```
