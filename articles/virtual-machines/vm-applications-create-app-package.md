---
title: Create packages and scripts for VM applications
description: Learn how to create application packages and install scripts in different formats for Windows and Linux VM applications in Azure.
author: tanmaygore
ms.author: tagore
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 02/23/2026
ms.reviewer: jushiman
---

# Create packages for VM applications

To create a VM Application, you need application package and scripts to properly install, update, and delete the application. This article explains how to create packages and scripts for different operating systems and formats that you can use as VM application payloads.

## Package and Install VM Applications on Linux
Third party applications for Linux can be packaged in a few ways. Let's explore how to handle creating the install commands for some of the most common.

### .tar and .gz files

These files are compressed archives and can be extracted to a desired location. Check the installation instructions for the original package in case they need to be extracted to a specific location. If .tar.gz file contains source code, see the instructions for the package for how to install from source.

Example to install command to install `golang` on a Linux machine:

```bash
sudo tar -C /usr/local -xzf go_linux
```

Example remove command:

```bash
sudo rm -rf /usr/local/go
```

### Creating application packages using `.deb`, `.rpm`, and other platform specific packages for VMs with restricted internet access

You can download individual packages for platform specific package managers, but they usually don't contain all the dependencies. For these files, you must also include all dependencies in the application package, or have the system package manager download the dependencies through the repositories that are available to the VM. If you're working with a VM with restricted internet access, you must package all the dependencies yourself.

Figuring out the dependencies can be a bit tricky. There are third party tools that can show you the entire dependency tree.

Following process shows how to identify application dependencies, download them and package them together for different linux OS flavors. 

#### [Ubuntu](#tab/ubuntu)

In Ubuntu, you can run `sudo apt show <package_name> | grep Depends` to show all the packages that are installed when executing the `sudo apt-get install <packge_name>` command. Then you can use that output to download all `.deb` files to create an archive that can be used as the application package.

To create a VM Application package for installing PowerShell on Ubuntu, perform the following steps: 
1. Run the following commands to enable the repository to download PowerShell and to identify package dependencies on a new Ubuntu VM
   
```bash
# Download the Microsoft repository GPG keys
wget -q "https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb"
# Register the Microsoft repository GPG keys
sudo dpkg -i packages-microsoft-prod.deb
sudo rm -rf packages-microsoft-prod.deb
sudo apt update
sudo apt show powershell | grep Depends
```

2. Check the output of the line **Depends** which lists the following packages:

```output
Depends: libc6, lib32gcc-s1, libgssapi-krb5-2, libstdc++6, zlib1g, libicu72|libicu71|libicu70|libicu69|libicu68|libicu67|libicu66|libicu65|libicu63|libicu60|libicu57|libicu55|libicu52, libssl3|libssl1.1|libssl1.0.2|libssl1.
```

3. Download each of these files using `sudo apt-get download <package_name>` and create a tar compressed archive with all files.

- Ubuntu 18.04:

```bash
mkdir /tmp/powershell
cd /tmp/powershell
sudo apt-get download libc6
sudo apt-get download lib32gcc
sudo apt-get download libgssapi-krb5-2
sudo apt-get download libstdc++6
sudo apt-get download zlib1g
sudo apt-get download libssl1.1
sudo apt-get download libicu60
sudo apt-get download powershell
sudo tar -cvzf powershell.tar.gz *.deb
```

- Ubuntu 20.04:

```bash
mkdir /tmp/powershell
cd /tmp/powershell
sudo apt-get download libc6
sudo apt-get download lib32gcc-s1
sudo apt-get download libgssapi-krb5-2
sudo apt-get download libstdc++6
sudo apt-get download zlib1g
sudo apt-get download libssl1.1
sudo apt-get download libicu66
sudo apt-get download powershell
sudo tar -cvzf powershell.tar.gz *.deb
```

- Ubuntu 22.04:

```bash
mkdir /tmp/powershell
cd /tmp/powershell
sudo apt-get download libc6
sudo apt-get download lib32gcc-s1
sudo apt-get download libgssapi-krb5-2
sudo apt-get download libstdc++6
sudo apt-get download zlib1g
sudo apt-get download libssl3
sudo apt-get download libicu70
sudo apt-get download powershell
sudo tar -cvzf powershell.tar.gz *.deb
```

- Ubuntu 24.04:

```bash
mkdir /tmp/powershell
cd /tmp/powershell
sudo apt-get download libc6
sudo apt-get download lib32gcc-s1
sudo apt-get download libgssapi-krb5-2
sudo apt-get download libstdc++6
sudo apt-get download zlib1g
sudo apt-get download libssl3t64
sudo apt-get download libicu74
sudo apt-get download powershell
sudo tar -cvzf powershell.tar.gz *.deb
```

4. This tar archive is the application package file.

