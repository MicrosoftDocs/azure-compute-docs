---
author: tomvcassidy
services: container-instances
ms.service: azure-container-instances
ms.topic: include
ms.date: 11/17/2025
ms.author: tomcassidy
# Customer intent: "As a cloud architect, I want to understand the maximum resource limits for GPU SKUs in container instances so that I can optimize resource allocation for my applications."
---
### Maximum resources per SKU

| OS | GPU SKU | GPU count | Max CPU | Max memory (GB) | Storage (GB) |
| --- | --- | --- | --- | --- | --- |
| Linux | V100 | 1 | 6 | 112 | 50 |
| Linux | V100 | 2 | 12 | 224 | 50 |
| Linux | V100 | 4 | 24 | 448 | 50 |
