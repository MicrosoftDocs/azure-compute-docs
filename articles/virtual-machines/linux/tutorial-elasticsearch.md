---
title: Deploy ElasticSearch on a development virtual machine in Azure
description: Install the Elastic Stack (ELK) onto a development Linux VM in Azure
services: virtual-machines
author: rloutlaw
manager: justhe
ms.service: azure-virtual-machines
ms.collection: linux
ms.devlang: azurecli
ms.custom: devx-track-azurecli, linux-related-content, innovation-engine
ms.topic: how-to
ms.date: 04/06/2025
ms.author: routlaw
# Customer intent: As a developer, I want to deploy the Elastic Stack on a development virtual machine in Azure so that I can test and visualize logging data effectively.
---

# Install the Elastic Stack (ELK) on an Azure VM

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321850)

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

This article walks you through how to deploy [Elasticsearch](https://www.elastic.co/products/elasticsearch), [Logstash](https://www.elastic.co/products/logstash), and [Kibana](https://www.elastic.co/products/kibana), on an Ubuntu VM in Azure. To see the Elastic Stack in action, you can optionally connect to Kibana  and work with some sample logging data. 

Additionally, you can follow the [Deploy Elastic on Azure Virtual Machines](/training/modules/deploy-elastic-azure-virtual-machines/) module for a more guided tutorial on deploying Elastic on Azure Virtual Machines.   

In this tutorial you learn how to:

> [!div class="checklist"]
> * Create an Ubuntu VM in an Azure resource group
> * Install Elasticsearch, Logstash, and Kibana on the VM
> * Send sample data to Elasticsearch with Logstash 
> * Open ports and work with data in the Kibana console

This deployment is suitable for basic development with the Elastic Stack. For more on the Elastic Stack, including recommendations for a production environment, see the [Elastic documentation](https://www.elastic.co/guide/index.html) and the [Azure Architecture Center](/azure/architecture/elasticsearch/).

[!INCLUDE [azure-cli-prepare-your-environment.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

- This article requires version 2.0.4 or later of the Azure CLI. If using Azure Cloud Shell, the latest version is already installed.

## Create a resource group

In this section, environment variables are declared for use in subsequent commands. A random suffix is appended to resource names for uniqueness.

```bash
export RANDOM_SUFFIX=$(openssl rand -hex 3)
export RESOURCE_GROUP="myResourceGroup$RANDOM_SUFFIX"
export REGION="eastus2"
az group create --name $RESOURCE_GROUP --location $REGION
```

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "id": "/subscriptions/xxxxx/resourceGroups/myResourceGroupxxxxxx",
  "location": "eastus",
  "managedBy": null,
  "name": "myResourceGroupxxxxxx",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create a virtual machine

This section creates a VM with a unique name, while also generating SSH keys if they do not already exist. A random suffix is appended to ensure uniqueness.

```bash
export VM_NAME="myVM$RANDOM_SUFFIX"
az vm create \
    --resource-group $RESOURCE_GROUP \
    --name $VM_NAME \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --generate-ssh-keys
```

When the VM has been created, the Azure CLI shows information similar to the following example. Take note of the publicIpAddress. This address is used to access the VM.

Results:

<!-- expected_similarity=0.3 -->
```JSON
{
  "fqdns": "",
  "id": "/subscriptions/xxxxx/resourceGroups/myResourceGroupxxxxxx/providers/Microsoft.Compute/virtualMachines/myVMxxxxxx",
  "location": "eastus",
  "macAddress": "xx:xx:xx:xx:xx:xx",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "x.x.x.x",
  "resourceGroup": "$RESOURCE_GROUP"
}
```

## SSH into your VM

If you don't already know the public IP address of your VM, run the following command to list it:

```azurecli-interactive
az network public-ip list --resource-group $RESOURCE_GROUP --query [].ipAddress
```

Use the following command to create an SSH session with the virtual machine. Substitute the correct public IP address of your virtual machine. In this example, the IP address is *40.68.254.142*.

```bash
export PUBLIC_IP_ADDRESS=$(az network public-ip list --resource-group $RESOURCE_GROUP --query [].ipAddress -o tsv)
```

## Install the Elastic Stack

In this section, you import the Elasticsearch signing key and update your APT sources list to include the Elastic package repository. This is followed by installing the Java runtime environment which is required for the Elastic Stack components.

```bash
ssh azureuser@$PUBLIC_IP_ADDRESS -o StrictHostKeyChecking=no "
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
"
```

Install the Java Virtual Machine on the VM and configure the JAVA_HOME variable:

```bash
ssh azureuser@$PUBLIC_IP_ADDRESS -o StrictHostKeyChecking=no "
sudo apt install -y openjdk-8-jre-headless
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
"
```

Run the following command to update Ubuntu package sources and install Elasticsearch, Kibana, and Logstash.

```bash
ssh azureuser@$PUBLIC_IP_ADDRESS -o StrictHostKeyChecking=no "
  wget -qO elasticsearch.gpg https://artifacts.elastic.co/GPG-KEY-elasticsearch
  sudo mv elasticsearch.gpg /etc/apt/trusted.gpg.d/

  echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list

  sudo apt update
  
  # Now install the ELK stack
  sudo apt install -y elasticsearch kibana logstash
"
```

> [!NOTE]
> Detailed installation instructions, including directory layouts and initial configuration, are maintained in [Elastic's documentation](https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html)

## Start Elasticsearch

Start Elasticsearch on your VM with the following command:

```bash
ssh azureuser@$PUBLIC_IP_ADDRESS -o StrictHostKeyChecking=no "
sudo systemctl start elasticsearch.service
"
```

This command produces no output, so verify that Elasticsearch is running on the VM with this curl command:

```bash
ssh azureuser@$PUBLIC_IP_ADDRESS -o StrictHostKeyChecking=no "
sleep 11
sudo curl -XGET 'localhost:9200/'
"
```

If Elasticsearch is running, you see output like the following:

Results:

<!-- expected_similarity=0.3 -->
```json
{
  "name" : "w6Z4NwR",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "SDzCajBoSK2EkXmHvJVaDQ",
  "version" : {
    "number" : "5.6.3",
    "build_hash" : "1a2f265",
    "build_date" : "2017-10-06T20:33:39.012Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

## Start Logstash and add data to Elasticsearch

Start Logstash with the following command:

```bash 
ssh azureuser@$PUBLIC_IP_ADDRESS -o StrictHostKeyChecking=no "
sudo systemctl start logstash.service
"
```

Test Logstash to make sure it's working correctly:

```bash
ssh azureuser@$PUBLIC_IP_ADDRESS -o StrictHostKeyChecking=no "
# Time-limited test with file input instead of stdin
sudo timeout 11s /usr/share/logstash/bin/logstash -e 'input { file { path => "/var/log/syslog" start_position => "end" sincedb_path => "/dev/null" stat_interval => "1 second" } } output { stdout { codec => json } }' || echo "Logstash test completed"
"
```

This is a basic Logstash [pipeline](https://www.elastic.co/guide/en/logstash/5.6/pipeline.html) that echoes standard input to standard output.

Set up Logstash to forward the kernel messages from this VM to Elasticsearch. To create the Logstash configuration file, run the following command which writes the configuration to a new file called vm-syslog-logstash.conf:

```bash
ssh azureuser@$PUBLIC_IP_ADDRESS -o StrictHostKeyChecking=no "
cat << 'EOF' > vm-syslog-logstash.conf
input {
    stdin {
        type => "stdin-type"
    }

    file {
        type => "syslog"
        path => [ "/var/log/*.log", "/var/log/*/*.log", "/var/log/messages", "/var/log/syslog" ]
        start_position => "beginning"
    }
}

output {

    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts  => "localhost:9200"
    }
}
EOF
"
```

Test this configuration and send the syslog data to Elasticsearch:

```bash
# Run Logstash with the configuration for 60 seconds
sudo timeout 60s /usr/share/logstash/bin/logstash -f vm-syslog-logstash.conf &
LOGSTASH_PID=$!

# Wait for data to be processed
echo "Processing logs for 60 seconds..."
sleep 65

# Verify data was sent to Elasticsearch with proper error handling
echo "Verifying data in Elasticsearch..."
ES_COUNT=$(sudo curl -s -XGET 'localhost:9200/_cat/count?v' | tail -n 1 | awk '{print $3}' 2>/dev/null || echo "0")

# Make sure ES_COUNT is a number or default to 0
if ! [[ "$ES_COUNT" =~ ^[0-9]+$ ]]; then
    ES_COUNT=0
    echo "Warning: Could not get valid document count from Elasticsearch"
fi

echo "Found $ES_COUNT documents in Elasticsearch"

if [ "$ES_COUNT" -gt 0 ]; then
    echo "✅ Logstash successfully sent data to Elasticsearch"
else
    echo "❌ No data found in Elasticsearch, there might be an issue with Logstash configuration"
fi
```

You see the syslog entries in your terminal echoed as they are sent to Elasticsearch. Use CTRL+C to exit out of Logstash once you've sent some data.

## Start Kibana and visualize the data in Elasticsearch

Edit the Kibana configuration file (/etc/kibana/kibana.yml) and change the IP address Kibana listens on so you can access it from your web browser:

```text
server.host: "0.0.0.0"
```

Start Kibana with the following command:

```bash
ssh azureuser@$PUBLIC_IP_ADDRESS -o StrictHostKeyChecking=no "
sudo systemctl start kibana.service
"
```

Open port 5601 from the Azure CLI to allow remote access to the Kibana console:

```azurecli-interactive
az vm open-port --port 5601 --resource-group $RESOURCE_GROUP --name $VM_NAME
```

## Next steps

In this tutorial, you deployed the Elastic Stack into a development VM in Azure. You learned how to:

> [!div class="checklist"]
> * Create an Ubuntu VM in an Azure resource group
> * Install Elasticsearch, Logstash, and Kibana on the VM
> * Send sample data to Elasticsearch from Logstash 
> * Open ports and work with data in the Kibana console