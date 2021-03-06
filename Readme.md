# vagrant-vbguest

*vagrant-vbguest* is a [Vagrant](http://vagrantup.com) plugin which automatically installs the host's VirtualBox Guest Additions on the guest system.

## Installation

Requires vagrant 0.9.4 or later (including 1.0)    
Since vagrant v1.0.0 the preferred installation method for vagrant is using the provided packages or installers. 
If you installed vagrant that way, you need to use vagrant's gem wrapper:

```bash
$ vagrant gem install vagrant-vbguest
```

If you installed vagrant using RubyGems, use:

```bash
$ gem install vagrant-vbguest
```

Compatibly for vagrant 0.8 is provided by version 0.0.3 (which lacks a bunch of new options)

## Configuration / Usage

If you're lucky, *vagrant-vbguest* does not require any configurations. 
However, here is an example for your `Vagrantfile`:

```ruby
Vagrant::Config.run do |config|
  # we will try to autodetect this path. 
  # However, if we cannot or you have a special one you may pass it like:
  # config.vbguest.iso_path = "#{ENV['HOME']}/Downloads/VBoxGuestAdditions.iso"
  # or
  # config.vbguest.iso_path = "http://company.server/VirtualBox/$VBOX_VERSION/VBoxGuestAdditions.iso"
  
  # set auto_update to false, if do NOT want to check the correct additions 
  # version when booting this machine
  config.vbguest.auto_update = false
  
  # do NOT download the iso file from a webserver
  config.vbguest.no_remote = true
end
```

### Config options

* `iso_path` : The full path or URL to the VBoxGuestAdditions.iso file. <br/>
The `iso_path` may contain the optional placeholder `$VBOX_VERSION` for the detected version (e.g. `4.1.8`).
The URI for the actual iso download reads: `http://download.virtualbox.org/virtualbox/$VBOX_VERSION/VBoxGuestAdditions_$VBOX_VERSION.iso`<br/>
vbguest will try to autodetect the best option for your system. WTF? see below.
* `auto_update` (Boolean, default: `true`) : Whether to check the correct additions version on each start (where start is _not_ resuming a box).
* `no_install` (Boolean, default: `false`) : Whether to check the correct additions version only. This will warn you about version mis-matches, but will not try to install anything.
* `no_remote` (Boolean, default: `false`) : Whether to _not_ download the iso file from a remote location. This includes any `http` location!


### Running as a middleware

Running as a middleware will is the default way using *vagrant-vbguest*. 
It will run automatically right after the box started. This is each time the box boots, i.e. `vagrant up` or `vagrant reload`. 
It won't run on `vagrant resume` (or `vagrant up` a suspended box) to save you some time resuming a box.

You may switch off the middleware by setting the vm's config `vbguest.auto_update` to `false`.
This is a per box settings. On multi vm environments you need to set that for each vm.

When *vagrant-vbguest* is running it will provide you some logs:

    ...
    [default] Booting VM...
    [default] Waiting for VM to boot. This can take a few minutes.
    [default] VM booted and ready for use!
    [default] Installing Virtualbox Guest Additions 4.1.14 - guest's version is 4.1.12
    [default] Copy iso file /Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso into the box /tmp/VBoxGuestAdditions.iso
    [default] Copy installer file setup_debian.sh into the box /tmp/install_vbguest.sh
    stdin: is not a tty
    Reading package lists...
    Building dependency tree...
    Reading state information...
    linux-headers-2.6.32-30-server is already the newest version.
    dkms is already the newest version.
    0 upgraded, 0 newly installed, 0 to remove and 142 not upgraded.
    Verifying archive integrity... All good.
    Uncompressing VirtualBox 4.1.14 Guest Additions for Linux..........
    VirtualBox Guest Additions installer
    Removing installed version 4.1.12 of VirtualBox Guest Additions...
    tar: Record size = 8 blocks
    Removing existing VirtualBox DKMS kernel modules ...done.
    Removing existing VirtualBox non-DKMS kernel modules ...done.
    Building the VirtualBox Guest Additions kernel modules ...done.
    Doing non-kernel setup of the Guest Additions ...done.
    You should restart your guest to make sure the new modules are actually used

    Installing the Window System drivers ...fail!
    (Could not find the X.Org or XFree86 Window System.)
    ...


The plugin's part starts at `[default] Installing Virtualbox Guest Additions 4.1.14 - guest's version is 4.1.1`, telling you that:

* the guest addition of the box *default* are outdated (or mismatch) 
* which guest additions iso file will be used 
* which installer script will be used
* all the VirtualBox Guest Additions installer output.

No worries on the `Installing the Window System drivers ...fail!`. Most dev boxes you are using won't run a Window Server, thus it's absolutely save to ignore that error.

When everything is fine, and no update is needed, you see log like:

    ...
    [default] Booting VM...
    [default] Waiting for VM to boot. This can take a few minutes.
    [default] VM booted and ready for use!
    [default] Detected Virtualbox Guest Additions 4.1.14 --- OK.
    ...


### Running as a Command

When you switched off the middleware auto update, or you have a box up and running you may also run the installer manually.

```bash
$ vagrant vbguest [vm-name] [-f|--force] [-I|--no-install] [-R|--no-remote] [--iso VBoxGuestAdditions.iso]
```

For example, when you just updated Virtual Box on your host system, you should update the gust additions right away. However, you may need to reload the box to get the guest additions working.

If you want to check the guest additions versions, without installing them, you may run:

```bash
$ vagrant vbguest --no-install
```

Telling you either about a version mismatch:

    [default] Virtualbox Guest Additions on host: 4.1.14 - guest's version is 4.1.0

or a match: 

    [default] Detected Virtualbox Guest Additions 4.1.14 --- OK

### ISO autodetection

*vagrant-vbguest* will try to autodetect a VirtualBox GuestAdditions iso file on your system, which usually matches your installed version of VirtualBox. If it cannot find one, it downloads one from the web (virtualbox.org).   
Those places will be checked in order:

1. Checks your VirualBox "Virtual Media Maganger" for a DVD called "VBoxGuestAdditions.iso"
2. Guess by your host operating system:
  * for linux : `/usr/share/virtualbox/VBoxGuestAdditions.iso`
  * for Mac : `/Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso`
  * for Windows : `%PROGRAMFILES%/Oracle/VirtualBox/VBoxGuestAdditions.iso`


## Knows Issues

* The installer script, which mounts and runs the GuestAdditions Installer Binary, works on linux only. Most likely it will run on most unix-like plattform. 
* The installer script requires a directory `/mnt` on the guest system
* On multi vm boxes, the iso file will be downloaded for each vm