- The install command in this case is:

```bash
sudo tar -xvzf powershell.tar.gz && sudo dpkg -i *.deb
```

- And the remove command is:

```bash
sudo apt remove powershell
```

Use `sudo apt autoremove` instead of explicitly trying to remove all the dependencies. You may have installed other applications with overlapping dependencies, and in that case, an explicit remove command would fail.

In case you don't want to resolve the dependencies yourself, and `apt` is able to connect to the repositories, you can install an application with just one `.deb` file and let `apt` handle the dependencies.

Example install command:

```bash
dpkg -i <package_name> || apt --fix-broken install -y
```

#### [Red Hat](#tab/rhel)

In Red Hat, you can run `sudo yum deplist <package_name>` to show all the packages that are installed when executing the `sudo yum install <package_name>` command. Then you can use that output to download all `.rpm` files to create an archive that can be used as the application package.

1. Example, to create a VM Application package to install PowerShell for Red Hat, first run the following commands to enable the repository where PowerShell can be downloaded from and also to identify the package dependencies on a new Red Hat Enterprise Linux (RHEL) VM.

- RHEL 7:

```bash
# Register the Microsoft RedHat repository
curl https://packages.microsoft.com/config/rhel/7/prod.repo | sudo tee /etc/yum.repos.d/microsoft.repo

sudo yum deplist powershell
```

- RHEL 8:

```bash
# Register the Microsoft RedHat repository
curl https://packages.microsoft.com/config/rhel/8/prod.repo | sudo tee /etc/yum.repos.d/microsoft.repo

sudo dnf deplist powershell
```

2. Check the output of each of the dependency entries, the dependencies are named after `provider:`:

```output
  dependency: /bin/sh
   provider: bash.x86_64 4.2.46-35.el7_9
  dependency: libicu
   provider: libicu.x86_64 50.2-4.el7_7
   provider: libicu.i686 50.2-4.el7_7
  dependency: openssl-libs
   provider: openssl-libs.x86_64 1:1.0.2k-26.el7_9
   provider: openssl-libs.i686 1:1.0.2k-26.el7_9
```

3. Download each of these files using `sudo yum install --downloadonly <package_name>`, to download a package when isn't yet installed in the system, or `sudo yum reinstall --downloadonly <package_name>`, to download a package that's already installed in the system, and create a tar compressed archive with all files.

```bash
mkdir /tmp/powershell
cd /tmp/powershell
sudo yum reinstall --downloadonly --downloaddir=/tmp/powershell bash
sudo yum reinstall --downloadonly --downloaddir=/tmp/powershell libicu
sudo yum reinstall --downloadonly --downloaddir=/tmp/powershell openssl-libs
sudo yum install --downloadonly --downloaddir=/tmp/powershell powershell
sudo tar -cvzf powershell.tar.gz *.rpm
```

4. This tar archive is the application package file.

- The install command in this case is:

```bash
sudo tar -xvzf powershell.tar.gz && sudo yum install *.rpm -y
```

- And the remove command is:

```bash
sudo yum remove powershell
```

In case you don't want to resolve the dependencies yourself and yum/dnf is able to connect to the repositories, you can install an application with just one `.rpm` file and let yum/dnf handle the dependencies.

Example install command:

```bash
yum install <package.rpm> -y
```

#### [SUSE](#tab/sles)

In SUSE, you can run `sudo zypper info --requires <package_name>` to show all the packages that are installed when executing the `sudo zypper install <package_name>` command. Then you can use that output to download all `.rpm` files to create an archive that can be used as the application package.

1. Example, to create a VM Application package to install `azure-cli` for SUSE, first run the following commands to enable the repository where Azure CLI can be downloaded from and also to identify the package dependencies on a new SUSE VM.

```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo zypper addrepo --name 'Azure CLI' --check https://packages.microsoft.com/yumrepos/azure-cli azure-cli
sudo zypper info --requires azure-cli
```

2. Check the output after **Requires** which lists the following packages:

