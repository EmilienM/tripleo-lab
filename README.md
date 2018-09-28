# Custom Lab for TripleO
A simple way to bootstrap a bunch of VMs on a libvirt host. Gives more control
than quickstart in some cases.

## Specs
It is expected to run the following resources:
- 1 undercloud
- 3 controllers
- 2 computes
- 3 ceph-storage

The amount of used memory for that env is about 52G, and about 20 CPU cores.

## Usage
### Standard deploy:
```Bash
ansible-playbook builder.yaml
```

It will deploy a containerized undercloud and prepare the baremetal.json file
you will use in order to register nodes in ironic.

### Redeploy the lab
If you issue a ```lab-destroy``` on the builder, you will need to redeploy the
lab:
```Bash
ansible-playbook builder.yaml --tags lab
```

### I have my undercloud - may I deploy the overcloud images
Yes !
```Bash
ansible-playbook builder.yaml --tags overcloud-images
```
### Extend the lab
You can pass variables and environment files using the ```-e``` option, like:
```Bash
ansible-playbook builder.yaml --skip-tags overcloud-images --tags lab -e @environments/3ctl-1compute.yaml
```
This will start a lab with just 3 controllers and 1 compute.

#### Variables
*vms*
  Hash with the following entries:
  - name (string)
  - cpu (int)
  - memory (int, MB)
  - interfaces (list)
    - mac (MAC address)
  - autostart (bool)
  - swap (string, for example 10G)

*centos_mirror*
  Create a CentOS repository mirror (default to no)

*mirror_base_directory*
  Location of the mirrors on your filesystem (default to /srv/mirror)

*manage_mirror_device*
  Toggle management of the dedicated device for the mirror content (default to yes)

*centos_mirror_device*
  Block device name for mirror content.

  Please note it will create a partition and format it as XFS.

*virt_user*
  Username on the builder (will be created)

*basedir*
  Base directory for all the VM images

*undercloud_password*
  Root password on the undercloud VM (debug purpose)

*undercloud_config*
  Hash representing ini configuration for undercloud.conf file

*tripleo_repo_version*
  Tripleo-repos package name/version

*overcloud_images*
  Hash with the following entries:
  - file (string)
  - content (string)

*containerized_undercloud*
  Boolean

*ci_tools*
  Install CI tools (Boolean)

*deploy_undercloud*
  Boolean

*proxy_host*
  Allows to use a proxy on the lab - put IP:PORT as value

*patches*
  List of hashes with the following entries:
  - name (string) - package name
  - refs (string) - patch reference
