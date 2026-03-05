---
title: Create and deploy VM Application on Azure
description: Learn how to create and deploy applications on Azure Virtual Machine (VM) using Azure Compute Gallery.
author: tanmaygore
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 08/18/2024
ms.author: tagore
ms.reviewer: jushiman
ms.custom: devx-track-azurepowershell, devx-track-azurecli
# Customer intent: As a cloud engineer, I want to create and deploy VM Application packages in an Azure Compute Gallery, so that I can efficiently manage, share, and distribute applications for my virtual machines.
---

# Create and deploy VM Application

VM Application is a resource type in Azure Compute Gallery that simplifies publishing, deployment, management, sharing, and global distribution of applications and scripts for your virtual machines. 
VM Applications support a wide range of scenarios, including high-scale deployments, low-latency requirements, failure resiliency, secure trusted rollouts, fleet-wide consistency, microservice architectures, and post-deployment management. [Learn more about VM Applications](./vm-applications.md).

To create and deploy applications on Azure VM, first package and upload your application to Azure Storage Account as a storage blob. Then create `Azure VM application` resource and `VM application version` resource referencing these storage blobs. Azure stores and replicates these blobs in regional managed storage accounts. Finally, deploy the application on any VM or Virtual Machine Scale Sets by passing application reference in `applicationProfile`.

:::image type="content" source="media/vmapps/vm-applications-how-to.png" alt-text="Diagram showing step-by-step process involved in publishing and deploying VM applications.":::

