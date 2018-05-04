# TrueOS OpenBSD Desktop: Build Base for Lightweight Development Environments

For general information about this repo and related ones, please see [Provision Lightweight Development Environments](http://github.com/neopragma/provision-lightweight-development-environments).

## From -> To

**From:** No operating system installed.

**To:** FreeBSD configured to receive provisioning to customize it for software development.

## 1. Install TrueOS Desktop edition

### 1.1. Download iso

Download "TrueOS Desktop (DVD Image)" iso from <a href="https://web.trueos.org/downloads/">https://web.trueos.org/downloads/</a>.

### 1.2. Install TrueOS Desktop.

Install TrueOS Desktop as a VM (any [hypervisor](hypervisor.md)). Define it as a FreeBSD OS, give it 1 GB memory and 20 GB storage.

Use the graphical installer. Choose the desktop option. (The server option, as well as the TrueOS Server iso, had serious problems when I tried them 03 May 2018.)

Set the language etc. as you please. Accept the default partition mapping.

When it reboots, it will ask for some additional setup information.

Set the timezone.

Set the root password: ```root```.

User: ```Developer```.
Username: ```dev```.
Password: ```developer```.

Optional software:

- SSH

Click "Finish" to log in.

Log in as user 'Developer' with password 'developer'. You will use ```sudo``` for privileged operations.

Choose the Lumina desktop option; Fluxbox is installed but is not configured, and out-of-the-box there is no way to start a terminal emulator from Fluxbox. 

Update the local package repository:

```shell
sudo pkg update -f
```

Git is already installed on this distro. Double-check that it is present:

```shell
git --version
```

If it is not present, install it:

```shell
sudo pkg install git
```

## 2. Configure the instance as a development template

### 2.1. Clone the repository.

Clone the repository for building a template instance from TrueOS Desktop:

```shell 
cd 
git clone git://github.com/neopragma/bootstrap-trueos-freebsd-dev-base
``` 

### 2.2. (Optional) Review default configuration and modify as desired.

If you want your template to be configured differently than the default, make the necessary changes to bash scripts, Chef recipes, and configuration files. 

In particular, look at:

The directory structure of the provisioning repository looks like this:

```
bootstrap-trueos-freebsd-dev-base/
    bootstrap      C Shell script to provision the instance.

    scripts/       ```bootstrap``` copies these files to /usr/home/dev/
        .cshrc     C Shell startup script
        .login     C Shell login script

    neovim/        ```bootstrap``` copies everything here
                   => /usr/home/dev/.config/nvim/

    openbox/       ```bootstrap``` copies everything here
                   => /usr/home/dev/.config/openbox/
```

### 2.4. Run the bootstrap script.

If all goes well, this will provision the instance as a base or template for building development environments. Check the results carefully in case of errors. There are many steps and anything can happen despite care in preparing the script. 

```shell 
cd /root/bootstrap-trueos-freebsd-dev-base
./bootstrap 
``` 

### 3. Manual configuration of NeoVim.

Some steps can't be scripted. 

#### 3.1. Enable plugins 

One-time run of :UpdateRemotePlugins for certain plugins.

- Start neovim 
- Run the editor command :UpdateRemotePlugins
- Quit neovim

### 4. Known issues with the bootstrap process

None.

### 5. Known issues after system comes up

None.

## 6. Usage Notes

### 6.1. Openbox

On the login form, choose Openbox.

It will start a terminal emulator automatically. 

To exit from Openbox and log out, you can type the command ```lo``` in a terminal emulator. Otherwise, you can type ```openbox --exit```, or you can right-click outside of a window and choose ```Exit``` from the Openbox menu.

### 6.2. NeoVim

See the notes on the [introductory README](http://github.com/neopragma/provision-lightweight-development-environments) for information about NeoVim key bindings and plugins.

### 6.3. VMware and Xorg config for automatic display resolution

If you are running a FreeBSD guest under VMware, consult this guide to configure automatic detection of display resolution: [http://www.unibia.com/unibianet/freebsd/open-vm-tools-freebsd-10-xorg](http://www.unibia.com/unibianet/breebsd/open-vm-tools-freebsd-10-xorg). 
