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

## Prerequisite
### Working ssh-agent on ansible-host
A working ssh-agent must be runnin on the host launching ansible playbook. It
is needed in order to hot-load the private ssh key for the undercloud access.

In case you're running the playbook for a "middle machine", you can just forward
the agent from your workstation.

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

### I have my undercloud and want to replay only specific tasks on it
Just limit with the tags, and be sure to have the ```inventory``` tag. For
example, in order to just manage the overcloud images:
```Bash
ansible-playbook builder.yaml -t inventory -t overcloue-images
```

```
### Extend the lab
You can pass variables and environment files using the ```-e``` option, like:
```Bash
ansible-playbook builder.yaml --skip-tags overcloud-images --tags lab -e @environments/3ctl-1compute.yaml
```
This will start a lab with just 3 controllers and 1 compute.

### Validations
You can run tempest validations against standalone and undercloud deploy. In order to run
just the validations and avoid having to wait to much time, you can run the following command:
```Bash
ansible-playbook builder.yaml -t inventory -t validations
```

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
  List of hashes representing ini configuration for undercloud.conf file
```YAML
---
undercloud_config:
  - section: DEFAULT
    option: container_cli
    value: podman
```

*tripleo_repo_version*
  Tripleo-repos package name/version

*overcloud_images*
  Hash with the following entries:
  - file (string)
  - content (string)

*containerized_undercloud*
  Boolean, default "yes"

*use_heat*
  Whether to add `--use-heat` to undercloud install command (Boolean).
  Defaults "no"

*ci_tools*
  Install CI tools (Boolean)

*deploy_undercloud*
  Boolean

*deploy_overcloud*
  Boolean

*proxy_host*
  Allows to use a proxy on the lab - put IP:PORT as value

*patches*
  List of hashes with the following entries:
  - name (string) - package name
  - refs (string) - patch reference
```YAML
patches:
  - name: 'tripleo-heat-templates'
    refs: '33/600533/8'
```

*synchronize*
  List of hashes with the following entries:
  - name
  - base
  - dest (defaults to /home/stack/tripleo/)
  Synchronize with rsync from your ansible controller to remote
```YAML
synchronize:
  - name: tripleo-heat-templates
    base: /home/user/work
    dest: /home/stack/tripleo/
```

*synchronize_default_dest*
  String. Used if you don't set "dest" in the synchronize hash.
  Default /home/stack/tripleo/

*custom_rpms*
  List of custom remote RPMS to install on your undercloud node

*standalone*
  Deploy "standalone" tripleo (Bool, default no)

*standalone_ceph*
  Deploy ceph in standalone tripleo (Bool, default no)

*standalone_container_cli*
  Deploy standalone using said container CLI (String, default docker)

*semodules*
  Build modules based on .te files you can provide. List of dicts.
```YAML
semodules:
  - name: foobar
    src: /full/path/foobar.te
```

*overcloud_image_update*
  Whether you want to get a `yum update` on the overcloud-full prior its
  upload into Glance. (Bool, default to yes)

*undercloud_low_memory*
  Creates a hieradata file to configure 1 worker per OpenStack
  service. This hieradata file is consummed by the Undercloud.
  (Bool, default no)