## Prerequisites
1. Create [Azure storage account](/azure/storage/common/storage-account-create#create-a-storage-account) and [storage container](/azure/storage/blobs/blob-containers-portal#create-a-container). This container is used to upload your application files. 
1. Create [Azure Compute Gallery for storing and sharing application resources](create-gallery.md).
1. Create [user-assigned managed identity](/entra/identity/managed-identities-azure-resources/overview) and [assign to Azure Compute Gallery](vm-applications-publish-with-managed-identity.md)

> [!NOTE]
> Create your storage account with anonymous access disabled for improved security. Use a [managed identity assigned to Azure Compute Gallery](vm-applications-publish-with-managed-identity.md) to publish your VM Applications securely without requiring SAS tokens.

## Package the application
Before publishing the VM Application resource in Azure Compute Gallery, package your application files for upload to Azure Storage Account. The following diagram shows possible folder structures for organizing your application files:

:::image type="content" source="media/vmapps/vm-application-folder-structure.png" alt-text="Screenshot showing the folder structure recommended for uploading and creating VM Applications.":::

### 1. Package the application files
VM Application requires a single file as the application package. 
  - If your application installation requires a single executable file (.exe, .msi, .sh, .ps, etc.) then you can use it as the application package.
  - If your application requires multiple files (executables, dependencies, scripts, metadata, etc.), archive them into a single file (`.zip`, `.tar`, `.tar.gz`) and use this file as the application package.
  - For microservice application, you can package and publish each microservice as a separate Azure VM Application. This modularization facilitates application reusability, cross-team development, and sequential installation of microservices using `order` property in the [applicationProfile](#deploy-the-vm-apps).

  Learn more about [creating application packages for Azure VM Applications](vm-applications-create-app-package.md). 
     
### 2. (Optional) Package the application configuration file
   - You can provide the configuration file separately using `defaultConfigurationLink` in the `publishingProfile` of the VM Application. This reduces the overhead of archiving and unarchiving application packages. 
   - Configuration files can also be passed during app deployment using `configurationReference` property in the `applicationProfile` of the VM enabling customized installation per VM. 
   - If both properties are set, `configurationReference` overrides the configuration passed using `defaultConfigurationLink`. 
     
### 3. Create the install script
After the application and configuration blob is downloaded on the VM, Azure executes the provided install script to install the application. **The install script is provided as a string** and has a maximum character limit of 4,096 chars. The install commands should be written assuming the application package and the configuration file are in the current directory.

There may be few operations required to be performed in the install script

- **(Optional) Use the right command interpreter**
	The default command interpreter used by Azure are `/bin/bash` on Linux OS and `cmd.exe` on Windows OS. It's possible to use a different interpreter like Chocolatey or PowerShell, if its installed on the machine. Call the executable and pass the commands to it. For example, `powershell.exe -command '<powershell command>'`. If you're using PowerShell, you need to be using version 3.11.0 of the Az.Storage module.

- **(Optional) Rename application blob and configuration blob**
	Azure can't retain the original blob name and the file extensions. Azure downloads the application file with the name of the VM application resource published on Azure Compute Gallery. So if name of VM App is "VMApp1", the file downloaded on VM has the name "VMApp1" without an extension. The configuration file has a default name as "VMApp1-config" without a file extension.
  
  You can rename the file with the file extension using the install script or you can also pass the names in `packageFileName` and `configFileName` properties of the [`publishingProfile` of VM Application version resource](#create-the-vm-application). Azure uses these names instead of default names while downloading the files. 
   
- **Unarchive application blob**
	For archived application packages, it needs to be unarchived before installing the application. Use .zip or .tar since most OS has built-in support for unarchiving these formats. For other formats, make sure the Guest OS provides support.     

- **Convert the script to string**
   	The install script is passed as a string for the `install` property in the `publishingProfile` of Azure VM Application version resource. 

- **Install quietly and synchronously**
  Install applications quietly or silently without any user interface prompts. Since the installation runs on a VM where customer input isn't available, you must use silent or unattended installation flags. Additionally, if the application installer spawns background or child processes or runs asynchronously, the parent script might return before the installation completes. To ensure the full installation finishes before Azure marks the operation as successful, use wait flags or blocking commands. This approach helps when installation involves multiple steps that run asynchronously or you need to verify installation completes before the subsequent application in the `order` sequence begins.

   Common examples include: 
    - **Windows .msi**: `start /wait msiexec /i app.msi /quiet /qn`
    - **Windows .exe**: `start /wait app.exe /silent` or `start /wait app.exe /quiet` or `start /wait app.exe /S`
    - **PowerShell**: `Start-Process -FilePath '.\app.exe' -ArgumentList '/quiet' -Wait`
    - **Linux .sh**: `./install.sh --silent` (shell scripts run synchronously by default)
    - **Linux .deb**: `sudo DEBIAN_FRONTEND=noninteractive dpkg -i app.deb` (runs synchronously by default)
    - **Linux .rpm**: `sudo rpm -ivh app.rpm` (runs synchronously by default) 

- **(Optional) Set right execution policy and permissions**
	After unarchiving, file permissions could be reset. It's a good practice to set the right permissions before executing the files.

- **(Optional) Set verbose execution and logging**
  VM Applications automatically captures any output from the install script and stores it in a stdout file on the VM. Use this capability to log installation progress, debug failures, and verify successful deployments.
  - **Enable verbose mode** to capture application installer logs. Bash: `set -x`, PowerShell: `$VerbosePreference='Continue'`, Cmd: `@echo on`.
  - **Add custom messages** to track script progress. Bash:  `echo "Install started"`, PowerShell: `Write-Output "Install started"`, Cmd: `echo Install started`
 
  You can retrieve these logs using [Run Command](./vm-applications-manage.md#view-logs-of-application-installation-using-run-command) or by connecting to the VM. The `stdout` and `stderr` files are located at the following path
  - Linux: `/var/lib/waagent/Microsoft.CPlat.Core.VMApplicationManagerLinux/<application name>/<application version>/`
  - Windows: `C:\Packages\Plugins\Microsoft.CPlat.Core.VMApplicationManagerWindows\1.0.9\Downloads\<application name>\<application version>/`

- **(Optional) Move application and configuration blob to appropriate location**
	Azure downloads the application blob and configuration blob to following locations. The install script must move the files to appropriate locations when necessary.

	Linux: `/var/lib/waagent/Microsoft.CPlat.Core.VMApplicationManagerLinux/<application name>/<application version>`

	Windows: `C:\Packages\Plugins\Microsoft.CPlat.Core.VMApplicationManagerWindows\1.0.16\Downloads\<application name>\<application version>`

Here are few sample install scripts based on the file extension of the application blob. If `packageFileName` & `configFileName` properties are provided in the `publishingProfile` with file name and extension, remove the code for renaming files in following scripts.

#### [.TAR](#tab/TAR)
Replace `VMApp1` with your VM App name in application package (VMApp1) and configuration file (VMApp1-config).
Replace `app.tar` with desired app name. 
Replace `config.yaml` with your desired config name.

**Install using bash without config file:**
```bash
mv VMApp1 app.tar && tar -xf app.tar && chmod -R +xr . && bash ./install.sh
```
```bash
mv VMApp1 app.tar && tar -xf app.tar && chmod -R +xr . && sudo DEBIAN_FRONTEND=noninteractive dpkg -i ./app.deb
```

**Install using bash with config file:**
```bash
mv VMApp1 app.tar && mv VMApp1-config config.yaml && tar -xf app.tar && chmod -R +xr . && bash ./install.sh --config config.yaml
```
If config file is packaged within the tar file
```bash
mv VMApp1 app.tar && tar -xf app.tar && chmod -R +xr . && sudo debconf-set-selections < config.cfg && sudo DEBIAN_FRONTEND=noninteractive dpkg -i ./app.deb
```

#### [.ZIP](#tab/ZIP)
Replace `VMApp1` with your VM App name in application package (VMApp1) and configuration file (VMApp1-config).
Replace `app.zip` with desired app name. 
Replace `config.json` with your desired config name.

Following examples assumes vm application is installing python application and the pythonInstaller.exe is packaged inside app.zip. 

**Install using CMD without config file:**

Replace VMApp1 with your VM App name and app.zip with desired name for your executable in the below script. 

```cli-interactive
ren VMApp1 app.zip && tar -xf app.zip && start /wait pythonInstaller.exe /quiet InstallAllUsers=1 PrependPath=1
```

**Install using CMD with config file:**
```cli-interactive
ren VMApp1 app.zip && tar -xf app.zip && ren VMApp1-config config.json && start /wait pythonInstaller.exe --config config.json /quiet InstallAllUsers=1 PrependPath=1
```

If config file is included in the .zip package
```cli-interactive
ren VMApp1 app.zip && tar -xf app.zip && start /wait pythonInstaller.exe --config config.json /quiet InstallAllUsers=1 PrependPath=1
```

**Install using PowerShell without config file:**
```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Rename-Item VMApp1 app.zip; Expand-Archive -Path '.\app.zip' -DestinationPath '.'; Start-Process -FilePath '.\pythonInstaller.exe' -ArgumentList '/quiet','InstallAllUsers=1','PrependPath=1' -Wait -NoNewWindow"
```

**Install using PowerShell with config file:**
```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Rename-Item VMApp1 app.zip; Rename-Item VMApp1-config config.json Expand-Archive -Path '.\app.zip' -DestinationPath '.'; Start-Process -FilePath '.\pythonInstaller.exe' -ArgumentList '--config','config.json','/quiet','InstallAllUsers=1','PrependPath=1' -Wait -NoNewWindow"
```
If config file is included in the .zip package
```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Rename-Item VMApp1 app.zip; Expand-Archive -Path '.\app.zip' -DestinationPath '.'; Start-Process -FilePath '.\pythonInstaller.exe' -ArgumentList '--config','config.json','/quiet','InstallAllUsers=1','PrependPath=1' -Wait -NoNewWindow"
```

#### [.EXE](#tab/EXE)
Replace `VMApp1` with your VM App name in application package (VMApp1) and configuration file (VMApp1-config).
Replace `pythonInstaller.exe` with desired app name. 
Replace `config.json` with your desired config name.

Following examples assumes vm application is installing python application using EXE. 

**Install using CMD without config file:**

Replace VMApp1 with your VM App name and pythonInstaller.exe with desired name for your executable. 

```cli-interactive
ren VMApp1 pythonInstaller.exe && start /wait pythonInstaller.exe /quiet InstallAllUsers=1 PrependPath=1
```

**Install using CMD with config file:**
```cli-interactive
ren VMApp1 pythonInstaller.exe && ren VMApp1-config config.json && start /wait pythonInstaller.exe --config config.json /quiet InstallAllUsers=1 PrependPath=1
```

**Install using PowerShell without config file:**
```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Rename-Item VMApp1 pythonInstaller.exe; Start-Process -FilePath '.\pythonInstaller.exe' -ArgumentList '/quiet','InstallAllUsers=1','PrependPath=1' -Wait -NoNewWindow"
```

**Install using PowerShell with config file:**
```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Rename-Item VMApp1 pythonInstaller.exe; Rename-Item VMApp1-config config.json; Start-Process -FilePath '.\pythonInstaller.exe' -ArgumentList '--config','config.json','/quiet','InstallAllUsers=1','PrependPath=1' -Wait -NoNewWindow"
```

#### [.MSI](#tab/MSI)
Replace `VMApp1` with your VM App name in application package (VMApp1) and configuration file (VMApp1-config).
Replace `pythonInstaller.msi` with desired app name. 
Replace `config.json` with your desired config name.

Following examples assumes vm application is installing python application using MSI. 

**Install using CMD without config file:**

Replace VMApp1 with your VM App name and pythonInstaller.msi with desired name for your MSI file. 

```cli-interactive
ren VMApp1 pythonInstaller.msi && msiexec /i pythonInstaller.msi /quiet InstallAllUsers=1 PrependPath=1
```

**Install using CMD with config file:**
```cli-interactive
ren VMApp1 pythonInstaller.msi && ren VMApp1-config config.json && msiexec /i pythonInstaller.msi CONFIGFILE=config.json /quiet InstallAllUsers=1 PrependPath=1
```

**Install using PowerShell without config file:**
```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Rename-Item VMApp1 pythonInstaller.msi; Start-Process -FilePath 'msiexec.exe' -ArgumentList '/i','.\pythonInstaller.msi','/quiet','InstallAllUsers=1','PrependPath=1' -Wait -NoNewWindow"
```

**Install using PowerShell with config file:**
```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Rename-Item VMApp1 pythonInstaller.msi; Rename-Item VMApp1-config config.json; Start-Process -FilePath 'msiexec.exe' -ArgumentList '/i','.\pythonInstaller.msi','CONFIGFILE=config.json','/quiet','InstallAllUsers=1','PrependPath=1' -Wait -NoNewWindow"
```

#### [.DEB](#tab/DEB)
Replace `VMApp1` with your VM App name in application package (VMApp1) and configuration file (VMApp1-config).
Replace `app.deb` with desired app name. 
Replace `config.cfg` with your desired config name.

**Install using bash without config file:**
```bash
mv VMApp1 app.deb && chmod +xr app.deb && sudo DEBIAN_FRONTEND=noninteractive dpkg -i ./app.deb
```

**Install using bash with config file:**
```bash
mv VMApp1 app.deb && mv VMApp1-config config.cfg  && chmod +xr app.deb && sudo debconf-set-selections < config.cfg && sudo DEBIAN_FRONTEND=noninteractive dpkg -i ./app.deb
```


#### [.RPM](#tab/RPM)
Replace `VMApp1` with your VM App name in application package (VMApp1) and configuration file (VMApp1-config).
Replace `app.rpm` with desired app name. 
Replace `config.cfg` with your desired config name.

**Install using bash without config file:**
```bash
mv VMApp1 app.rpm && chmod +xr app.rpm && sudo rpm -ivh ./app.rpm
```

**Install using bash with config file:**
```bash
mv VMApp1 app.rpm && mv VMApp1-config config.cfg && chmod +xr app.rpm && sudo rpm -ivh ./app.rpm --rcfile config.cfg
```

#### [.SH](#tab/SH)
Replace `VMApp1` with your VM App name in application package (VMApp1) and configuration file (VMApp1-config).
Replace `install.sh` with desired script name. 
Replace `config.yaml` with your desired config name.

**Install using bash without config file:**
```bash
mv VMApp1 install.sh && chmod +xr install.sh && bash ./install.sh
```

**Install using bash with config file:**
```bash
mv VMApp1 install.sh && mv VMApp1-config config.yaml && chmod +xr install.sh && bash ./install.sh --config config.yaml
```

#### [.PS1](#tab/powershell)
Replace `VMApp1` with your VM App name in application package (VMApp1) and configuration file (VMApp1-config).
Replace `install.ps1` with desired script name. 
Replace `config.json` with your desired config name.

**Install using PowerShell without config file:**
```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Rename-Item -Path '.\VMApp1' -NewName 'install.ps1'; .\install.ps1"
```

**Install using PowerShell with config file:**
```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Rename-Item -Path '.\VMApp1' -NewName 'install.ps1'; Rename-Item -Path '.\VMApp1-config' -NewName 'config.json'; .\install.ps1 -ConfigFile 'config.json'"
```
---

### 4. Create the remove script

The `remove` script enables you to define the operations for removing the application. The `remove` script is provided as a string and has a maximum character limit of 4,096 characters. Write the commands assuming the application package and the configuration file are in the current directory. During uninstall operation, Azure runs the uninstall script and then deletes all files from the repository. 

There are few operations that the `remove` script must perform. 

- **Uninstall application:**
  Properly uninstall the application from the VM. For example: 

#### [CMD](#tab/cmd1)

CMD on Windows: 

```cli-interactive
start /wait uninstall.exe /quiet
```

```cli-interactive
start /wait python.exe /uninstall /quiet
```

```cli-interactive
start /wait msiexec /x app.msi /quiet /qn
```

#### [PowerShell](#tab/powershell1)

PowerShell on Windows:

```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Start-Process -FilePath '.\uninstall.exe' -ArgumentList '/quiet' -Wait -NoNewWindow"
```

```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Start-Process -FilePath '.\python.exe' -ArgumentList '/uninstall','/quiet' -Wait -NoNewWindow"
```

```powershell-interactive
powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "Start-Process -FilePath 'msiexec.exe' -ArgumentList '/x','.\app.msi','/quiet','/qn' -Wait -NoNewWindow"
```

#### [Bash](#tab/bash1)

Bash on Linux: 

```bash
sudo ./uninstall.sh
```

```bash
sudo apt remove -y app
```

```bash
sudo rpm -e app
```

---

- **Remove residual files:**
   	If the application installation creates files in other parts of the file system, remove those files.


## Upload the application files to Azure storage account

> [!IMPORTANT]
> Before uploading files or generating SAS URLs, assign roles to the user, service principal, or managed identity that performs these operations on the target Storage account or container:
> - Storage Blob Data Contributor or Storage Blob Data Owner: required to upload, modify, or delete blobs.
> - For generating SAS with Azure AD (for example, using `--auth-mode` login in CLI/PowerShell), also assign Storage Blob Delegator at the storage account scope.
>
### 1. **Upload your application and configuration files to a [container](/azure/storage/blobs/blob-containers-portal#create-a-container) in your [Azure storage account](/azure/storage/common/storage-account-create)**.

Upload your packaged application and optional configuration file as blobs. Block blobs work for most scenarios. 

Optionally, if you need to use page blobs, byte-align your files before uploading using one of the following scripts:

#### [CLI](#tab/cli2)
```azurecli-interactive
inputFile="<the file you want to pad>"

# Get the file size
fileSize=$(stat -c %s "$inputFile")

# Calculate the remainder when divided by 512
remainder=$((fileSize % 512))

if [ "$remainder" -ne 0 ]; then
  # Calculate how many bytes to pad
  difference=$((512 - remainder))
  
  # Create padding (empty bytes)
  dd if=/dev/zero bs=1 count=$difference >> "$inputFile"
fi
```

#### [PowerShell](#tab/powershell2)
```azurepowershell-interactive
$inputFile = <the file you want to pad>

$fileInfo = Get-Item -Path $inputFile

$remainder = $fileInfo.Length % 512

if ($remainder -ne 0){

  $difference = 512 - $remainder

  $bytesToPad = [System.Byte[]]::CreateInstance([System.Byte], $difference)

  Add-Content -Path $inputFile -Value $bytesToPad -Encoding Byte
}
```
---

### 2. **Generate SAS or Blob URL for the application package and the configuration file** 
Once the application and configuration files are uploaded to the storage account, you need to generate a Blob URL or [SAS URL](/azure/storage/common/storage-sas-overview#get-started-with-sas) with read privilege for these blobs. These URLs are then provided as reference while creating the VM Application version resource. 

> [!NOTE]
> Use of blob URL with managed identity is strongly recommended for improved security and privacy.

**Use blob URL:**
- If [managed identity is assigned to Azure Compute Gallery](vm-applications-publish-with-managed-identity.md).
- If storage account is enabled for anonymous access. Use this method only for testing since its insecure. 

**Use SAS URL:**
- If storage account is disabled for anonymous access and managed identity isn't assigned to Azure Compute Gallery. 

Blob URL = https://${STORAGE_ACCOUNT}.blob.core.windows.net/${CONTAINER_NAME}/${BLOB_NAME}
SAS URL  = https://${STORAGE_ACCOUNT}.blob.core.windows.net/${CONTAINER_NAME}/${BLOB_NAME}?${SAS_TOKEN}

#### [CLI using Blob URL](#tab/cli31)

Use the following script if [managed identity is assigned to Azure Compute Gallery](vm-applications-publish-with-managed-identity.md). This approach uses blob URLs without SAS tokens.

```azurecli-interactive
#!/bin/bash
set -euo pipefail

# === CONFIGURATION ===
STORAGE_ACCOUNT="your-storage-account-name"
CONTAINER_NAME="your-container-name"
APP_FILE="./path/to/your-app-file"           # Path to your application payload file        
CONFIG_FILE="./path/to/your-config-file"     # Path to your configuration file (optional)   

# === LOGIN (if not already logged in) ===
az login --only-show-errors

# === CREATE CONTAINER IF NOT EXISTS ===
az storage container create \
  --name "$CONTAINER_NAME" \
  --account-name "$STORAGE_ACCOUNT" \
  --auth-mode login \
  --only-show-errors

# === UPLOAD APPLICATION FILE ===
APP_BLOB_NAME=$(basename "$APP_FILE")
az storage blob upload \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --name "$APP_BLOB_NAME" \
  --file "$APP_FILE" \
  --auth-mode login \
  --overwrite \
  --only-show-errors

# === UPLOAD CONFIG FILE (optional) ===
if [ -n "$CONFIG_FILE" ] && [ -f "$CONFIG_FILE" ]; then
  CONFIG_BLOB_NAME=$(basename "$CONFIG_FILE")
  az storage blob upload \
    --account-name "$STORAGE_ACCOUNT" \
    --container-name "$CONTAINER_NAME" \
    --name "$CONFIG_BLOB_NAME" \
    --file "$CONFIG_FILE" \
    --auth-mode login \
    --overwrite \
    --only-show-errors
fi

# === GENERATE BLOB URLs ===
echo "Generating Blob URLs..."

APP_BLOB_URL="https://${STORAGE_ACCOUNT}.blob.core.windows.net/${CONTAINER_NAME}/${APP_BLOB_NAME}"
echo "Application file: $APP_BLOB_URL"

if [ -n "$CONFIG_FILE" ] && [ -f "$CONFIG_FILE" ]; then
  CONFIG_BLOB_URL="https://${STORAGE_ACCOUNT}.blob.core.windows.net/${CONTAINER_NAME}/${CONFIG_BLOB_NAME}"
  echo "Configuration file: $CONFIG_BLOB_URL"
fi
```

#### [CLI using SAS URL](#tab/cli32)

Use the following script if storage account has anonymous access disabled and managed identity isn't assigned to Azure Compute Gallery. This approach generates time-limited SAS tokens.

```azurecli-interactive
#!/bin/bash
set -euo pipefail

# === CONFIGURATION ===
STORAGE_ACCOUNT="yourstorageaccount"
CONTAINER_NAME="yourcontainer"
APP_FILE="./your-app-file"           # Path to your application payload file
CONFIG_FILE="./your-config-file"     # Path to your configuration file (optional)
SAS_EXPIRY_HOURS=24

# === LOGIN (if not already logged in) ===
az login --only-show-errors

# === CREATE CONTAINER IF NOT EXISTS ===
az storage container create \
  --name "$CONTAINER_NAME" \
  --account-name "$STORAGE_ACCOUNT" \
  --auth-mode login \
  --only-show-errors

# === UPLOAD APPLICATION FILE ===
APP_BLOB_NAME=$(basename "$APP_FILE")
az storage blob upload \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --name "$APP_BLOB_NAME" \
  --file "$APP_FILE" \
  --auth-mode login \
  --overwrite \
  --only-show-errors

# === UPLOAD CONFIG FILE (optional) ===
if [ -n "$CONFIG_FILE" ] && [ -f "$CONFIG_FILE" ]; then
  CONFIG_BLOB_NAME=$(basename "$CONFIG_FILE")
  az storage blob upload \
    --account-name "$STORAGE_ACCOUNT" \
    --container-name "$CONTAINER_NAME" \
    --name "$CONFIG_BLOB_NAME" \
    --file "$CONFIG_FILE" \
    --auth-mode login \
    --overwrite \
    --only-show-errors
fi

# === GENERATE SAS URLs ===
# Note: Using --auth-mode login --as-user requires "Storage Blob Delegator" role at the storage account scope

echo "Generating SAS URLs..."
EXPIRY=$(date -u -d "+$SAS_EXPIRY_HOURS hours" '+%Y-%m-%dT%H:%MZ')

APP_SAS_TOKEN=$(az storage blob generate-sas \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --name "$APP_BLOB_NAME" \
  --permissions r \
  --expiry "$EXPIRY" \
  --auth-mode login \
  --as-user \
  -o tsv)

APP_SAS_URL="https://${STORAGE_ACCOUNT}.blob.core.windows.net/${CONTAINER_NAME}/${APP_BLOB_NAME}?${APP_SAS_TOKEN}"
echo "Application file: $APP_SAS_URL"

if [ -n "$CONFIG_FILE" ] && [ -f "$CONFIG_FILE" ]; then
  CONFIG_SAS_TOKEN=$(az storage blob generate-sas \
    --account-name "$STORAGE_ACCOUNT" \
    --container-name "$CONTAINER_NAME" \
    --name "$CONFIG_BLOB_NAME" \
    --permissions r \
    --expiry "$EXPIRY" \
    --auth-mode login \
    --as-user \ 
    -o tsv)

  CONFIG_SAS_URL="https://${STORAGE_ACCOUNT}.blob.core.windows.net/${CONTAINER_NAME}/${CONFIG_BLOB_NAME}?${CONFIG_SAS_TOKEN}"
  echo "Configuration file: $CONFIG_SAS_URL"
fi
```

#### [PowerShell using Blob URL](#tab/powershell31)

Use the following script if [managed identity is assigned to Azure Compute Gallery](vm-applications-publish-with-managed-identity.md). This approach uses blob URLs without SAS tokens.

```azurepowershell-interactive
# === CONFIGURATION ===
$subscriptionId = "your-subscription-id"
$resourceGroupName = "yourresourcegroup"
$storageAccountName = "yourstorageaccount"
$containerName = "yourcontainer"
$appFile = "C:\path\to\your-app-file"           # Path to your application payload file
$configFile = "C:\path\to\your-config-file"     # Path to your configuration file (optional, set to $null if not needed)
$sasExpiryHours = 24

# === LOGIN (if not already logged in) ===
Connect-AzAccount -Subscription $subscriptionId

# === GET STORAGE CONTEXT ===
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
$context = $storageAccount.Context

# === CREATE CONTAINER IF NOT EXISTS ===
$container = Get-AzStorageContainer -Name $containerName -Context $context -ErrorAction SilentlyContinue
if (-not $container) {
    New-AzStorageContainer -Name $containerName -Context $context -Permission Off
}

# === UPLOAD APPLICATION FILE ===
$appBlobName = Split-Path -Leaf $appFile
Set-AzStorageBlobContent `
    -File $appFile `
    -Container $containerName `
    -Blob $appBlobName `
    -Context $context `
    -Force

# === UPLOAD CONFIG FILE (optional) ===
if ($configFile -and (Test-Path $configFile)) {
    $configBlobName = Split-Path -Leaf $configFile
    Set-AzStorageBlobContent `
        -File $configFile `
        -Container $containerName `
        -Blob $configBlobName `
        -Context $context `
        -Force
}

# === GENERATE BLOB URLs ===
Write-Host "`nGenerating Blob URLs..."

$appBlobUrl = "https://$storageAccountName.blob.core.windows.net/$containerName/$appBlobName"
Write-Host "Application file: $appBlobUrl"

if ($configFile -and (Test-Path $configFile)) {
  $configBlobUrl = "https://$storageAccountName.blob.core.windows.net/$containerName/$configBlobName"
  Write-Host "Configuration file: $configBlobUrl"
}
```

#### [PowerShell using SAS URL](#tab/powershell32)

Use the following script if storage account has anonymous access disabled and [managed identity isn't assigned to Azure Compute Gallery](vm-applications-publish-with-managed-identity.md). This approach generates time-limited SAS tokens.

```azurepowershell-interactive
# === CONFIGURATION ===
$subscriptionId = "your-subscription-id"
$resourceGroupName = "yourresourcegroup"
$storageAccountName = "yourstorageaccount"
$containerName = "yourcontainer"
$appFile = "C:\path\to\your-app-file"           # Path to your application payload file
$configFile = "C:\path\to\your-config-file"     # Path to your configuration file (optional, set to $null if not needed)
$sasExpiryHours = 24

# === LOGIN (if not already logged in) ===
Connect-AzAccount -Subscription $subscriptionId

# === GET STORAGE CONTEXT ===
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
$context = $storageAccount.Context

# === CREATE CONTAINER IF NOT EXISTS ===
$container = Get-AzStorageContainer -Name $containerName -Context $context -ErrorAction SilentlyContinue
if (-not $container) {
    New-AzStorageContainer -Name $containerName -Context $context -Permission Off
}

# === UPLOAD APPLICATION FILE ===
$appBlobName = Split-Path -Leaf $appFile
Set-AzStorageBlobContent `
    -File $appFile `
    -Container $containerName `
    -Blob $appBlobName `
    -Context $context `
    -Force

# === UPLOAD CONFIG FILE (optional) ===
if ($configFile -and (Test-Path $configFile)) {
    $configBlobName = Split-Path -Leaf $configFile
    Set-AzStorageBlobContent `
        -File $configFile `
        -Container $containerName `
        -Blob $configBlobName `
        -Context $context `
        -Force
}

# === GENERATE SAS URLs ===
Write-Host "`nGenerating SAS URLs..."
$expiryTime = (Get-Date).AddHours($sasExpiryHours)

$appSasToken = New-AzStorageBlobSASToken `
    -Container $containerName `
    -Blob $appBlobName `
    -Permission r `
    -ExpiryTime $expiryTime `
    -Context $context

$appSasUrl = "https://$storageAccountName.blob.core.windows.net/$containerName/$appBlobName$appSasToken"
Write-Host "Application file: $appSasUrl"

if ($configFile -and (Test-Path $configFile)) {
    $configSasToken = New-AzStorageBlobSASToken `
        -Container $containerName `
        -Blob $configBlobName `
        -Permission r `
        -ExpiryTime $expiryTime `
        -Context $context

    $configSasUrl = "https://$storageAccountName.blob.core.windows.net/$containerName/$configBlobName$configSasToken"
    Write-Host "Configuration file: $configSasUrl"
}
```

#### [Portal](#tab/portal3)

1. In the [Azure portal](https://portal.azure.com), navigate to your storage account and select **Containers**.
1. Select your container (or create one), then select **Upload**.
1. Browse to your application package and configuration files, select them, and select **Upload**.
1. After upload completes, select each blob and copy the **URL**. Generate SAS tokens if [managed identity isn't attached to compute gallery](vm-applications-publish-with-managed-identity.md) or anonymous access is disabled.

For detailed steps, see [Upload blobs to Azure Storage using the portal](/azure/storage/blobs/storage-quickstart-blobs-portal).

---

## Create the VM Application
To create the VM Application, first create the **VM Application** resource, which describes the application. Then create **VM Application Version** resource within it, which contains the VM application payload and scripts to install, update, and delete the application. Payload is supplied using Blob URL or SAS URL to the blob container in Azure Storage Account. 

Refer [schema for VM Application and VM Application version resource](vm-applications.md#publish-application-as-azure-vm-application) to learn more about each property. 

#### [Template](#tab/template4)

Use the following ARM template to create an Azure Compute Gallery, VM Application, and VM Application version. This template demonstrates the key properties and configuration options for publishing your application package.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2020-06-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "galleryName": {
      "type": "string"
    },
    "applicationName": {
      "type": "string"
    },
    "versionName": {
      "type": "string",
      "metadata": {
        "description": "Must follow the format: major.minor.patch (Example: 1.0.0)"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "West US"
    },
    "supportedOSType": {
      "type": "string",
      "allowedValues": ["Windows", "Linux"]
    },
    "endOfLifeDate": {
      "type": "string",
      "metadata": {
        "description": "Optional. This property is for information only and doesn't block app deployment."
      }
    },
    "description": {
      "type": "string",
      "defaultValue": "Description of the application"
    },
    "eula": {
      "type": "string",
      "defaultValue": ""
    },
    "privacyStatementUri": {
      "type": "string",
      "defaultValue": ""
    },
    "releaseNoteUri": {
      "type": "string",
      "defaultValue": ""
    },
    "mediaLink": {
      "type": "string"
    },
    "configLink": {
      "type": "string"
    },
    "appConfigFileName": {
      "type": "string"
    },
    "appPackageFileName": {
      "type": "string"
    },
    "replicaRegion1": {
      "type": "string",
      "defaultValue": "East US"
    },
    "replicaRegion2": {
      "type": "string",
      "defaultValue": "South Central US"
    },
    "installScript": {
      "type": "string",
      "metadata": {
        "description": "Optional. Script to run to install the application. Example: echo 'Installing application...'"
      }
    },
    "updateScript": {
      "type": "string",
      "metadata": {
        "description": "Optional. Script to run to update the application. Example: echo 'Updating application...'"
      }
    },
    "removeScript": {
      "type": "string",
      "metadata": {
        "description": "Optional. Script to run to delete the application. Example: echo 'Deleting application...'"
      }
    },
    "storageAccountType": {
      "type": "string",
      "allowedValues": ["PremiumV2_LRS", "Premium_LRS", "Standard_LRS", "Standard_ZRS"],
      "defaultValue": "Standard_LRS"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/galleries",
      "apiVersion": "2024-03-03",
      "name": "[parameters('galleryName')]",
      "location": "[parameters('location')]",
      "properties": {
        "identifier": {}
      }
    },
    {
      "type": "Microsoft.Compute/galleries/applications",
      "apiVersion": "2024-03-03",
      "name": "[format('{0}/{1}', parameters('galleryName'), parameters('applicationName'))]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/galleries', parameters('galleryName'))]"
      ],
      "properties": {
        "supportedOSType": "[parameters('supportedOSType')]",
        "endOfLifeDate": "[parameters('endOfLifeDate')]",
        "description": "[parameters('description')]",
        "eula": "[if(equals(parameters('eula'), ''), json('null'), parameters('eula'))]",
        "privacyStatementUri": "[if(equals(parameters('privacyStatementUri'), ''), json('null'), parameters('privacyStatementUri'))]",
        "releaseNoteUri": "[if(equals(parameters('releaseNoteUri'), ''), json('null'), parameters('releaseNoteUri'))]"
      }
    },
    {
      "type": "Microsoft.Compute/galleries/applications/versions",
      "apiVersion": "2024-03-03",
      "name": "[format('{0}/{1}/{2}', parameters('galleryName'), parameters('applicationName'), parameters('versionName'))]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Compute/galleries/applications', parameters('galleryName'), parameters('applicationName'))]"
      ],
      "properties": {
        "publishingProfile": {
          "source": {
            "mediaLink": "[parameters('mediaLink')]",
            "defaultConfigurationLink": "[parameters('configLink')]"
          },
          "manageActions": {
            "install": "[parameters('installScript')]",
            "remove": "[parameters('removeScript')]",
            "update": "[parameters('updateScript')]"
          },
          "settings": {
            "scriptBehaviorAfterReboot": "Rerun",
            "configFileName": "[parameters('appConfigFileName')]",
            "packageFileName": "[parameters('appPackageFileName')]"
          },
          "targetRegions": [
            {
              "name": "[parameters('location')]",
              "regionalReplicaCount": 3,
              "storageAccountType": "[parameters('storageAccountType')]"
            },
            {
              "name": "[parameters('replicaRegion1')]",
              "regionalReplicaCount": 1,
              "storageAccountType": "[parameters('storageAccountType')]"
            },
            {
              "name": "[parameters('replicaRegion2')]"
            },
          ],
          "excludeFromLatest": false,
          "replicaCount": 2,
          "storageAccountType": "[parameters('storageAccountType')]"
        },
        "safetyProfile": {
          "allowDeletionOfReplicatedLocations": true
        },
        "endOfLifeDate": "[parameters('endOfLifeDate')]"
      }
    }
  ]
}

```

#### [REST](#tab/rest4)

**Create the VM Application definition** using the ['create gallery application API'](/rest/api/compute/gallery-applications)

- We're creating a VM application definition named *myApp*.

```rest
PUT
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{applicationName}?api-version=2024-03-03

{
  "location": "West US",
  "name": "myApp",
  "properties": {
    "supportedOSType": "Windows | Linux",
    "endOfLifeDate": "2030-01-01",
	  "description": "Description of the App",
	  "eula": "Link to End-User License Agreement (EULA)",
	  "privacyStatementUri": "Link to privacy statement for the application",
	  "releaseNoteUri": "Link to release notes for the application"
  }
}

```

**Create a VM application version** using the ['create gallery application version API'](/rest/api/compute/gallery-applications).

Next we're creating version within the application definition. It takes blob/SAS URL created in previous step to pull application and configuration blob from storage account.

1. Update mediaLink, install and remove properties for your application.
1. (Optional) Update defaultConfigurationLink to pass configuration file. 
1. (Optional) Update packageFileName and configFileName enabling Azure to download files with this name. These properties eliminate the need to rename downloaded files in install script. 

```rest
PUT
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{applicationName}/versions/{versionName}?api-version=2024-03-03

{
  "location": "West US",
  "properties": {
    "publishingProfile": {
      "source": {
        "mediaLink": "Blob or SAS URL of application blob",
        "defaultConfigurationLink": "Blob or SAS URL of configuration blob. Optional"
      },
      "manageActions": {
        "install": "Install script for application as string",
        "remove": "Uninstall script for application as string",
        "update": "Update script for application as string. Optional "
      },
      "targetRegions": [
        {
          "name": "West US",
          "regionalReplicaCount": 1
        },
	      {
	        "name": "East US. Optional"
	      }
      ]
      "endofLifeDate": "2030-01-01",
      "replicaCount": 1,
      "excludeFromLatest": false,
      "storageAccountType": "PremiumV2_LRS | Premium_LRS | Standard_LRS | Standard_ZRS",
      "safetyProfile": {
	      "allowDeletionOfReplicatedLocations": false
      },
      "settings": {
	      "scriptBehaviorAfterReboot": "None | Rerun",
	      "configFileName": "Name for downloaded configuration file on the VM",
	      "packageFileName": "Name for downloaded application file on the VM"
      }
   }
}

```

#### [CLI](#tab/cli4)

**Create the VM application definition** using ['az sig gallery-application create'](/cli/azure/sig/gallery-application#az_sig_gallery_application_create).

- We're creating a VM application definition named *myApp* for Linux-based VMs. 
- VM applications require [Azure CLI](/cli/azure/install-azure-cli) version 2.30.0 or later.

```azurecli-interactive
application_name="myApp"
gallery_name="myGallery"
resource_group="myResourceGroup"
location="East US"
os_type="Linux"

az sig gallery-application create \
  --application-name "$application_name" \
  --gallery-name "$gallery_name" \
  --resource-group "$resource_group" \
  --os-type "$os_type" \
  --location "$location"
```

**Create a VM application version** using ['az sig gallery-application version create'](/cli/azure/sig/gallery-application/version#az-sig-gallery-application-version-create). 

Next we're creating version within the application definition. It takes blob/SAS Url created in previous step to pull application blob from storage account.

1. Update package_url, install_command, remove_command for your application.
1. (Optional) Uncomment and update config_url, and default-configuration-file-link to optionally pass configuration file. 
1. (Optional) Update package_file_name and config_file_name. Azure downloads files with this name eliminating the need to rename downloaded files in install script. 

```azurecli-interactive
version_name="1.0.0"
package_url="Blob or SAS URL for application blob"
package_file_name="Name to give for downloaded application file on VM"
# config_url="Blob or SAS URL for configuration"
# config_file_name="Name to give for downloaded config file on VM"

install_command="install script for your application"
remove_command="uninstall script for you application"
target_regions='[
  {"name": "South Central US", "regionalReplicaCount": 2},
  {"name": "West Europe", "regionalReplicaCount": 1}
]'

az sig gallery-application version create \
   --version-name "$version_name" \
   --application-name "$application_name" \
   --gallery-name "$gallery_name" \
   --location "$location" \
   --resource-group "$resource_group" \
   --package-file-link "$package_url" \
   --package-file-name "$package_file_name" \
   --install-command "$install_command" \
   --remove-command "$remove_command" \
   --target-regions "$target_regions"
   # --default-configuration-file-link "$config_url" \
   # --config-file-name "$config_file_name" \

```


#### [PowerShell](#tab/powershell4)

**Create the VM Application definition** using ['New-AzGalleryApplication'](/powershell/module/az.compute/new-azgalleryapplication). 
- We're creating a Linux app named *myApp* in the *myGallery* Azure Compute Gallery and in the *myGallery* resource group. 
- Replace the values for variables as needed.

```azurepowershell-interactive
$galleryName = "myGallery"
$rgName = "myResourceGroup"
$applicationName = "myApp"
$description = "Backend Linux application for finance."
$location = "East US"

New-AzGalleryApplication `
  -ResourceGroupName $rgName `
  -GalleryName $galleryName `
  -Location $location `
  -Name $applicationName `
  -SupportedOSType Windows `
  -Description $description
```

**Create a version of your VM Application** using ['New-AzGalleryApplicationVersion'](/powershell/module/az.compute/new-azgalleryapplicationversion). 

Next we're creating version within the application definition. It takes blob/SAS Url created in previous step to pull application blob from storage account.

1. Update package_url, install_command, remove_command for your application.
1. (Optional) Uncomment and update config_url, and default-configuration-file-link to optionally pass configuration file. 
1. (Optional) Update package_file_name and config_file_name. Azure downloads files with this name eliminating the need to rename downloaded files in install script. 

```azurepowershell-interactive
$galleryName = "myGallery"
$rgName = "myResourceGroup"
$location = "East US"
$applicationName = "myApp"
$version = "1.0.0"

$package_url = "Blob or SAS URL for application blob in storage account"
$package_file_name="Name to give for downloaded application file on VM"
# $config_url = "Blob or SAS URL for configuration blob in storage account"
# $config_file_name="Name to give for downloaded config file on VM"
$install_command = "Install script for your application"
$remove_command = "Uninstall script for your application"
$target_regions = @(
  @{ Name = "South Central US"; RegionalReplicaCount = 2 },
  @{ Name = "West Europe"; RegionalReplicaCount = 1 }
)

New-AzGalleryApplicationVersion `
   -ResourceGroupName $rgName `
   -GalleryName $galleryName `
   -GalleryApplicationName $applicationName `
   -Name $version `
   -PackageFileLink $package_url `
   -PackageFileName $package_file_name `
   -Location $location `
   -Install $install_command `
   -Remove $remove_command `
   -TargetRegion $target_regions
   # -DefaultConfigFileLink $config_url `
   # -ConfigFileName $config_file_name `

```

#### [Portal](#tab/portal4)

1. Go to the [Azure portal](https://portal.azure.com), then search for and select **Azure Compute Gallery**.
1. Select the gallery you want to use from the list.
1. On the page for your gallery, select **Add** from the top of the page and then select **VM application definition** from the drop-down. The **Create a VM application definition** page opens.
1. In the **Basics** tab, enter a name for your application and choose whether the application is for VMs running Linux or Windows.
1. Select the **Publishing options** tab if you want to specify any of the following optional settings for your VM Application definition:
    - A description of the VM Application definition.
    - End of life date
    - Link to an End User License Agreement (EULA)
    - URI of a privacy statement
    - URI for release notes
1. When you're done, select **Review + create**.
1. When validation completes, select **Create** to have the definition deployed.
1. After deployment completes, select **Go to resource**.
1. On the page for the application, select **Create a VM application version**. The **Create a VM Application Version** page opens.
1. Enter a version number like 1.0.0.
1. Select the region where your application packages are uploaded.
1. Under **Source application package**, select **Browse**. Select the storage account, then the container where your package is located. Select the package from the list and then select **Select** when you're done. Alternatively, you can paste the SAS URI in this field if preferred.
1. Provide the '**Install script**'.
1. Provide the '**Uninstall script**'.
1. Optionally, provide the '**Update script**'.
1. If you have a default configuration file uploaded to a storage account, you can select it in **Default configuration**.
1. Select **Exclude from latest** if you don't want this version to appear as the latest version when you create a VM.
1. For **End of life date**, choose a date in the future to track when this version should be retired. It isn't deleted or removed automatically, it's only for your own tracking.
1. To replicate this version to other regions, select the **Replication** tab, add more regions, and make changes to the number of replicas per region. The original region where your version was created must be in the list and can't be removed.
1. When you're done making changes, select **Review + create** at the bottom of the page.
1. When validation shows as passed, select **Create** to deploy your VM application version.

#### [GitHub Actions](#tab/ga4)

```yaml
name: Deploy Azure VM Application

on:
  push:
    branches:
      - main

env:
  APP_FILE: app.exe
  CONFIG_FILE: app-config.json

  AZURE_RESOURCE_GROUP: ${{ secrets.AZURE_RESOURCE_GROUP }}
  AZURE_LOCATION: ${{ secrets.AZURE_LOCATION }}
  AZURE_STORAGE_ACCOUNT: ${{ secrets.AZURE_STORAGE_ACCOUNT }}
  AZURE_CONTAINER_NAME: ${{ secrets.AZURE_CONTAINER_NAME }}
  GALLERY_NAME: ${{ secrets.GALLERY_NAME }}
  APPLICATION_NAME: ${{ secrets.AZURE_VM_APPLICATION_NAME }}

jobs:
  deploy:
    runs-on: ubuntu-latest

    permissions:
      id-token: write
      contents: read

    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Azure Login
      uses: azure/login@v2
      with:
        client-id: ${{ secrets.AZURE_CLIENT_ID }}
        tenant-id: ${{ secrets.AZURE_TENANT_ID }}
        subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

    - name: Upload files to Azure Blob Storage
      run: |
        set -euo pipefail

        echo "Creating container if missing..."
        az storage container create \
          --name "$AZURE_CONTAINER_NAME" \
          --account-name "$AZURE_STORAGE_ACCOUNT" \
          --auth-mode login \
          --only-show-errors

        echo "Uploading files..."
        az storage blob upload \
          --account-name "$AZURE_STORAGE_ACCOUNT" \
          --container-name "$AZURE_CONTAINER_NAME" \
          --name "$APP_FILE" \
          --file "$APP_FILE" \
          --auth-mode login \
          --overwrite \
          --only-show-errors

        az storage blob upload \
          --account-name "$AZURE_STORAGE_ACCOUNT" \
          --container-name "$AZURE_CONTAINER_NAME" \
          --name "$CONFIG_FILE" \
          --file "$CONFIG_FILE" \
          --auth-mode login \
          --overwrite \
          --only-show-errors

    - name: Create VM Application if missing
      run: |
        set -euo pipefail

        echo "Checking for existing VM Application..."
        if ! az sig gallery-application show \
          --resource-group "$AZURE_RESOURCE_GROUP" \
          --gallery-name "$GALLERY_NAME" \
          --application-name "$APPLICATION_NAME" &>/dev/null; then

          az sig gallery-application create \
            --resource-group "$AZURE_RESOURCE_GROUP" \
            --gallery-name "$GALLERY_NAME" \
            --application-name "$APPLICATION_NAME" \
            --location "$AZURE_LOCATION" \
            --os-type Windows
        fi

    - name: Generate SAS URLs
      id: sas
      run: |
        set -euo pipefail

        echo "Generating SAS URLs valid for 24 hours..."
        EXPIRY=$(date -u -d "+1 day" '+%Y-%m-%dT%H:%MZ')

        APP_SAS=$(az storage blob generate-sas \
          --account-name "$AZURE_STORAGE_ACCOUNT" \
          --container-name "$AZURE_CONTAINER_NAME" \
          --name "$APP_FILE" \
          --permissions r \
          --expiry "$EXPIRY" \
          --auth-mode login \
          --as-user \
          -o tsv)

        CONFIG_SAS=$(az storage blob generate-sas \
          --account-name "$AZURE_STORAGE_ACCOUNT" \
          --container-name "$AZURE_CONTAINER_NAME" \
          --name "$CONFIG_FILE" \
          --permissions r \
          --expiry "$EXPIRY" \
          --auth-mode login \
          --as-user \
          -o tsv)

        echo "APP_SAS=$APP_SAS" >> $GITHUB_ENV
        echo "CONFIG_SAS=$CONFIG_SAS" >> $GITHUB_ENV

    - name: Create VM Application Version
      run: |
        set -euo pipefail

        MAJOR=1
        MINOR=0
        PATCH=$(date +%Y%m%d)
        VERSION="$MAJOR.$MINOR.$PATCH"

        INSTALL_CMD=$(jq -Rs '.' < install-script-as-string.txt)
        REMOVE_CMD=$(jq -Rs '.' < uninstall-script-as-string.txt)

        PACKAGE_URL="https://${AZURE_STORAGE_ACCOUNT}.blob.core.windows.net/${AZURE_CONTAINER_NAME}/${APP_FILE}?${APP_SAS}"
        CONFIG_URL="https://${AZURE_STORAGE_ACCOUNT}.blob.core.windows.net/${AZURE_CONTAINER_NAME}/${CONFIG_FILE}?${CONFIG_SAS}"

        az sig gallery-application version create \
          --resource-group "$AZURE_RESOURCE_GROUP" \
          --gallery-name "$GALLERY_NAME" \
          --application-name "$APPLICATION_NAME" \
          --version-name "$VERSION" \
          --location "$AZURE_LOCATION" \
          --package-file-link "$PACKAGE_URL" \
          --default-configuration-file-link "$CONFIG_URL" \
          --install-command "$INSTALL_CMD" \
          --remove-command "$REMOVE_CMD" \
          --only-show-errors
```

#### [Azure DevOps Pipeline](#tab/devops4)
```yaml
trigger:
  branches:
    include: [ main ]

variables:
  APP_FILE: app.exe
  CONFIG_FILE: app-config.json
  AZURE_RESOURCE_GROUP: $(AZURE_RESOURCE_GROUP)
  AZURE_LOCATION: $(AZURE_LOCATION)
  AZURE_STORAGE_ACCOUNT: $(AZURE_STORAGE_ACCOUNT)
  AZURE_CONTAINER_NAME: $(AZURE_CONTAINER_NAME)
  GALLERY_NAME: $(GALLERY_NAME)
  APPLICATION_NAME: $(AZURE_VM_APPLICATION_NAME)

stages:
  - stage: DeployVMApp
    displayName: Upload files and deploy Azure VM Application
    jobs:
      - job: Deploy
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - checkout: self

          - task: AzureCLI@2
            displayName: Upload app and config to Blob
            inputs:
              azureSubscription: 'AzureServiceConnection'
              scriptType: bash
              scriptLocation: inlineScript
              inlineScript: |
                set -euo pipefail

                echo "Creating container if it doesn't exist..."
                az storage container create \
                  --name "$AZURE_CONTAINER_NAME" \
                  --account-name "$AZURE_STORAGE_ACCOUNT" \
                  --auth-mode login \
                  --only-show-errors

                echo "Uploading files..."
                az storage blob upload \
                  --account-name "$AZURE_STORAGE_ACCOUNT" \
                  --container-name "$AZURE_CONTAINER_NAME" \
                  --name "$APP_FILE" \
                  --file "$APP_FILE" \
                  --auth-mode login \
                  --overwrite \
                  --only-show-errors

                az storage blob upload \
                  --account-name "$AZURE_STORAGE_ACCOUNT" \
                  --container-name "$AZURE_CONTAINER_NAME" \
                  --name "$CONFIG_FILE" \
                  --file "$CONFIG_FILE" \
                  --auth-mode login \
                  --overwrite \
                  --only-show-errors

          - task: AzureCLI@2
            displayName: Create VM Application Definition
            inputs:
              azureSubscription: 'AzureServiceConnection'
              scriptType: bash
              scriptLocation: inlineScript
              inlineScript: |
                set -euo pipefail

                echo "Checking for existing VM Application..."
                if ! az sig gallery-application show \
                  --resource-group "$AZURE_RESOURCE_GROUP" \
                  --gallery-name "$GALLERY_NAME" \
                  --application-name "$APPLICATION_NAME" &>/dev/null; then
                  echo "Creating new VM Application..."
                  az sig gallery-application create \
                    --resource-group "$AZURE_RESOURCE_GROUP" \
                    --gallery-name "$GALLERY_NAME" \
                    --application-name "$APPLICATION_NAME" \
                    --location "$AZURE_LOCATION" \
                    --os-type Windows
                else
                  echo "VM Application definition already exists."
                fi

          - task: AzureCLI@2
            displayName: Generate SAS URLs
            inputs:
              azureSubscription: 'AzureServiceConnection'
              scriptType: bash
              scriptLocation: inlineScript
              inlineScript: |
                set -euo pipefail

                echo "Generating SAS URLs valid for 24 hours..."
                EXPIRY=$(date -u -d "+1 day" '+%Y-%m-%dT%H:%MZ')

                APP_SAS=$(az storage blob generate-sas \
                  --account-name "$AZURE_STORAGE_ACCOUNT" \
                  --container-name "$AZURE_CONTAINER_NAME" \
                  --name "$APP_FILE" \
                  --permissions r \
                  --expiry "$EXPIRY" \
                  --auth-mode login \
                  --as-user \
                  -o tsv)

                CONFIG_SAS=$(az storage blob generate-sas \
                  --account-name "$AZURE_STORAGE_ACCOUNT" \
                  --container-name "$AZURE_CONTAINER_NAME" \
                  --name "$CONFIG_FILE" \
                  --permissions r \
                  --expiry "$EXPIRY" \
                  --auth-mode login \
                  --as-user \
                  -o tsv)

                echo "##vso[task.setvariable variable=APP_SAS]$APP_SAS"
                echo "##vso[task.setvariable variable=CONFIG_SAS]$CONFIG_SAS"

          - task: AzureCLI@2
            displayName: Create VM Application Version
            inputs:
              azureSubscription: 'AzureServiceConnection'
              scriptType: bash
              scriptLocation: inlineScript
              inlineScript: |
                set -euo pipefail

                MAJOR=1
                MINOR=0
                PATCH=$(date +%Y%m%d)
                VERSION="$MAJOR.$MINOR.$PATCH"

                INSTALL_CMD=$(jq -Rs '.' < install-script-as-string.txt)
                REMOVE_CMD=$(jq -Rs '.' < uninstall-script-as-string.txt)

                PACKAGE_URL="https://${AZURE_STORAGE_ACCOUNT}.blob.core.windows.net/${AZURE_CONTAINER_NAME}/${APP_FILE}?${APP_SAS}"
                CONFIG_URL="https://${AZURE_STORAGE_ACCOUNT}.blob.core.windows.net/${AZURE_CONTAINER_NAME}/${CONFIG_FILE}?${CONFIG_SAS}"

                az sig gallery-application version create \
                  --resource-group "$AZURE_RESOURCE_GROUP" \
                  --gallery-name "$GALLERY_NAME" \
                  --application-name "$APPLICATION_NAME" \
                  --version-name "$VERSION" \
                  --location "$AZURE_LOCATION" \
                  --package-file-link "$PACKAGE_URL" \
                  --default-configuration-file-link "$CONFIG_URL" \
                  --install-command "$INSTALL_CMD" \
                  --remove-command "$REMOVE_CMD" \
                  --only-show-errors
```

#### [GitLab Pipeline](#tab/gitlab4)
```yaml
stages:
  - deploy

variables:
  APP_FILE: "app.exe"
  CONFIG_FILE: "app-config.json"
  AZURE_RESOURCE_GROUP: "$AZURE_RESOURCE_GROUP"
  AZURE_LOCATION: "$AZURE_LOCATION"
  AZURE_STORAGE_ACCOUNT: "$AZURE_STORAGE_ACCOUNT"
  AZURE_CONTAINER_NAME: "$AZURE_CONTAINER_NAME"
  GALLERY_NAME: "$GALLERY_NAME"
  APPLICATION_NAME: "$AZURE_VM_APPLICATION_NAME"

deploy_vm_app:
  image: mcr.microsoft.com/azure-cli
  stage: deploy
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  script:
    - az login --service-principal -u "$AZURE_CLIENT_ID" -p "$AZURE_CLIENT_SECRET" --tenant "$AZURE_TENANT_ID"
    - az account set --subscription "$AZURE_SUBSCRIPTION_ID"

    - |
      echo "Creating container and uploading files..."
      az storage container create \
        --name "$AZURE_CONTAINER_NAME" \
        --account-name "$AZURE_STORAGE_ACCOUNT" \
        --auth-mode login \
        --only-show-errors

      az storage blob upload \
        --account-name "$AZURE_STORAGE_ACCOUNT" \
        --container-name "$AZURE_CONTAINER_NAME" \
        --name "$APP_FILE" \
        --file "$APP_FILE" \
        --auth-mode login \
        --overwrite \
        --only-show-errors

      az storage blob upload \
        --account-name "$AZURE_STORAGE_ACCOUNT" \
        --container-name "$AZURE_CONTAINER_NAME" \
        --name "$CONFIG_FILE" \
        --file "$CONFIG_FILE" \
        --auth-mode login \
        --overwrite \
        --only-show-errors

    - |
      echo "Checking for existing VM Application..."
      if ! az sig gallery-application show \
          --resource-group "$AZURE_RESOURCE_GROUP" \
          --gallery-name "$GALLERY_NAME" \
          --application-name "$APPLICATION_NAME" &>/dev/null; then
        echo "Creating VM Application definition..."
        az sig gallery-application create \
          --resource-group "$AZURE_RESOURCE_GROUP" \
          --gallery-name "$GALLERY_NAME" \
          --application-name "$APPLICATION_NAME" \
          --location "$AZURE_LOCATION" \
          --os-type Windows \
          --only-show-errors
      else
        echo "VM Application already exists."
      fi

    - |
      echo "Generating SAS URLs and creating VM Application Version..."
      EXPIRY=$(date -u -d "+1 day" '+%Y-%m-%dT%H:%MZ')

      APP_SAS=$(az storage blob generate-sas \
        --account-name "$AZURE_STORAGE_ACCOUNT" \
        --container-name "$AZURE_CONTAINER_NAME" \
        --name "$APP_FILE" \
        --permissions r \
        --expiry "$EXPIRY" \
        --auth-mode login \
        --as-user \
        -o tsv)

      CONFIG_SAS=$(az storage blob generate-sas \
        --account-name "$AZURE_STORAGE_ACCOUNT" \
        --container-name "$AZURE_CONTAINER_NAME" \
        --name "$CONFIG_FILE" \
        --permissions r \
        --expiry "$EXPIRY" \
        --auth-mode login \
        --as-user \
        -o tsv)

      MAJOR=1
      MINOR=0
      PATCH=$(date +%Y%m%d)
      VERSION="$MAJOR.$MINOR.$PATCH"
      echo "Creating VM Application Version: $VERSION"

      INSTALL_CMD=$(jq -Rs '.' < install-script-as-string.txt)
      REMOVE_CMD=$(jq -Rs '.' < uninstall-script-as-string.txt)

      az sig gallery-application version create \
        --resource-group "$AZURE_RESOURCE_GROUP" \
        --gallery-name "$GALLERY_NAME" \
        --application-name "$APPLICATION_NAME" \
        --version-name "$VERSION" \
        --location "$AZURE_LOCATION" \
        --package-file-link "https://${AZURE_STORAGE_ACCOUNT}.blob.core.windows.net/${AZURE_CONTAINER_NAME}/${APP_FILE}?${APP_SAS}" \
        --default-configuration-file-link "https://${AZURE_STORAGE_ACCOUNT}.blob.core.windows.net/${AZURE_CONTAINER_NAME}/${CONFIG_FILE}?${CONFIG_SAS}" \
        --install-command "$INSTALL_CMD" \
        --remove-command "$REMOVE_CMD" \
        --only-show-errors
```

#### [Jenkins](#tab/jenkins4)
```
pipeline {
  agent any

  environment {
    APP_FILE = 'app.exe'
    CONFIG_FILE = 'app-config.json'

    AZURE_RESOURCE_GROUP = credentials('AZURE_RESOURCE_GROUP')
    AZURE_LOCATION = credentials('AZURE_LOCATION')
    AZURE_STORAGE_ACCOUNT = credentials('AZURE_STORAGE_ACCOUNT')
    AZURE_CONTAINER_NAME = credentials('AZURE_CONTAINER_NAME')
    GALLERY_NAME = credentials('GALLERY_NAME')
    APPLICATION_NAME = credentials('AZURE_VM_APPLICATION_NAME')

    AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
    AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET')
    AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
    AZURE_SUBSCRIPTION_ID = credentials('AZURE_SUBSCRIPTION_ID')
  }

  stages {
    stage('Login to Azure') {
      steps {
        sh '''
          az login --service-principal \
            -u "$AZURE_CLIENT_ID" \
            -p "$AZURE_CLIENT_SECRET" \
            --tenant "$AZURE_TENANT_ID"
          az account set --subscription "$AZURE_SUBSCRIPTION_ID"
        '''
      }
    }

    stage('Upload to Blob Storage') {
      steps {
        sh '''
          az storage container create \
            --name "$AZURE_CONTAINER_NAME" \
            --account-name "$AZURE_STORAGE_ACCOUNT" \
            --auth-mode login \
            --only-show-errors

          az storage blob upload \
            --account-name "$AZURE_STORAGE_ACCOUNT" \
            --container-name "$AZURE_CONTAINER_NAME" \
            --name "$APP_FILE" \
            --file "$APP_FILE" \
            --auth-mode login \
            --overwrite \
            --only-show-errors

          az storage blob upload \
            --account-name "$AZURE_STORAGE_ACCOUNT" \
            --container-name "$AZURE_CONTAINER_NAME" \
            --name "$CONFIG_FILE" \
            --file "$CONFIG_FILE" \
            --auth-mode login \
            --overwrite \
            --only-show-errors
        '''
      }
    }

    stage('Create VM Application if Needed') {
      steps {
        sh '''
          if ! az sig gallery-application show \
            --resource-group "$AZURE_RESOURCE_GROUP" \
            --gallery-name "$GALLERY_NAME" \
            --application-name "$APPLICATION_NAME" &>/dev/null; then
            az sig gallery-application create \
              --resource-group "$AZURE_RESOURCE_GROUP" \
              --gallery-name "$GALLERY_NAME" \
              --application-name "$APPLICATION_NAME" \
              --location "$AZURE_LOCATION" \
              --os-type Windows \
              --only-show-errors
          fi
        '''
      }
    }

    stage('Generate SAS and Create Version') {
      steps {
        sh '''
          EXPIRY=$(date -u -d "+1 day" '+%Y-%m-%dT%H:%MZ')

          APP_SAS=$(az storage blob generate-sas \
            --account-name "$AZURE_STORAGE_ACCOUNT" \
            --container-name "$AZURE_CONTAINER_NAME" \
            --name "$APP_FILE" \
            --permissions r \
            --expiry "$EXPIRY" \
            --auth-mode login \
            --as-user \
            -o tsv)

          CONFIG_SAS=$(az storage blob generate-sas \
            --account-name "$AZURE_STORAGE_ACCOUNT" \
            --container-name "$AZURE_CONTAINER_NAME" \
            --name "$CONFIG_FILE" \
            --permissions r \
            --expiry "$EXPIRY" \
            --auth-mode login \
            --as-user \
            -o tsv)

          MAJOR=1
          MINOR=0
          PATCH=$(date +%Y%m%d)
          VERSION="$MAJOR.$MINOR.$PATCH"

          INSTALL_CMD=$(jq -Rs '.' < install-script-as-string.txt)
          REMOVE_CMD=$(jq -Rs '.' < uninstall-script-as-string.txt)

          PACKAGE_URL="https://${AZURE_STORAGE_ACCOUNT}.blob.core.windows.net/${AZURE_CONTAINER_NAME}/${APP_FILE}?${APP_SAS}"
          CONFIG_URL="https://${AZURE_STORAGE_ACCOUNT}.blob.core.windows.net/${AZURE_CONTAINER_NAME}/${CONFIG_FILE}?${CONFIG_SAS}"

          az sig gallery-application version create \
            --resource-group "$AZURE_RESOURCE_GROUP" \
            --gallery-name "$GALLERY_NAME" \
            --application-name "$APPLICATION_NAME" \
            --version-name "$VERSION" \
            --location "$AZURE_LOCATION" \
            --package-file-link "$PACKAGE_URL" \
            --default-configuration-file-link "$CONFIG_URL" \
            --install-command "$INSTALL_CMD" \
            --remove-command "$REMOVE_CMD" \
            --only-show-errors
        '''
      }
    }
  }
}
```

----

## Deploy the VM Apps
One or more VM Applications can now be referenced in the `applicationProfile` of Azure VM or Azure Virtual Machine Scale Sets. Azure then pulls the payload of the VM Application and installs it on each VM using the provided install script. The `order` property defines the sequential order in which the VM Applications are installed on the VM. 

Refer [schema of applicationProfile of the VM / Virtual Machine Scale Sets](vm-applications.md#deploy-azure-vm-applications) to learn more about each property. 

#### [Template](#tab/Template5)

Use the following ARM template to deploy VM Application on Azure VM or Azure Virtual Machine Scale Sets. This template demonstrates the key properties and configuration options for deploying your VM application.

**Deploy VM Application on Virtual Machine Scale Sets**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmssName": {
      "type": "string"
    },
    "location": {
      "type": "string"
    },
    "subscriptionId": {
      "type": "string"
    },
    "resourceGroupName": {
      "type": "string"
    },
    "galleryName": {
      "type": "string"
    },
    "applicationName1": {
      "type": "string"
    },
    "applicationVersion1": {
      "type": "string",
      "defaultValue": "latest"
    },
    "configurationReference1": {
      "type": "string",
      "metadata": {
        "description": "Optional path to configuration file from Storage Account. Overrides default configuration file."
      }
    },
    "applicationName2": {
      "type": "string"
    },
    "applicationVersion2": {
      "type": "string",
      "defaultValue": "latest"
    }
  },
  "variables": {
    "packageReferenceId1": "[format('/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Compute/galleries/{2}/applications/{3}/versions/{4}', parameters('subscriptionId'), parameters('resourceGroupName'), parameters('galleryName'), parameters('applicationName1'), parameters('applicationVersion1'))]",
    "packageReferenceId2": "[format('/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Compute/galleries/{2}/applications/{3}/versions/{4}', parameters('subscriptionId'), parameters('resourceGroupName'), parameters('galleryName'), parameters('applicationName2'), parameters('applicationVersion2'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachineScaleSets",
      "apiVersion": "2024-03-03",
      "name": "[parameters('vmssName')]",
      "location": "[parameters('location')]",
      "properties": {
        "virtualMachineProfile": {
          "applicationProfile": {
            "galleryApplications": [
              {
                "order": 1,
                "packageReferenceId": "[variables('packageReferenceId1')]",
                "configurationReference": "[parameters('configurationReference1')]",
                "treatFailureAsDeploymentFailure": true
              },
              {
                "order": 2,
                "packageReferenceId": "[variables('packageReferenceId2')]",
                "treatFailureAsDeploymentFailure": false
              }
            ]
          }
        }
      }
    }
  ]
}
```

**Deploy VM Application on Azure VM**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string"
    },
    "location": {
      "type": "string"
    },
    "subscriptionId": {
      "type": "string"
    },
    "resourceGroupName": {
      "type": "string"
    },
    "galleryName": {
      "type": "string"
    },
    "applicationName1": {
      "type": "string"
    },
    "applicationVersion1": {
      "type": "string",
      "defaultValue": "latest"
    },
    "configurationReference1": {
      "type": "string",
      "metadata": {
        "description": "Optional path to configuration blob for application 1"
      }
    },
    "applicationName2": {
      "type": "string"
    },
    "applicationVersion2": {
      "type": "string",
      "defaultValue": "latest"
    }
  },
  "variables": {
    "packageReferenceId1": "[format('/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Compute/galleries/{2}/applications/{3}/versions/{4}', parameters('subscriptionId'), parameters('resourceGroupName'), parameters('galleryName'), parameters('applicationName1'), parameters('applicationVersion1'))]",
    "packageReferenceId2": "[format('/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Compute/galleries/{2}/applications/{3}/versions/{4}', parameters('subscriptionId'), parameters('resourceGroupName'), parameters('galleryName'), parameters('applicationName2'), parameters('applicationVersion2'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2024-07-01",
      "name": "[parameters('vmName')]",
      "location": "[parameters('location')]",
      "properties": {
        "applicationProfile": {
          "galleryApplications": [
            {
              "order": 1,
              "packageReferenceId": "[variables('packageReferenceId1')]",
              "configurationReference": "[parameters('configurationReference1')]",
              "treatFailureAsDeploymentFailure": true
            },
            {
              "order": 2,
              "packageReferenceId": "[variables('packageReferenceId2')]",
              "treatFailureAsDeploymentFailure": false
            }
          ]
        }
      }
    }
  ]
}
```

#### [REST](#tab/rest5)

To add a VM application version to a VM, perform a PUT on the VM.

```rest
PUT
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VMName}?api-version=2024-03-03

{
  "properties": {
    "applicationProfile": {
      "galleryApplications": [
        {
          "order": 1,
          "packageReferenceId": "/subscriptions/{subscriptionId}/resourceGroups/<resource group>/providers/Microsoft.Compute/galleries/{gallery name}/applications/{application name}/versions/{version | latest}",
          "configurationReference": "{path to configuration storage blob}",
          "treatFailureAsDeploymentFailure": false
        }
      ]
    }
  },
  "name": "{vm name}",
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resource group}/providers/Microsoft.Compute/virtualMachines/{vm name}",
  "location": "{vm location}"
}
```


To apply the VM application to a uniform Virtual Machine Scale Sets:

```rest
PUT
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{VMSSName}?api-version=2024-03-03

