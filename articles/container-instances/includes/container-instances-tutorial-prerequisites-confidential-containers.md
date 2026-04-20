---
services: container-instances
author: tomvcassidy
ms.service: azure-container-instances
ms.topic: include
ms.date: 11/17/2025
ms.author: tomcassidy
# Customer intent: "As a cloud developer, I want to install and configure Azure CLI and Docker on my local machine, so that I can complete the tutorial for deploying containerized applications using confidential computing policies."
---

To complete this tutorial, you must satisfy the following requirements:

* **Azure CLI**: You must have Azure CLI version 2.44.1 or later installed on your local computer. To find your version, run `az --version`. If you need to install or upgrade, see [Install the Azure CLI](/cli/azure/install-azure-cli).

* **Azure CLI confcom extension**: You must have Azure CLI confcom extension version 0.30+ installed to generate confidential computing enforcement policies.

  ```bash
   az extension add -n confcom
  ```

* **Docker**: You need Docker installed locally. Docker provides packages that configure the Docker environment on [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/), and [Linux](https://docs.docker.com/engine/installation/#supported-platforms).

  This tutorial assumes a basic understanding of core Docker concepts like containers, container images, and basic `docker` commands. For a primer on Docker and container basics, see the [Docker overview](https://docs.docker.com/engine/docker-overview/).

> [!IMPORTANT]
> Because Azure Cloud Shell doesn't include the Docker daemon, you must install both the Azure CLI and Docker Engine on your *local computer* to complete this tutorial. You can't use Azure Cloud Shell for this tutorial.
