---
services: container-instances
author: tomvcassidy
ms.service: azure-container-instances
ms.topic: include
ms.date: 08/29/2024
ms.author: tomcassidy
# Customer intent: "As a developer, I want to install Azure CLI and Docker locally, so that I can complete the tutorial and effectively work with container instances."
---

You must satisfy the following requirements to complete this tutorial:

**Azure CLI**: You must have Azure CLI version 2.0.29 or later installed on your local computer. To find the version, run `az --version`. If you need to install or upgrade, see [Install the Azure CLI][azure-cli-install].

**Docker**: This tutorial assumes a basic understanding of core Docker concepts like containers, container images, and basic `docker` commands. For a primer on Docker and container basics, see the [Docker overview][docker-get-started].

**Docker**: To complete this tutorial, you need Docker installed locally. Docker provides packages that configure the Docker environment on [macOS][docker-mac], [Windows][docker-windows], and [Linux][docker-linux].

> [!IMPORTANT]
> Because the Azure Cloud shell does not include the Docker daemon, you *must* install both the Azure CLI and Docker Engine on your *local computer* to complete this tutorial. You cannot use the Azure Cloud Shell for this tutorial.

<!-- LINKS - External -->
[docker-get-started]: https://docs.docker.com/engine/docker-overview/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/desktop/install/mac-install/
[docker-windows]: https://docs.docker.com/desktop/install/windows-install/

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