{
  "properties": {
    "virtualMachineProfile": {
      "applicationProfile": {
        "galleryApplications": [
          {
            "order": 1,
            "packageReferenceId": "/subscriptions/{subscriptionId}/resourceGroups/<resource group>/providers/Microsoft.Compute/galleries/{gallery name}/applications/{application name}/versions/{version | latest}",
            "configurationReference": "{path to configuration storage blob}",
            "treatFailureAsDeploymentFailure": false
          }
        ]
      }
    }
  },
  "name": "{vm name}",
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resource group}/providers/Microsoft.Compute/virtualMachines/{vm name}",
  "location": "{vm location}"
}
```

The response includes the full VM model. The following are the
relevant parts.

```rest
{
  "name": "{vm name}",
  "id": "{vm id}",
  "type": "Microsoft.Compute/virtualMachines",
  "location": "{vm location}",
  "properties": {
    "applicationProfile": {
      "galleryApplications": ""
    },
    "provisioningState": "Updating"
  },
  "resources": [
    {
      "name": "VMAppExtension",
      "id": "{extension id}",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "location": "centraluseuap",
      "properties": "@{autoUpgradeMinorVersion=True; forceUpdateTag=7c4223fc-f4ea-4179-ada8-c8a85a1399f5; provisioningState=Creating; publisher=Microsoft.CPlat.Core; type=VMApplicationManagerLinux; typeHandlerVersion=1.0; settings=}"
    }
  ]
}

