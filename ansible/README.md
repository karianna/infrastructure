# Ansible playbooks to download and install dependencies for OpenJDK on various platforms

# Quickstart Guide

To test the ansible scripts, you'll need to install the following programs.

For macOS:

1. Install Homebrew 2.1.7 or later
  ```bash
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```
2. Install Vagrant 2.2.5 or later
  ```bash
  brew cask install vagrant
  ```
3. Install Virtualbox 6.0.8 or later:
  ```bash
  brew cask install virtualbox
  ```
 
# Running via Vagrant and VirtualBox

To test the ansible scripts you can set up a Virtual Machine isolated from your own local system.
A `Vagrantfile` has been provided and the usual `vagrant` commands should get it up and running.

The following method runs the ansible playbooks on the local connection.
Normally you will be running ansible on your development machine, and using it
to modify remote hosts.

**NOTE** The `/vagrant/` directory maps to the directory on your host that you launched the `VagrantFile` from
e.g. `~/workspace/AdoptOpenJDK/openjdk-infrastructure/ansible`

```bash
vagrant up

vagrant ssh # Uses default ssh login, user=vagrant, password=vagrant

cd /vagrant/playbooks
```

Note:
 - A `hosts` file containing `localhost ansible_connection=local` will already be present in the directory with the playbook scripts (`/vagrant/playbooks`).
 - A public key file `id_rsa.pub` will already be present in the `/home/vagrant/.ssh/` folder

1) Run a playbook to install dependencies, for Ubuntu 14.x on x86:

`ansible-playbook -s AdoptOpenJDK_Unix_Playbook/main.yml`

or

`ansible-playbook -i hosts -s AdoptOpenJDK_Unix_Playbook/main.yml`

In case one or more tasks fail or should not be run in the local environment, see [Skipping one or more tags via CLI when running Ansible playbooks](https://github.com/AdoptOpenJDK/openjdk-infrastructure/tree/master/ansible#skipping-one-or-more-tags-via-cli-when-running-ansible-playbooks) for further details. Ideally, the below can be run for smooth execution in the `vagrant` box:

```bash
ansible-playbook -i hosts -s AdoptOpenJDK_Unix_Playbook/main.yml --skip-tags="install_zulu,jenkins_authorized_key,nagios_add_key,add_zeus_user_key"
```

# Running Manually

## Do I need to be a superuser to run the playbooks?

Yes, in order to access the package repositories (we will perform either `yum install` or `apt-get` commands)

## How do I run the playbooks?

1) Install Ansible 2.4 or later (

    - On RHEL 7.x
    ```bash
    yum install epel-release
    yum install ansible
    ```

    - For Ubuntu
    ```bash
    sudo apt-add-repository ppa:ansible/ansible
    sudo apt update
    sudo apt install ansible
    ```

    - On another system with `pip` available:
    ```bash
    sudo pip install ansible
    ```

2) Ensure that you have edited the `hosts` in `/etc/ansible/` or in the project root directory. For running locally `hosts` file should contain something as simple as `localhost ansible_connection=local`.

3) Run a playbook to install dependencies, e.g. for Ubuntu 14.x on x86:
    ```bash
    ansible-playbook -s playbooks/AdoptOpenJDK_Unix_Playbook/main.yml

    # Or to use a custom hosts file:
    ansible-playbook -i /path/to/hosts -s AdoptOpenJDK_Unix_Playbook/main.yml
    ```

4) The Ansible playbook will download and install any dependencies needed to build OpenJDK

## How do I run the playbook on a Windows Machine?

As Ansible can't run on Windows, it has to be run on a seperate system and then pointed at a Windows machine (such as a VM). To test the playbook, a vagrant VM can be used.
You can do this by following these steps:

1) Run `vagrant plugin install vagrant-disksize`. This will install a plugin that allows the user to specify the disksize of the VM.
2) `git clone "https://github.com/AdoptOpenJdk/openjdk-infrastructure/"`
3) `ln -sf Vagrantfile.WindowsS12 Vagrantfile && vagrant up` in the cloned `/openjdk-infrastructure/ansible` directory
4) Note the IP address that is output from running `vagrant up` and place it into a text file. e.g. `hosts.win`. Place this text file in `../anisble/playbooks/AdoptopenJDK_Windows_Playbook`
5) Edit `../AdoptOpenJDK_Windows_Playbook/main.yml` so the `- hosts :{{ groups['Vendor_groups'] ... etc` becomes `- hosts: all`
6) Add into `../AdoptOpenJDK_Windows_Playbook/group_vars/all/adoptopenjdk_variables.yml` the line `ansible_winrm_transport: credssp`. You'll also need to uncomment and change `ansible_password: CHANGE_ME`.
7) From the `../ansible` directory, running `sudo ansible-playbook -i playbooks/AdoptOpenJDK_Windows_Playbook/hosts.win -u vagrant playbooks/AdoptOpenJDK_Windows_Playbook/main.yml` will start the playbook.

