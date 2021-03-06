#  rhc-ose ansible Automation

Automation of OpenShift 3 using [Ansible](http://www.ansible.com/)

## Provisioning Quickstart

### Local Setup (one time, only)

In addition to _cloning this repo_, you'll need the following:

* Access to an OpenStack environment using an [OpenStack RC File](http://docs.openstack.org/user-guide/common/cli-set-environment-variables-using-openstack-rc.html)
  * File should be placed at `~/.config/openstack/openrc.sh`
>**NOTE**: OpenStack environment currently requires HEAT to be enabled, and the user must have the `heat_stack_owner` role assigned.
* A [Key-pair created in OpenStack](https://github.com/naturalis/openstack-docs/wiki/Howto:-Creating-and-using-OpenStack-SSH-keypairs-on-Linux-and-OSX)
* Docker installed (`yum install -y docker` on RHEL/Centos, `dnf install docker -y` on Fedora)
  * If you plan to run docker as yourself (non-root), your username must be added to the `docker` user group.
* An `~/.ansible.cfg` file containing the following:
```
[defaults]
roles_path=/root/repository/casl-ansible/roles:/root/repository/openshift-ansible/roles
filter_plugins= /usr/share/ansible_plugins/filter_plugins:/root/repository/openshift-ansible/filter_plugins
host_key_checking = False
```
* Clone this, and dependent `openshift-ansible` repositories into the same directory
```
cd ~/src/
git clone git@github.com:redhat-cop/casl-ansible.git
```
* Copy `casl-ansible/inventory/sample.casl.example.com.d/inventory/clouds.yaml` to `~/.config/openstack/clouds.yaml`

* Download/untar `openshift-ansible` for use as part of the install. Make sure to do so within the same directory as above (i.e.: `~/src`), and either rename the directory to `openshift-ansible` or create symlink to it (see example below). See table below for versions / urls to be used for the download. Note that other versions may work as well, but these are the ones we have tested and found to be stable. Feel free to submit PRs with updated versions as they are found to be functional. 

(*hint* right-click the `openshift-ansible` version number in the table below and copy the URL)

```
wget <url> -O - | tar -xz
ln -fs openshift-ansible-*<version>* openshift-ansible
```

| openshift-ansible url     | OpenShift version | 
|:-------------------------:|:-----------------:|
| [3.4.50-1](https://github.com/openshift/openshift-ansible/archive/openshift-ansible-3.4.60-1.tar.gz) | OCP 3.4 |

Cool! Now you're ready to provision OpenShift clusters

### Provision a Cluster

To start, we'll provision the `sample.casl.example.com` cluster defined in the `casl-ansible/inventory` directory. 

**Note**: *It is recommended that you use a different inventory by duplicating the `sample.casl.example.com` directory and keep it elsewhere. In that way, you can update/remove/change your casl-ansble directory without losing your inventory. It may take some time to get the inventory just right, hence it is very beneficial to keep it around for future use without having to redo everything.*

The following is just an example on how the `sample.casl.example.com` inventory can be used:

1) Edit `casl-ansible/inventory/sample.casl.example.com/inventory/hosts` and edit the `# OpenStack Provisioning Variables` and `# Subscription Management` sections at the top to match your environment/project/tenant. See comments in the file for more detailed information on how to fill these in.

2) Start the `openstack-client` container.
```
./casl-ansible/docker/openstack-client-centos/run.sh --repository=/path/to/repos/dir/
```

3) Run the `end-to-end` provisioning playbook
```
ansible-playbook -i /root/repository/casl-ansible/inventory/sample.casl.example.com.d/inventory /root/repository/casl-ansible/playbooks/openshift/end-to-end.yml
```

## Roles

The following are a list of Absible roles available

* cicd - Installs and configures CICD tools such as Jenkins and Nexus
* cicd-common - Sets common CICD related facts
* common - Provides for the generation of an environment id per execution and sets common facts
* openshift-common - Sets common OpenShift related facts
* openshift-provision - Installs and configures OpenShift on a set of masters and nodes (Work in Progress)
* openstack-create - Creates an OpenStack instances and attaches block storage

## Playbooks

The following are a list of Ansible playbooks

* OpenShift Provision (ose-provision.yml)
    * Provision machines on OpenStack for OpenShift
	    * Attach persistent storage
	    * Update machines with latest packages
        * Additional work in progress
* Continuous Integration/Continuous Delivery (cicd-provision.yml)
	* Provision instance on OpenStack
		* Attach persistent storage
	* Install prerequisite packages
	* Install and configure Java, Groovy, Maven, Jenkins, Nexus, Docker