```

#### [CLI](#tab/cli5)
Set a VM application to an existing VM using ['az vm application set'](/cli/azure/vm/application#az-vm-application-set) and replace the values of the parameters with your own.

```azurecli-interactive
az vm application set \
	--resource-group myResourceGroup \
	--name myVM \
  	--app-version-ids /subscriptions/{subID}/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp/versions/1.0.0 \
  	--treat-deployment-as-failure true
```
For setting multiple applications on a VM:

```azurecli-interactive
az vm application set \
	--resource-group myResourceGroup \
	--name myVM \
	--app-version-ids /subscriptions/{subId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp/versions/1.0.0 /subscriptions/{subId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp2/versions/1.0.1 \
	--treat-deployment-as-failure true true
```
To add an application to a Virtual Machine Scale Sets, use ['az vmss application set'](/cli/azure/vmss/application#az-vmss-application-set):

```azurecli-interactive
az vmss application set \
	--resource-group myResourceGroup \
	--name myVmss \
	--app-version-ids /subscriptions/{subId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp/versions/1.0.0 \
	--treat-deployment-as-failure true
```
To add multiple applications to a Virtual Machine Scale Sets:
```azurecli-interactive
az vmss application set \
	--resource-group myResourceGroup \
	--name myVmss
	--app-version-ids /subscriptions/{subId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp/versions/1.0.0 /subscriptions/{subId}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/galleries/myGallery/applications/myApp2/versions/1.0.0 \
	--treat-deployment-as-failure true
```

#### [PowerShell](#tab/powershell5)

To add the application to an existing VM, get the application version and use that to get the VM application version ID. Create the application object using ID with ['New-AzVmGalleryApplication'](/powershell/module/az.compute/new-azvmgalleryapplication) and add the application to the VM configuration using ['Add-AzVmGalleryApplication'](/powershell/module/az.compute/add-azvmgalleryapplication).

```azurepowershell-interactive
$galleryName = "myGallery"
$rgName = "myResourceGroup"
$applicationName = "myApp"
$version = "1.0.0"
$vmName = "myVM"

$vm = Get-AzVM -ResourceGroupName $rgname -Name $vmName
$appVersion = Get-AzGalleryApplicationVersion `
   -GalleryApplicationName $applicationName `
   -GalleryName $galleryName `
   -Name $version `
   -ResourceGroupName $rgName
$packageId = $appVersion.Id
$app = New-AzVmGalleryApplication -PackageReferenceId $packageId
Add-AzVmGalleryApplication -VM $vm -GalleryApplication $app -TreatFailureAsDeploymentFailure true
Update-AzVM -ResourceGroupName $rgName -VM $vm
```

To add the application to a Virtual Machine Scale Sets:
```azurepowershell-interactive
$galleryName = "myGallery"
$rgName = "myResourceGroup"
$applicationName = "myApp"
$version = "1.0.0"
$vmssName = "myVMSS"

$vmss = Get-AzVmss -ResourceGroupName $rgname -Name $vmssName
$appVersion = Get-AzGalleryApplicationVersion `
   -GalleryApplicationName $applicationName `
   -GalleryName $galleryName `
   -Name $version `
   -ResourceGroupName $rgName
$packageId = $appVersion.Id
$app = New-AzVmssGalleryApplication -PackageReferenceId $packageId
Add-AzVmssGalleryApplication -VirtualMachineScaleSetVM $vmss.VirtualMachineProfile -GalleryApplication $app
Update-AzVmss -ResourceGroupName $rgName -VirtualMachineScaleSet $vmss -VMScaleSetName $vmssName
```

#### [Portal](#tab/portal5)

Now you can create a VM and deploy the VM application to it using the portal. Just create the VM as usual, and under the **Advanced** tab, choose **Select a VM application to install**.

:::image type="content" source="media/vmapps/advanced-tab.png" alt-text="Screenshot of the Advanced tab where you can choose to install a VM application.":::

Select the VM application from the list and then select **Save** at the bottom of the page.

:::image type="content" source="media/vmapps/select-app.png" alt-text="Screenshot showing selecting a VM application to install on the VM.":::

If you have more than one VM application to install, you can set the install order for each VM application back on the **Advanced tab**.

You can also deploy the VM application to currently running VMs. Select the **Extensions + applications** option under **Settings** in the left menu when viewing the VM details in the portal.

Choose **VM applications** and then select **Add application** to add your VM application.

:::image type="content" source="media/vmapps/select-extension-app.png" alt-text="Screenshot showing selecting a VM application to install on a currently running VM.":::

Select the VM application from the list and then select **Save** at the bottom of the page.

:::image type="content" source="media/vmapps/select-app.png" alt-text="Screenshot showing selecting a VM application to install on the VM.":::

----


## Next steps
- Learn to [manage, update, or delete](vm-applications-manage.md) Azure VM Applications.
- Learn more about [Azure VM Applications](vm-applications.md).
