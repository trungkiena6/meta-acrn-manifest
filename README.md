# meta-acrn-manifest
Xml manifest files used by project meta-acrn. (Provides a Yocto layer to bundle poky images, Celadon Android, Zephyr, ACRN hypervisor together to a single image)


# Set up the development environment

meta-acrn consist of multiple Git repositories and repo is the tool that makes it easier to work with those repositories as a whole. Create a local bin/ directory, download the repo tool to that directory, and allow the binary executable with the following commands:

$ make -p ~/bin
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
$ export PATH=~/bin:$PATH


# Download the source

Create an empty directory that will hold the meta-acrn and Yocto source files and serve as the working directory. Enter the following commands to bring down the latest version of repo tool, including its most recent fixes. The URL specifies the manifest that refers various repositories used by meta-acrn, which are placed within the working directory. For now, a .repo folder is created to store the manifest and the metadata of the source repositories.

$ mkdir ~/acrn-workspace
$ cd ~/acrn-workspace
$ repo init -u https://github.com/liuming50/meta-acrn-manifest.git -b master

Enter the following command to pull down the meta-acrn source tree to your working directory. The repo sync operation might take time depending on your Internet download speed.

$ repo sync


# Build the source (take sos-image-weston and the bundled uos-image-weston as a example)

$ cd ~/acrn-workspace
$ . acrn-init-build-env
$ MACHINE=apl-nuc bitbake sos-image-weston


# Deploy the image to a USB disk (assume the USB disk is /dev/sdb on your host machine)

$ sudo dd if=~/acrn-workspace/build/tmp/deploy/images/apl-nuc/sos-image-weston-apl-nuc.wic.acrn of=/dev/sdb


# Run the ACRN image on a APL-NUC device

- Insert the USB disk to your APL-NUC device, choose USB disk as the UEFI boot device. (F10)
- You will be able to see "EFI ACRN Loader" entry on the boot menu, click it to choose boot from ACRN hypervisor.
- After system boots up, open a console, and start the default UOS (uos-image-weston)
  $ systemctl start acrn-guest@uos-image-weston.service