```output
Requires       : [98]
  /usr/bin/python3
  python(abi) = 3.11
  azure-cli-command-modules-nspkg >= 2.0
  azure-cli-nspkg >= 3.0.3
  python3-azure-loganalytics >= 0.1.0
  python3-azure-mgmt-apimanagement >= 3.0.0
  python3-azure-mgmt-authorization >= 4.0.0
  python3-azure-mgmt-batch >= 17.0.0
  python3-azure-mgmt-cognitiveservices >= 13.0.0
  python3-azure-mgmt-containerservice >= 28.0.0
  python3-azure-mgmt-cosmosdb >= 9.0.0
  python3-azure-mgmt-datalake-store >= 1.0.0
  python3-azure-mgmt-deploymentmanager >= 2.0.0
  python3-azure-mgmt-imagebuilder >= 1.2.0
  python3-azure-mgmt-iothubprovisioningservices >= 1.1.0
  python3-azure-mgmt-maps >= 2.0.0
  python3-azure-mgmt-media >= 10.0.0
<truncated>
...
<truncated>
  python3-vsts-cd-manager >= 1.0.2
  python3-websocket-client >= 1.6.0
  python3-xmltodict >= 0.13
  python3-azure-mgmt-keyvault >= 10.0.0
  python3-azure-mgmt-storage >= 21.0.0
  python3-azure-mgmt-billing >= 6.0.0
  python3-azure-mgmt-cdn >= 12.0.0
  python3-azure-mgmt-hdinsight >= 9.0.0
  python3-azure-mgmt-netapp >= 10.0.0
  python3-azure-mgmt-synapse >= 2.0.0
  azure-cli-core = 2.61.0
  python3-azure-batch >= 14.0
  python3-azure-mgmt-compute >= 30.0
  python3-azure-mgmt-containerregistry >= 10.0.0
  python3-azure-mgmt-databoxedge >= 1.0.0
  python3-azure-mgmt-network >= 25.0.0
  python3-azure-mgmt-security >= 5.0.0
```

3. Download each of these files using `sudo zypper install -f --download-only <package_name>` and create a tar compressed archive with all files.

```bash
mkdir /tmp/azurecli
cd /tmp/azurecli
for i in $(sudo zypper info --requires azure-cli | sed -n -e '/Requires*/,$p' | grep -v "Requires" | awk -F '[>=]' '{print $1}') ; do sudo zypper --non-interactive --pkg-cache-dir /tmp/azurecli install -f --download-only $i; done
for i in $(sudo find /tmp/azurecli -name "*.rpm") ; do sudo cp $i /tmp/azurecli; done
sudo tar -cvzf azurecli.tar.gz *.rpm
```

4. This tar archive is the application package file.

- The install command in this case is:

```bash
sudo tar -xvzf azurecli.tar.gz && sudo zypper --no-refresh --no-remote --non-interactive install *.rpm
```

- And the remove command is:

```bash
sudo zypper remove azure-cli
```

---

## Creating VM Applications on Windows

Most third party applications in Windows are available as .exe or .msi installers. Some are also available as extract and run zip files. Let us look at the best practices for each of them.

### .exe installer

Installer executables typically launch a user interface (UI) and require someone to select through the UI. If the installer supports a silent mode parameter, it should be included in your installation string.

Cmd.exe also expects executable files to have the extension `.exe`, so you need to rename the file to have the `.exe` extension.

If I want to create a VM Application package for `myApp.exe`, which ships as an executable, my VM Application is called 'myApp', so I write the command assuming the application package is in the current directory:

```terminal
"move .\\myApp .\\myApp.exe & myApp.exe /S -config myApp_config"
```

If the installer executable file doesn't support an uninstall parameter, you can sometimes look up the registry on a test machine to know where the uninstaller is located.

In the registry, the uninstall string is stored in `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\<installed application name>\UninstallString` so I would use the contents as my remove command:

```terminal
'\"C:\\Program Files\\myApp\\uninstall\\helper.exe\" /S'
```

### .msi installer

For command line execution of `.msi` installers, the commands to install or remove an application should use `msiexec`. Typically, `msiexec` runs as its own separate process and `cmd` doesn't wait for it to complete, which can lead to problems when installing more than one VM application.  The `start` command can be used with `msiexec` to ensure that the installation completes before the command returns. For example:

```terminal
start /wait %windir%\\system32\\msiexec.exe /i myapp /quiet /forcerestart /log myapp_install.log
```

Example remove command:

```terminal
start /wait %windir%\\system32\\msiexec.exe /x myapp /quiet /forcerestart /log myapp_uninstall.log
```

Typically, the `start` command would be called within a batch script. If used with the `/wait` parameter, the calling script is paused until the called process terminates. Once complete, the batch script would check for the `errorlevel` variable set by the `start` command and exit as follows:
 
```batch
start /wait %windir%\\system32\\msiexec.exe /i myapp /quiet /forcerestart /log myapp_install.log
if %errorlevel% neq 0 exit /b %errorlevel%
...
```

### Zipped files

For .zip or other zipped files, rename and unzip the contents of the application package to the desired destination.

Example install command:

```terminal
rename myapp myapp.zip && mkdir C:\myapp && powershell.exe -Command "Expand-Archive -path myapp.zip -destinationpath C:\myapp"
```

Example remove command:

```terminal
rmdir /S /Q C:\\myapp
```
---

## Next steps
- Learn more about [Azure VM Applications](vm-applications.md).
- Learn how to [create and deploy VM application packages](vm-applications-how-to.md).
- Learn how to [manage and delete Azure VM Applications](vm-applications-manage.md).