# README #

A [Docker](https://www.docker.com/use-cases) Ubuntu server (Ubuntu 20.04 LTS 64bit) ~~hosted on an instance of Oracle Cloud Free Tier~~.

~~Cloud instance creation, tear down and~~ Server provisioning is done using [Ansible](https://opensource.com/resources/what-ansible). The server is provisioned with some server tools and docker. You can provision any number of machines that you have access to via ssh. Just edit the `host` file and add a new `yml` file for each server in the `host_vars` folder with the appropriate values. This installation is compatible with **`arm`** or **`x86`** machines.

An optional [Portainer Agent](https://shownotes.opensourceisawesome.com/installing-portainer/) and [Tailscale](https://tailscale.com/) are provided as a docker GUI accessed through a secure tailscale vpn tunnel.

I've also added a docker `/etc/docker/daemon.json` file to enable log rotation, this can also be over ridden on a container basis.

The only skill you need to implement this is the ability to edit [yml](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html) files however knowledge of server configuration and management is required to properly maintain a running server.

**EDIT:** I've decided not to automatically create an Oracle Cloud Free Tier machine. I might in the future, but I have no need as the web interface is sufficient for my current needs.

* [Setup an Oracle Cloud Free Tier account](https://www.oracle.com/cloud/free/).

## Preparing the Control Node ###

The `Control Node` is the machine you will be using to manage and configure your server(s). I've included installation instructions to take advantage of a secure private vpn tunnel to manage your servers.

 You'll need a Tailscale account for this.

* [Setup a Tailscale account](https://tailscale.com/start). (Optional)
* Configure Ansible for [Tailscale](https://tailscale.com/) by pasting this in your terminal
  * ```sudo ansible-galaxy install artis3n.tailscale```
* You must supply a `tailscale_auth_key variable`, which can be generated under your Tailscale account [here](https://login.tailscale.com/admin/authkeys).

The machine will also need the following software.

* [SSH keys](https://jumpcloud.com/blog/what-are-ssh-keys) for this installation.
* [Install Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html) for your Mac or Linux machine.
* [Windows WSL](https://docs.ansible.com/ansible/latest/user_guide/windows_faq.html#can-ansible-run-on-windows) for your Windows machine with WSL installed.
* Configure Ansible for this installation by pasting this in your terminal
  * ```sudo ansible-galaxy install geerlingguy.pip```
  * ```sudo ansible-galaxy install geerlingguy.docker```
  * ```sudo ansible-galaxy install geerlingguy.docker_arm```
  <!-- * ```sudo ansible-galaxy collection install oracle.oci``` -->

## Optional for the Control Node ###

The following are optional for the `Control Node`. They provide additional convinces for the `Control Node`.

* [Docker](https://www.docker.com/use-cases) is an open-source project for automating the deployment of applications as portable, self-sufficient containers that can run on the cloud or on-premises.
* [Portainer](https://codeopolis.com/posts/beginners-guide-to-portainer/)  is a simple and lightweight, but powerful application that is used to provide a web management interface that you can use to perform functions on your Docker host.
* [TMUX](https://www.nuno-sarmento.com/how-to-install-and-use-tmux-on-ubuntu/) allows users to switch between multiple programs in one terminal. Users can detach and reattach the multiple programs to different terminal sections. TMUX sessions are persistent. The running programs continue to run even if you are disconnected.
* [Mosh](https://linuxhandbook.com/mosh/) stands for MObile SHell and is a replacement of SSH for remote connections to Unix/Linux systems that brings a few noticeable advantages over well known SSH connections. In brief, it’s faster and more responsive, especially on long delay and/or unreliable links. So we can say that Mosh is an alternative for SSH.
* [SSHFS](https://phoenixnap.com/kb/sshfs) (SSH File System) is a client for mounting a file system located on a remote machine onto your local system through an SSH connection. Using the SFTP (SSH file transfer protocol), the SSHFS command-line tool mounts a physical or virtual disk locally, allowing file transfer between a local and remote machine.

## Finish the Control Node ##

Edit the ```group_vars/all/server.yml``` file to customize this installation. The default values work fine for a Virtual Box installation. (See below). You'll need to edit the file for a live server. There are some advanced configuration settings not shown below.

```yaml
hostname: "docker-vm" # your hostname
tz: "America/New_York" # the timezone of your server

# This is the name of the user that all of the containers will run as.
# This user has no shell and cannot login to the server.
portmaster: pm # I like short names for typing ;)

# These are only needed if you plan on building containers from a private git repo

# The path on the control node where the ssh keys are stored.
# The default value is standard on mac and linux
portmaster_key_path: "~/.ssh/"
# The public ssh key associated with a github account
# id_rsa.pub seems to be the most common name.
portmaster_pub_key: "id_rsa.pub"

# The developer user is a privileged user with shell and sudo access.
developer: "dv" # I like short names for typing ;)
# The path on the control node where the ssh keys are stored.
# The default value is standard on mac and linux
developer_key_path: "~/.ssh/"
# The public ssh key associated with a github account
# id_rsa.pub seems to be the most common name.
developer_pub_key: "id_rsa.pub"

tailscale_auth_key: "" # If left empty tailscale won't install.
```

---
**Edit these two files before running the playbook to provision a live machine.**

You'll also need to edit the `/hosts` file:

```ini
[servers]
oracle
```

Now create or edit a file in `/host_vars` with the same name as your servers in the above section. Here's what the default `/host_vars/oracle.yml` file looks like;

```yaml
# For machines without a domain name
# you can use it's public IP address
ansible_host: newmachine.example.com
# This is the default user on an ubuntu install.
ansible_ssh_user: ubuntu
# The path on the control node where the
# ssh private key is stored.
# The default value is common
ansible_ssh_private_key_file: "~/.ssh/id_rsa"
```

---

## Packages we will be installing ##

The following software is going to be installed on the server based on the values set in the `/roles/update/tasks/main.yml` with the exception of docker which is set in the `/playbook.yml` file.

* [Docker](https://www.docker.com/use-cases) is an open-source project for automating the deployment of applications as portable, self-sufficient containers that can run on the cloud or on-premises.
* [unzip](https://phoenixnap.com/kb/how-to-unzip-file-linux-ubuntu) If you downloaded a file that ends in .tar.gz or .zip, this indicates that it has been compressed.
* [curl](https://linuxize.com/post/curl-command-examples/) is a command-line utility for transferring data from or to a server designed to work without user interaction.
* [Git](https://www.freecodecamp.org/news/learn-the-basics-of-git-in-under-10-minutes-da548267cc91/) is a version-control system for tracking changes in computer files and coordinating work on those files among multiple people.
* [tree](https://linuxhint.com/tree-command-in-ubuntu/) is a command that shows the directory output in a tree-like hierarchical structure.
* [Htop](https://vitux.com/how-to-use-htop-to-monitor-system-processes-in-ubuntu-20-04/) is the improved version of the top command-line utility. Using the htop utility, the user can view the crucial details about the Ubuntu system such as CPU running processes, memory utilization, load average, PID’s, etc.
* [csvkit](https://csvkit.readthedocs.io/en/latest/) is a suite of command-line tools for converting to and working with CSV, the king of tabular file formats.
* [apache2-utils](https://packages.debian.org/stretch/apache2-utils) Provides some add-on programs useful for any web server
* [TMUX](https://www.nuno-sarmento.com/how-to-install-and-use-tmux-on-ubuntu/) allows users to switch between multiple programs in one terminal. Users can detach and reattach the multiple programs to different terminal sections. TMUX sessions are persistent. The running programs continue to run even if you are disconnected.
* [Mosh](https://linuxhandbook.com/mosh/) stands for MObile SHell and is a replacement of SSH for remote connections to Unix/Linux systems that brings a few noticeable advantages over well known SSH connections. In brief, it’s faster and more responsive, especially on long delay and/or unreliable links. So we can say that Mosh is an alternative for SSH.
* [Ranger](https://vitux.com/how-to-install-ranger-terminal-file-manager-on-linux/) is a lightweight and powerful file manager that works in a terminal window. It offers a smooth way to move into directories, view files and content, or open an editor to make changes to files.
* [Ncdu](https://computingforgeeks.com/ncdu-analyze-disk-usage-in-linux-with-ncdu/) was designed to ease finding space hogs on a remote server where you don’t have an entire graphical setup available
* [iperf3](https://www.tecmint.com/test-network-throughput-in-linux/) is a free open source, cross-platform command-line based program for performing real-time network throughput measurements.
  * libiperf0
  * libsctp1

## Developer Tools ##

The following shell enhancements and developer tools are going to be installed on the server based on the values set in the `/roles/common/tasks/developer.yml` file.

* [ZSH](https://www.tecmint.com/install-zsh-in-ubuntu/) stands for Z Shell which is a shell program for Unix-like operating systems. ZSH is an extended version of Bourne Shell which incorporates some features of BASH, KSH, TSH.
* [powerline](https://ubunlog.com/en/powerline-customize-command-line/) provides status lines and prompts and offers useful information on the terminal prompt.
  * fonts-powerline
* [Powerlevel10k](https://dev.to/abdfnx/oh-my-zsh-powerlevel10k-cool-terminal-1no0) is a theme for Zsh. It emphasizes speed, flexibility and out-of-the-box experience.
* [zsh-syntax-highlighting](https://linuxhint.com/enable-syntax-highlighting-zsh/) automatically highlights your commands as you type them, which can help you catch syntax errors and fix them before running the command.
* [Hollywood](https://ostechnix.com/turn-ubuntu-terminal-hollywood-technical-melodrama-hacker-interface/) will turn your Ubuntu Linux console into a veritable Hollywood technical melodrama hacker interface.
* [Ansible](https://techexpert.tips/ansible/ansible-playbook-examples-ubuntu-linux/) is a software tool that provides simple but powerful automation for cross-platform computer support. It is primarily intended for IT professionals, who use it for application deployment, updates on workstations and servers, cloud provisioning, configuration management, intra-service orchestration, and nearly anything a systems administrator does on a weekly or daily basis.

We will also be installing a set of of `.dot` files and fonts for the developer.

* .zshrc
* .aliases
* .gitconfig
* .tmux.conf
* .p10k.zsh
* Fira Code Nerd Mono
* Fira Code Nerd Regular

## Containers we will be installing ##

The only container we will be installing with Ansible is the [Portainer Agent](https://docs.portainer.io/v/be-2.10/advanced/edge-agent), which connects to your `Control Node` [Portainer](https://codeopolis.com/posts/beginners-guide-to-portainer/) service.

View the README.md file about how we deploy containers using our secure vpn tunnel and our `Control Node` portainer service.

## Samples of containers ##

I've provided a few examples of containers with instructions on how to install them simply with the Portainer GUI.

* [Nginx Proxy Manager](./stacks/network/README.md)
* [Administration Stack](./stacks/admin/README.md)
* [Wordpress Stack](./stacks/wordpress/README.md)

## A note about docker logs ##

The default log driver for docker saves container logs as JSON in the docker logs directory at `/var/lib/docker/containers/<container id>/<container id>-json.log`. By default the logs don't rotate and can build up on your host. The docker config file included in this installation at `/etc/docker/daemon.json` sets the log files to rotate by setting the docker log options to a maximum size of 10 MB and maximum number of files to 10. See here for more [information](https://docs.docker.com/config/containers/logging/configure/) and options.

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "10"
  }
}
```

[CSVkit](https://www.techrepublic.com/article/how-to-convert-xls-and-json-files-to-csv-in-linux-with-csvkit/) is pre-installed on the server as a tool for converting and manipulating the docker logs. It can be used to convert docker logs from JSON to CSV format.

## Can I test it first? ##

You can test this server setup with the Vagrantfile included. You'll need to install [VirtualBox](https://en.wikipedia.org/wiki/VirtualBox) and [Vagrant](https://www.vagrantup.com/intro).

* [Install Virtualbox](https://www.virtualbox.org/wiki/Downloads) is a free and open-source hosted hypervisor for x86 virtualization, developed by Oracle Corporation.
* [Install Vagrant](https://www.vagrantup.com/downloads) is a tool for building and managing virtual machine environments in a single workflow. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases production parity, and makes the "works on my machine" excuse a relic of the past.
  * Install vagrant plugin autonetwork by pasting this in your terminal: `vagrant plugin install vagrant-auto_network`

## Quick Start assuming Ansible, Virtualbox and Vagrant is installed ##

The ```group_vars/all/server.yml``` defaults work unedited in the vm. I suggest editing this and experimenting with your setup.

The `/hosts` and `/host_vars` files aren't used by Vagrant.

* Configure Ansible for this installation by pasting this in your terminal
  * ```sudo ansible-galaxy install geerlingguy.pip```
  * ```sudo ansible-galaxy install geerlingguy.docker```
  * ```sudo ansible-galaxy install geerlingguy.docker_arm```
  * ```sudo ansible-galaxy install artis3n.tailscale```
* You must supply a `tailscale_auth_key variable`, which can be generated under your Tailscale account [here](https://login.tailscale.com/admin/authkeys). (Optional)
* Start the virtual machine by pasting this in your terminal
  * `vagrant up`
* Vagrant will start the VM and automatically provision it from Ansible. Learn how they work together [here](https://docs.ansible.com/ansible/2.9/scenario_guides/guide_vagrant.html).
* View the docker web interface from your `Control Node` Portainer GUI or ssh into the virtual machine and poke around.
* Useful vagrant commands
  * `vagrant ssh` ssh into vm as user `vagrant` do `sudo su dv` to become a privileged user.
  * `vagrant destroy -f && vagrant up` useful after making changes.
  * `vagrant provision` This happens automatically on `vagrant up` useful after making changes.
* Things to look at once you become dv user. These commands take advantage of the dv user's aliases.
  * `sudo su dv` become a privileged user.
  * `cfd` change to default docker container config files directory.
  * `cfl` view the docker container config files directory.
  * `ds` docker stats
  * `dr` docker ps -a


## OK Lets Do It ##

* Run the provision command by pasting this in your terminal
* ```ansible-playbook playbook.yml -i hosts```

## Who do I talk to ##

I've created this project mostly for my future self but sharing is caring so your mileage may vary.

* Gary Smith github@kc.gs

## Contributors ##

* Github Co-pilot
* Tabnine
* VS Code
