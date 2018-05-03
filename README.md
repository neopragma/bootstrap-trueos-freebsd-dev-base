# Debian 9: Build Base for Lightweight Development Environments

For general information about this repo and related ones, please see [Provision Lightweight Development Environments](http://github.com/neopragma/provision-lightweight-development-environments).

## From -> To

**From:** No operating system installed.

**To:** Debian 9 configured to receive provisioning to customize it for software development.

## 1. Install Debian 9

### 1.1. Download iso

Download iso from <a href="http://www.debian.org/distrib/netinst">http://www.debian.org/distrib/netinst</a>.

### 1.2. Install Debian 9.

Install Debian 9 as a VM (any [hypervisor](hypervisor.md)). 

When prompted, create user 'dev' with password 'developer'. Set the root password to 'admin'.

On the "Software selections" form, ensure the only selection is "standard system utilities." 

### 1.3. Add user 'dev' to sudoers

Debian distros are configured to disallow direct login as 'root'. The minimal installation we are doing doesn't install ```sudo``` at all. Add it now. 

Log in as 'dev' and switch to 'root':

```shell 
su - 
[admin]
``` 

Then install ```sudo```. 

```shell 
apt install sudo 
``` 

Now use ```visudo``` to grant sudo rights to user 'dev'.

```shell 
visudo 
``` 

Find the line that looks like this:

```
root     ALL=(ALL:ALL) ALL
```

Add a line just like it for user 'dev' so that it looks like this:

```
root     ALL=(ALL:ALL) ALL
dev      ALL=(ALL:ALL) ALL
```

Save the file and return to user 'dev'. See if you can do a simple command that requires sudo, like this:

```shell 
sudo ls -la /usr/bin 
``` 

If that looks okay, proceed.

## 2. Provision the instance as a "base" for development environments.

This will install enough software on the instance to enable it to be used as a template for building software development environments tailored to different programming languages and development/testing tools. 

### 2.1. Install git.

The provisioning scripts are on Github. The instance needs git support to clone the repository and complete the configuration. 

```shell 
sudo apt install -y git 
``` 

### 2.2. Clone the repository.

Become root.

```shell 
su - 
[admin]
```

Clone the repository for building a template instance from Debian 9:

```shell 
cd 
git clone git://github.com/neopragma/bootstrap-debian-9-dev-base
``` 

### 2.3. (Optional) Review default configuration and modify as desired.

If you want your template to be configured differently than the default, make the necessary changes to bash scripts, Chef recipes, and configuration files. 

In particular, look at:

The directory structure of the provisioning repository looks like this:

```
bootstrap-debian-9-dev-base/
    bootstrap              Bash script to prepare the instance to run Chef
                           and kick off the Chef cookbook that completes
                           the provisioning.

    scripts/               ```bootstrap``` copies these files to /usr/local/bin.
        cli                Escape from OpenBox to command line from terminal
        provision          Run the Chef cookbook to provision the instance
        cook               Run one Chef cookbook or cookbook::recipe
        recipes            List the available Chef recipes for provisioning
        runchefspec        Run `bundle exec rake` to run rspec on Chef recipes

    debian_prep/           ```bootstrap``` copies these files to prepare Chef
        Gemfile            => /root/chef-repo/cookbooks/debian_prep/
        Rakefile           => /root/chef-repo/cookbooks/debian_prep/
        recipes/           => /root/chef-repo/cookbooks/debian_prep/
        spec/
            spec_helper.rb => /root/chef-repo/cookbooks/debian_prep/spec
            unit/recipes/  => /root/chef-repo/cookbooks/debian_prep/spec/unit/recipes

    neovim/                Chef recipe ```install_neovim``` performs these copies.
                           => /root/.config/nvim/
                           => /dev/.config/nvim/

    openbox/               Chef recipe ```install_x``` performs these copies.
                           => /dev/.config/openbox/
```

### 2.4. Run the bootstrap script.

If all goes well, this will provision the instance as a base or template for building development environments. Check the results carefully in case of errors. There are many steps and anything can happen despite care in preparing the script. 

```shell 
cd /root/bootstrap-debian-9-dev-base
./bootstrap 
``` 

### 3. Manual configuration of NeoVim.

Some steps can't be scripted. 

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
