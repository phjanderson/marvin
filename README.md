Marvin
======
Marvin the Paranoid Android Kernel Builder  
by phjanderson

More info / discussion:  
http://www.freaktab.com/showthread.php?6701-Marvin-automated-multi-device-kernel-build-tool

About
------

Marvin is an Android kernel build tool intended to make compiling kernels for multiple devices and multiple configurations easier. While it's originally designed for the Rockchip RK3188 platform, it could also be used for other platforms.

Linux kernels are configured using a .config file. The settings in the config file decide which components are added to the kernel and how they should be configured. The settings can be chosen by using make menuconfig or by editing the config file directly. Different hardware devices require a different config file, different settings such as CPU or RAM clock speeds also need different config files. For example, if you would want to build kernels for 3 different devices, with 2 different CPU speeds and 2 different RAM speeds, you will end up with 12 config files. Each config needs to be built manually.

Here's where Marvin comes to the rescue! Marvin works with a three level layered config system:

1. a base config, common to all devices and configurations
2. device configs, one for each device (T428, MK908, QX1, MK802IV, etc)
3. option configs, any choice of settings can be combined in one config (a set of CPU frequencies, etc)

The first contains a full config, the last two only the changes compared to the base config.

These 3 are combined in a build profile, for example:  
t428 cpu1800 ddr600

You can create a list of profiles and ask Marvin to build each configuration and package the kernels together with the generated config files and option descriptions in a single zip file.


Getting started
------
Marvin runs on linux. The recommended platform is Ubuntu 12.04 LTS 64bit:  
http://releases.ubuntu.com/precise/

You can download and install it into a virtual machine, for example in virtual box:  
https://www.virtualbox.org/

The following commands should be run from the terminal.

To download Marvin, you will first need to install git:  
sudo apt-get install git

Next, create a working directory:  
mkdir android  
cd android

Download Marvin:  
git clone https://github.com/phjanderson/marvin

You will also need a kernel source tree, for the RK3188 download:  
git clone https://github.com/phjanderson/Kernel-3188

By default Marvin is configured for the RK3188 platform and looks for the kernel sources in a folder Kernel-3188 next to the marvin folder.

Marvin can use other kernel source trees, but many of the bundled device and option configs require patches in the kernel source tree.

Enter the marvin directory and start Marvin:  
cd marvin  
./marvin

Marvin will show a list of all supported commands.

To build kernels, you'll need to have some software installed for cross compiling. Let's ask Marvin to install these (on Ubuntu):  
./marvin install_builddep


Selecting a configuration
------

The config command shows all available device and option configs:  
./marvin config

Let's choose a config for a T428:  
./marvin config t428

Marvin will generate the .config file and store your choice as the current build profile.

You can check the current build profile by running the config command:  
./marvin config

Let's add some options:  
./marvin config t428 cpu1800 vsfsam

This will add an overclock to 1800mhz and the sam321's vsync video fix to the config.


Building a kernel
------

Let's ask Marvin to build the kernel:  
./marvin build

This will compile the kernel and make a copy to the output folder in platform/rk3188, together with the config file and a text file containing the descriptions of the selected options.

Perhaps you want to post the kernel on a forum? You can ask Marvin to also package the kernel in a zip file and use a timestamp in the filename:  
./marving build_package

This will build the kernel and package it in the output folder.


Working with build profiles
------

You can provide Marvin with a list of build profiles:  
./marvin profiles

This will open a text editor where you can enter a list of build profiles, for example:  
t428 vsfsam  
t428 vsfsam cpu1800  
mk908 vsfsam  
mk908 vsfsam cpu1800

The default text editor will be used, to change the default text editor to Midnight Commander (mcedit):  
sudo apt-get install mc  
sudo update-alternatives –config editor  
Midnight Commander is also an excellent tool to browse around your system from a terminal

We can now ask Marvin to all build profiles at once:  
./marvin build_all

If you also want Marvin to package the kernels for you:  
./marvin build_package_all


Editing configs
------

Let's try to add a new option config:  
./marvin menuconfig_option test

Marvin will start make menuconfig, a UI where you can browse through the settings and change them to your wishes. Make some changes here and exit menuconfig saving the changes. Next a text editor will appear asking you to enter a description for this option config, add an appropriate description and exit the editor saving the changes. Marvin will now calculate the differences in the kernel config, show you a list of the changes you made and save these to the config file.

You can edit existing configs in the same way.

The calculated changes might sometimes also list settings which you didn't modify directly. This is normal, changing a setting can cause dependent settings to be altered as well.

You can do the same fore device configs:  
./marvin menuconfig_device

You can also use textconfig if you prefer to use a text editor instead of menuconfig:  
./marvin textconfig_option test  
./marvin textconfig_device  
This can come in handy sometimes as the menu structure is quite complex.

Warning: it is not recommended to edit the base config. Any device specific settings should go to the device configs. Most other settings can be added in option configs. You do not need to make a separate option config for each individual settings, you can bundle a complete set of your favorite settings in one option config and name it "tweaks" or so.

Marvin only stores the "changes" to the base config in the device and option configs. If you then change these same settings again in the base config, the end result might be unpredictable.

Probably the only reason for editing the base config is when there are changes in the config structure of the source tree or when switching to an entirely different source tree.

_RK3188 tip: the overclock settings are located in System Type > RK3188 Custom Tweaks > CPU/GPU/DDR Frequency Scaling_


Cleaning up
------

If you want to clean up your kernel source tree, you can use one of the following commands:

Run make clean on the source tree:  
./marvin clean  
Removes most of the files generated by the kernel build system, but keep the kernel configuration.

Run make mrproper on the source tree:  
./marvin mrproper  
Removes all of the generated files by the kernel build system, including the configuration and some various backup files.

Run make distclean on the source tree:  
./marvin distclean  
Does everything mrproper does and removes some editor backup and patch leftover files.


Settings
------

Marvin has default settings configured so you can use it right away. You might want to change some of it's settings though.

Let's change the for example the *builder alias*. This is a short name added to the file names of the generated kernels and folders, kind of like your signature.

First we make a copy of the example config file:  
cp platform/rk3188/settings.example platform/rk3188/settings

Now open it in an editor and add this line:  
`MARVIN_BUILDER_ALIAS="phja"`

The example config contains information about the available settings.
