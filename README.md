# Ubuntu Autoinstall Generator 
## Updated for 21.10
A script to generate a fully-automated ISO image for installing Ubuntu onto a machine without human interaction. This uses the new autoinstall method
for Ubuntu 20.04 and newer.

This script was updated and adapted from the original found [here](https://github.com/covertsh/ubuntu-autoinstall-generator) for 21.10. Impish has no daily builds, so the script will always use the release version instead. Because of this, the `-r` option has been removed.

## [Looking for the desktop version?](https://github.com/covertsh/ubuntu-preseed-iso-generator)

### Behavior
Check out the usage information below for arguments. The basic idea is to take an unmodified Ubuntu ISO image, extract it, add some kernel command line parameters, then repack the data into a new ISO. This is needed for full automation because the ```autoinstall``` parameter must be present on the kernel command line, otherwise the installer will wait for a human to confirm. This script automates the process of creating an ISO with this built-in.

Autoinstall configuration (disk layout, language etc) can be passed along with cloud-init data to the installer. Some minimal information is needed for
the installer to work - see the Ubuntu documentation for an example, which is also in the ```user-data.example``` file in this repository (password: ubuntu). This data can be passed over the network (not yet supported in this script), via an attached volume, or be baked into the ISO itself.

To attach via a volume (such as a separate ISO image), see the Ubuntu autoinstall [quick start guide](https://ubuntu.com/server/docs/install/autoinstall-quickstart). It's really very easy! To bake everything into a single ISO instead, you can use the ```-a``` flag with this script and provide a user-data file containing the autoinstall configuration and optionally cloud-init data, plus a meta-data file if you choose. The meta-data file is optional and will be empty if it is not specified. With an 'all-in-one' ISO, you simply boot a machine using the ISO and the installer will do the rest. At the end the machine will reboot into the new OS.

This script can use an existing ISO image or download the latest daily image from the Ubuntu project. Using a fresh ISO speeds things up because there won't be as many packages to update during the installation.

By default, the source ISO image is checked for integrity and authenticity using GPG. This can be disabled with ```-k```.

### Requirements
Tested on Ubuntu 21.04 and Debian Buster.
- Utilities required:
    - ```xorriso```
    - ```sed```
    - ```curl```
    - ```gpg```

### Usage
```
Usage: ubuntu-autoinstall-generator.sh [-h] [-v] [-a] [-e] [-u user-data-file] [-m meta-data-file] [-k] [-c] [-s source-iso-file] [-d destination-iso-file]

💁 This script will create fully-automated Ubuntu 21.10 Impish Indri installation media.

Available options:

-h, --help              Print this help and exit
-v, --verbose           Print script debug info
-a, --all-in-one        Bake user-data and meta-data into the generated ISO. By default you will
                        need to boot systems with a CIDATA volume attached containing your
                        autoinstall user-data and meta-data files.
                        For more information see: https://ubuntu.com/server/docs/install/autoinstall-quickstart
-e, --use-hwe-kernel    Force the generated ISO to boot using the hardware enablement (HWE) kernel.
-u, --user-data         Path to user-data file. Required if using -a
-m, --meta-data         Path to meta-data file. Will be an empty file if not specified and using -a
-k, --no-verify         Disable GPG verification of the source ISO file. By default SHA256SUMS-<current date> and
                        SHA256SUMS-<current date>.gpg files in the script directory will be used to verify the authenticity and integrity
                        of the source ISO file. If they are not present the latest daily SHA256SUMS will be
                        downloaded and saved in the script directory. The Ubuntu signing key will be downloaded and
                        saved in a new keyring in the script directory.
-s, --source            Source ISO file. By default the latest ISO for Ubuntu 21.10 will be downloaded
                        and saved as <script directory>/ubuntu-original-<current date>.iso
                        That file will be used by default if it already exists.
-d, --destination       Destination ISO file. By default <script directory>/ubuntu-autoinstall-<current date>.iso will be
                        created, overwriting any existing file.
-t, --no-timestamp      Omit timestamps in logging output.
-i, --no-icons          Omit icons in logging output.
```

### Example
```
user@testbox:~$ bash ubuntu-autoinstall-generator.sh -a -u user-data.example -d ubuntu-autoinstall-example.iso
[2022-01-20 10:43:38] 👶 Starting up...
[2022-01-20 10:43:38] 🔎 Checking for current release...
[2022-01-20 10:43:38] 💿 Current release is 21.10
[2022-01-20 10:43:38] 📁 Created temporary working directory /tmp/tmp.fXG2y4Rv84
[2022-01-20 10:43:38] 🔎 Checking for required utilities...
[2022-01-20 10:43:38] 👍 All required utilities are installed.
[2022-01-20 10:43:38] 🌎 Downloading ISO image for Ubuntu 21.10 Impish Indri...
[2022-01-20 10:44:24] 👍 Downloaded and saved to /home/user/ubuntu-autoinstall-generator/ubuntu-21.10-live-server-amd64.iso
[2022-01-20 10:44:24] 🌎 Downloading SHA256SUMS & SHA256SUMS.gpg files...
[2022-01-20 10:44:25] 🌎 Downloading and saving Ubuntu signing key...
[2022-01-20 10:44:25] 👍 Downloaded and saved to /home/user/ubuntu-autoinstall-generator/843938DF228D22F7B3742BC0D94AA3F0EFE21092.keyring
[2022-01-20 10:44:25] 🔐 Verifying /home/user/ubuntu-autoinstall-generator/ubuntu-21.10-live-server-amd64.iso integrity and authenticity...
[2022-01-20 10:44:29] 👍 Verification succeeded.
[2022-01-20 10:44:29] 🗄️ Extracting MBR template and EFI partition...
[2022-01-20 10:44:29] 👍 Extracted and saved to ubuntu-21.10-amd64.mbr and ubuntu-21.10-amd64.efi
[2022-01-20 10:44:29] 🔧 Extracting ISO image...
[2022-01-20 10:44:30] 👍 Extracted to /tmp/tmp.fXG2y4Rv84
[2022-01-20 10:44:30] 🧩 Adding autoinstall parameter to kernel command line...
[2022-01-20 10:44:30] 👍 Added parameter to UEFI and BIOS kernel command lines.
[2022-01-20 10:44:30] 🧩 Setting GRUB timeout to 5 seconds...
[2022-01-20 10:44:30] 👍 GRUB boot timeout set to 5 seconds.
[2022-01-20 10:44:30] 🧩 Adding user-data and meta-data files...
[2022-01-20 10:44:30] 👍 Added data and configured kernel command line.
[2022-01-20 10:44:30] 👷 Updating /tmp/tmp.fXG2y4Rv84/md5sum.txt with hashes of modified files...
[2022-01-20 10:44:30] 👍 Updated hashes.
[2022-01-20 10:44:30] 📦 Repackaging extracted files into an ISO image...
[2022-01-20 10:44:31] 💿 Repackaged into /home/user/ubuntu-autoinstall-generator/ubuntu-autoinstall-example.iso
[2022-01-20 10:44:31] ✅ Completed.
[2022-01-20 10:44:31] 🗑️ Deleted temporary working directory /tmp/tmp.fXG2y4Rv84
```

Now you can boot your target machine using ```ubuntu-autoinstall-example.iso``` and it will automatically install Ubuntu using the configuration from ```user-data.example```.

### Thanks
Based on [covertsh's script](https://github.com/covertsh/ubuntu-autoinstall-generator), which in turn was based on [this](https://betterdev.blog/minimal-safe-bash-script-template/) minimal safe bash template, and steps found in [this](https://discourse.ubuntu.com/t/please-test-autoinstalls-for-20-04/15250) discussion thread (particularly [this](https://gist.github.com/s3rj1k/55b10cd20f31542046018fcce32f103e) script).
The somewhat outdated Ubuntu documentation [here](https://help.ubuntu.com/community/LiveCDCustomization#Assembling_the_file_system) was also useful.
Likewise [this discussion](https://askubuntu.com/questions/1289400/remaster-installation-image-for-ubuntu-20-10) on AskUbuntu.


### License
MIT license.