Note: if using a macOS Mojave, `export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=yes` will be required before starting the playbook.
 
## Can I have multiple VMs on different OSs?

As vagrant uses Virtualbox to create VMs, multiple VMs on different OSs can be setup.
You can do this by following these steps:

  1. Make a copy of the existing directory you have.
  2. The "vagrantfile" is an alias to the vagrantfile that is labelled with the desired OS. Replace this so it is an alias of the OS you want. (default OS is CentOS)
  3. Continue the vagrant functions as normal.

To access each vagrant VM, you'll need to be in the correct directory to `vagrant ssh` into. 

## Which playbook do I run?

Our playbooks are named according to the operating system they are supported for, keep in mind that package availability may differ between operating system releases

## Where can I run the playbooks?

On any machine you have SSH access to: in the playbooks here we are using `hosts: local`,
our playbook will run on the hosts defined in the Ansible install directory's `hosts` file. To run on the local machine,
we will have the following text in our `/etc/ansible/hosts` file:
```
[local]
127.0.0.1
```
Running `ansible --version` will display your Ansible configuration folder that contains the `hosts` file you can modify

## Skipping one or more tags via CLI when running Ansible playbooks

One of the two examples below is appropriate to run a playbook and skip tags, leading to linked and dependent tasks to be not executed:

```bash
ansible-playbook -s playbooks/AdoptOpenJDK_Unix_Playbook/main.yml --skip-tags "jenkins_user"

ansible-playbook -s playbooks/AdoptOpenJDK_Unix_Playbook/main.yml --skip-tags "install_zulu, jenkins_authorized_key, nagios_add_key, add_zeus_user_key"
```

## Skipping tasks via CLI when running Ansible playbooks (conditional and skipping tasks)

The below example is appropriate to run playbook by skipping tasks by using a combination of conditionals and tags (linked and dependent tasks will not be executed):

```bash
ansible-playbook -i [/path/to/hosts] -s AdoptOpenJDK_Unix_Playbook/main.yml --extra-vars "Jenkins_Username=jenkins Jenkins_User_SSHKey=[/path/to/id_rsa.pub] Nagios_Plugins=Disabled Slack_Notification=Disabled Superuser_Account=Disabled" --skip-tags="install_zulu"
```

Note that when running from inside the `vagrant` instance:
 - the `[/path/to/hosts]` can be replace with `/vagrant/playbooks/hosts`
 - the `[/path/to/id_rsa.pub]` can be replaced with `/home/vagrant/.ssh/id_rsa.pub`

Useful if one or more tasks are failing to execute successfully or if they need to be skipped due to not deemed to be executed in the right environment.

## Verbose mode, debugging Ansible playbooks

Below are the levels of verbosity available with using ansible scripts:

```bash
ansible-playbook -v -s playbooks/AdoptOpenJDK_Unix_Playbook/main.yml
ansible-playbook -vv -s playbooks/AdoptOpenJDK_Unix_Playbook/main.yml
ansible-playbook -vvv -s playbooks/AdoptOpenJDK_Unix_Playbook/main.yml
ansible-playbook -vvvv -s playbooks/AdoptOpenJDK_Unix_Playbook/main.yml
```

A snippet from the man pages of Ansible:

>     -v, --verbose
>           verbose mode (-vvv for more, -vvvv to enable connection debugging)

## Expected output of a successful Ansible build

When the above `ansible-playbook` commands succeed, we should get something of this output (snippet):

```
TASK [Start NTP] ***************************************************************
 changed: [localhost]

TASK [Remove unneeded packages from the cache] *********************************
 ok: [localhost]

TASK [Remove apt dependencies that are no longer required] *********************
 ok: [localhost]

TASK [Send Slack notification, successful] *************************************
 skipping: [localhost]

PLAY RECAP *********************************************************************
 localhost                  : ok=49   changed=21   unreachable=0    failed=0
```
