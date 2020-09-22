# Elementary Base Box

Goal of this project is to create an elementary os base box for vagrant with [packer](https://www.packer.io).
The creation of the box fails in 60% of the builds which looks like timing issues because it succeeds in the other 40% without changing anything.

Because of missing resources the development is currently only done for the windows packer file `elementaryos-base-windows.json`

> :bulb: Example json files for packer are available here: <https://github.com/taliesins/packer-baseboxes>

## Build the ElementaryOS basebox for virtualbox

We start by creating the image for virtualbox. We do this because it is easier to test the created image that way. Once we decide that the image is good we will create a base box instead of a virtualbox image.

VirtualBox images can be created from several sources:

* An .iso file
* An OVF/OVA file
* An available virtual box vm

However as we do not have an already available ElementaryOS VM or an ovf/ova file we will use an iso image to create the basebox.

1. Open the file `elementaryos-base-windows.json` in your prefered editor
1. Locate the entry `iso_checksum` and provide the checksum for the downloaded iso file. It can be found in the elementary docs at the install section: <https://elementary.io/en/docs/installation#verify-your-download>
1. Open a terminal and navigate to the folder with the `elementaryos-base-windows.json` file
1. Make sure the packer file is valid: `packer validate -var "elementaryos_iso_path=F:\\setups\\isos\\elementaryos-5.1-stable.20200706.iso" elementaryos-base-windows.json` and hit enter  
![Template validated](images/template_validated.png)
1. Build the ElementaryOS basebox: `packer build -var "elementaryos_iso_path=F:\\setups\\isos\\elementaryos-5.1-stable.20200706.iso" -force elementaryos-base-windows.json`  
![Image creation in progress](images/packer_creating_vm.png)
1. Packer will now start the vm and install the os  
![OS Installation in progress](images/packer_vm_os_installing.png)

### Test the basebox

1. Remove a maybe already existing basebox: `vagrant box remove ElementaryOS`
1. Add the created box to vagrant with: `vagrant box add ElementaryOS .\ElementaryOS.box`  
![Adding the box to vagrant](images/vagrant_add_box.png)
1. Create a new vagrant project with the ElementaryOS box as base: `vagrant init -m ElementaryOS`
1. Open the vagrantfile
1. Add the following:

    ```ruby
    config.vm.provider "virtualbox" do |vb|
        # Display the VirtualBox GUI when booting the machine
        vb.gui = true
    end
    ```

1. Start the machine: `vagrant up`
1. You should have a running ElementaryOS
![BaseBox Running](images/basebox_running.png)
1. With vagrant ssh access enabled
![Vagrant ssh access](images/basebox_ssh_access.png)

References:

* <https://stackoverflow.com/questions/22065698/how-to-add-a-downloaded-box-file-to-vagrant>

## Explanations

### SSH Configuration

As stated by the packer documentation the initial ssh setup needs to be done by the boot command.

> If you are building from a brand-new and unconfigured operating system image, you will almost always have to perform some extra work to configure SSH on the guest machine. For most operating system distributions, this work will be performed by a boot command that references a file which provides answers to the normally-interactive questions you get asked when installing an operating system.

Unluckily for us ElementaryOS doesn't support a preseed file. Because of this we need to wait until the installation has been done and then we simulate the keys that are needed to login, open a terminal and install the ssh server. This is done the following way:

1. Install the openssh-server and wget which is needed to download the current vagrant key: `sudo apt install wget openssh-server -y`
1. Create the ssh directory for the vagrant user: `mkdir ~/.ssh`
1. Set the correct access rights for the ssh directory: `sudo chmod 700 ~/.ssh`
1. Chante to the ssh directory: `cd ~/.ssh`
1. Add the current vagrant key to the authorized_keys: `wget --no-check-certificate 'https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub' -O authorized_keys`
1. Add the public key of the ephemeral ssh keypair that packer created automatically: `echo {{.SSHPublicKey}} >> ~/.ssh/authorized_keys`
1. Set the correct access rights for the authorized_keys file: `sudo chmod 600 ~/.ssh/authorized_keys`
1. Make sure that the ssh file belongs to the vagrant user: `chown -R vagrant ~/.ssh`
1. Restart the ssh daemon to activate the changes: `sudo systemctl restart sshd`

You can see that all these steps are done in the boot_command:

```
"<wait15s>sudo apt install wget openssh-server -y<enter>",
"<wait2m>mkdir ~/.ssh<enter>",
"<wait2s>sudo chmod 700 ~/.ssh<enter>",
"<wait2s>cd ~/.ssh<enter>",
"<wait2s>wget --no-check-certificate 'https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub' -O authorized_keys<enter>",
"<wait3s>echo {{.SSHPublicKey}} >> ~/.ssh/authorized_keys<enter>",
"<wait3s>sudo chmod 600 ~/.ssh/authorized_keys<enter>",
"<wait3s>chown -R vagrant ~/.ssh<enter>",
"<wait3s>sudo systemctl restart sshd<enter><wait3s>"
```

References:

* <https://www.packer.io/docs/communicators/ssh>
* <https://www.packer.io/guides/automatic-operating-system-installs>
* <https://github.com/hashicorp/vagrant/tree/master/keys>
* <https://superuser.com/questions/287651/can-i-have-multiple-ssh-keys-in-my-ssh-folder>

### Guest Additions

Install necessary libraries for guest additions and Vagrant NFS Share: `sudo apt-get -y -q install linux-headers-$(uname -r) build-essential dkms nfs-common`

References:

* <https://www.packer.io/docs/provisioners/shell>
* <https://www.vagrantup.com/docs/providers/virtualbox/boxes>

<!--
## Jenkins Buildjob

1. Clone the project: git clone ssh://git@git.zit.local:2222/~dwolff/elementaryos-basebox.git
1. Run packer from container and bind mount the cloned directory:
    ```bash
    docker run -it \
        --mount type=bind,source=/absolute/path/to/test_docker_packer,target=/mnt/test_docker_packer \
        hashicorp/packer:latest build \
        /mnt/test_docker_packer/elementaryos-base-linux.json
    ```

References:

* https://hub.docker.com/r/hashicorp/packer
-->