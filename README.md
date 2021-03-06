# meta-acrn-manifest
Xml manifest files used by project meta-acrn. (Provides a Yocto layer to bundle poky images, Celadon Android, Zephyr, ACRN hypervisor together to a single image)


# Set up the development environment

meta-acrn consist of multiple Git repositories and repo is the tool that makes it easier to work with those repositories as a whole. Create a local bin/ directory, download the repo tool to that directory, and allow the binary executable with the following commands:

```
$ make -p ~/bin
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
$ export PATH=~/bin:$PATH
```


# Download the source

Create an empty directory that will hold the meta-acrn and Yocto source files and serve as the working directory. Enter the following commands to bring down the latest version of repo tool, including its most recent fixes. The URL specifies the manifest that refers various repositories used by meta-acrn, which are placed within the working directory. For now, a .repo folder is created to store the manifest and the metadata of the source repositories.

```
$ mkdir ~/acrn-workspace
$ cd ~/acrn-workspace
$ repo init -u https://github.com/liuming50/meta-acrn-manifest.git -b master
```

Enter the following command to pull down the meta-acrn source tree to your working directory. The repo sync operation might take time depending on your Internet download speed.

```
$ repo sync
```


# Build the source in a docker image (Default SOS/UOS: sos-image-weston, uos-image-weston, uos-image-celadon)

```
$ cd ~/acrn-workspace
$ ./setup -a /path/to/your/celadon-source
$ MACHINE=apl-nuc bitbake sos-image-weston
```

in which, /path/to/your/celadon-source is where your Celadon Android source locates at, please refer to [Build Celadon from source](https://01.org/projectceladon/documentation/getting_started/build-source)

Notice: there is a bug in Poky WIC, when run 'wic rm' in a Docker container with aufs storage driver, please refer to "Known issues" section bout the bug. So please try build natively on your host if you run into that issue within Docker build.


# Build the source directly on host (Only verified on Ubuntu-16.04 LTS)

To be able to build Celadon Android, firstly of all, you need download Celadon Android source and install the required packages, please refer to [Build Celadon from source](https://01.org/projectceladon/documentation/getting_started/build-source).

Set up the environment:

```
$ cd ~/acrn-workspace
$ . acrn-init-celadon-host-tools
$ . acrn-init-build-env
```

Add the following line to conf/local.conf:
```
EXTERNALSRC_pn-uos-image-celadon = "/path/to/your/celadon-source"
```

in which, /path/to/your/celadon-source is where your Celadon Android source locates at.

Now you can run bitbake to build the SOS/UOS images:
```
$ MACHINE=apl-nuc bitbake sos-image-weston
```


# Deploy the image to a USB disk (assume the USB disk is /dev/sdb on your host machine)

```
$ sudo dd if=~/acrn-workspace/build/tmp/deploy/images/apl-nuc/sos-image-weston-apl-nuc.wic.acrn of=/dev/sdb
```


# Run the ACRN image on a APL-NUC device

- Insert the USB disk to your APL-NUC device, choose USB disk as the UEFI boot device. (F10)
- You will be able to see "EFI ACRN Loader" entry on the boot menu, click it to choose boot from ACRN hypervisor.
- After system boots up, open a console, and start the Linux UOS uos-image-weston:

```
  $ systemctl start acrn-guest@uos-image-weston.service
```

- Or start the Celadon Android UOS:
```
  $ systemctl start acrn-guest@uos-image-celadon.service
```


# Known issues

Poky WIC has a problem running 'wic rm' in a Docker container with aufs storage driver, which is needed to generate a ACRN bootable image, this issue's being tracked by Yocto bugzilla:

[Yocto bug #12988](https://bugzilla.yoctoproject.org/show_bug.cgi?id=12988)
