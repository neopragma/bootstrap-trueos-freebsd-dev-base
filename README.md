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
bootstrap-debian-9-dev-base/
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
cd /root/bootstrap-debian-9-dev-base
./bootstrap 
``` 

### 3. Manual configuration of NeoVim.

Some steps can't be scripted. 

check this

#### 3.1. Install python support for NeoVim plugins.

There are issues on debian distros with pip2. Might require some fiddling.

```shell 
pip2 install --user neovim 
pip3 install --user neovim 
```

### 3.2. Create alias to start X session 

For both the 'dev' and 'root' users, add an alias to the end of .bashrc to start an X session. Source .bashrc to enable the setting. 

```shell 
cd 
echo "alias gui='startx'" >> .bashrc 
. .bashrc
```

#### 3.3. Set NeoVim as the default editor 

The install_neovim Chef recipe installs NeoVim into the alternatives system, but you may have to make it the default selection manually:

```shell 
update-alternatives --config editor 
``` 

Choose the number corresponding to NeoVim and press Enter.

#### 3.4. Enable plugins 

One-time run of :UpdateRemotePlugins for certain plugins.

- Start neovim 
- Run the editor command :UpdateRemotePlugins
- Quit neovim

### 4. Known issues with the bootstrap process

#### 4.1. Please use apt-cdrom

You may get the error "Please use apt-cdrom to make this CD-ROM recognized by APT, apt-get update cannot be used to add new CD-ROMs".

Find out which lines in the sources list refer to CD-ROMs:

```shell 
cat /etc/apt/sources.list | grep cdrom
``` 

Comment those lines out of the sources list and re-run the script.

#### 4.2 update-alternatives reports broken link group

If the bootstrap script partially completes and you re-run it, you might see the following warnings:

```
update-alternatives: warning: forcing reinstallation of alternative /usr/bin/gem2.4 because link group gem is broken

update-alternatives: warning: forcing reinstallation of alternative /usr/bin/ruby2.4 because link group ruby is broken
```

This will not affect the installation and no corrective action is needed.

#### 4.3. dpkg status database is locked by another process

If you interrupt the bootstrap script while an `apt install` is in progress, the dpkg status lock file may not be deleted. When that happens you will get this message on re-running the script. 

Delete the lock file and reconfigure dpkg:

```shell 
rm /var/lib/dpkg/lock 
dpkg --configure -a 
```

### 5. Known issues after system comes up

#### 5.1. Errors from xinit/startx for non-root user

Odd behavior on startx/xinit for non-root user; error messages appear but everything works anyway. Impact is 20 second delay before first terminal window is usable. Observed on debian Server 16.04 builds.
