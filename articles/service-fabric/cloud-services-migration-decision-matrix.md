---
title: Azure Cloud Services migration decision matrix 
description: This article provides a decision matrix to help evaluate migration options for an Azure solution currently built on Cloud Services.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 06/26/2025
# Customer intent: As a cloud architect currently using Cloud Services, I want to evaluate my options to migrate my current Cloud Services solution to other Azure services, so that I can make the right choice for my architecture.
---

# Azure Cloud Services migration decision matrix

This article provides a decision matrix to help evaluate migration options for an Azure solution currently built on Cloud Services. The matrix compares various Azure platform technologies to help guide your rearchitecting decisions.

## Decision matrix

| **Criteria / Option**          | **Service Fabric**                                        | **Azure Functions**                                           | **Azure Web Apps**                              | **Azure App Service (Container Apps, Logic Apps, etc.)** | **Azure Kubernetes Service (AKS)**                         | **Virtual Machine Scale Sets**                              | **Azure Virtual Machines**                                  |
|--------------------------------|-----------------------------------------------------------|---------------------------------------------------------------|------------------------------------------------|--------------------------------------------------------|------------------------------------------------------------|------------------------------------------------------------|-------------------------------------------------------------|
| **Migration Complexity**       | Moderate-to-high. Requires rearchitecting roles to stateful/stateless services. Often a learning curve for state management and orchestration. | Low-to-moderate. Best suited for event-driven, lightweight tasks. Significant rework if the app isn't designed for serverless. | Low-to-moderate. Often similar to current web role structure. Minimal rework required. | Low-to-moderate. Migration complexity varies by service but well-supported with migration tools. | High. Containerizing your app requires a deep understanding of microservices and orchestration. | Moderate. Closer to Cloud Services model, offering easier lift-and-shift with autoscaling capabilities. | High. Lift-and-shift may be simpler, but it doesn't leverage cloud-native benefits. |
| **Scalability & Performance**  | High. Designed for complex microservices and can scale stateful/stateless workloads effectively. | High. Automatic scaling based on events, but can be subject to cold starts. | Moderate-to-high. Built-in scaling features, but may need manual configuration for advanced scenarios. | High. Automated scaling with various triggers (HTTP requests, event-based, schedule). | Very high. Container orchestration allows fine-grained scaling and resilience. | High. Automatic scaling based on metrics, with predictive autoscaling capabilities. | Limited. Scalability depends on manual scaling and VM size/configuration. |
| **Control & Customization**    | High. Full control over service orchestration and state management. | Limited. Managed service with constrained runtime environment and execution limits. | Moderate. Managed platform with some configuration options; less control over underlying infrastructure. | Moderate. More flexibility than Web Apps with container support, but still PaaS constraints. | High. Maximum control over container orchestration and runtime environment. | High. Control over VM configuration, networking, and load balancing, with autoscaling policies. | Very high. Complete control over the OS, middleware, and runtime. |
| **Operational Overhead**       | Moderate. Requires management of cluster health, upgrades, and stateful services. | Low. Managed service abstracts infrastructure, reducing operational overhead. | Low. Microsoft manages the platform, though some app-specific issues remain. | Low. Managed service with minimal infrastructure management required. | High. Requires in-depth operational expertise (monitoring, updates, networking, etc.). | Moderate. Handles instance scaling automatically but requires VM image maintenance and updates. | High. Full responsibility for maintenance, patching, and security. |
| **Ecosystem Integration**      | Strong. Integrates well with Azure's service-oriented ecosystem, but may require further configuration. | Strong. Natively integrated with Azure's Event Grid, logic apps, and other serverless services. | Strong. First-class integration with CI/CD, monitoring, and other Azure services. | Very strong. Designed to work together with other Azure services with minimal configuration. | Strong. Works well with modern DevOps tools, though integration complexity increases with microservices. | Strong. Integrates with Azure Load Balancer, Application Gateway, and Azure Monitor. | Variable. While integration is possible, it often requires more custom development. |
| **Use Case Suitability**       | Ideal for applications needing fine-grained control over microservices, including stateful services. | Best for event-driven workloads, batch processing, and where latency is acceptable. | Well-suited for web apps and APIs with a standard request/response model. | Good for mixed workloads combining web, mobile backends, integration logic, and API scenarios. | Excellent for microservices and containerized workloads that require high availability and scalability. | Well-suited for stateless applications with variable load patterns and IaaS requirements. | Good for legacy applications that require minimal changes, but less cloud-optimized. |

---

## Additional considerations

- **Legacy vs. Re-architected:**  
  If you're looking to modernize your architecture beyond a lift-and-shift, then options like Service Fabric, AKS, or even a serverless approach with Functions might provide better long-term benefits despite higher initial rearchitecting costs.

- **Skill Set and Team Expertise:**  
  Evaluate your team's familiarity with microservices (Service Fabric, AKS) versus serverless or PaaS (Web Apps, Functions). Training and adoption curves could influence your decision.

- **Cost Implications:**  
  Managed services (Functions, Web Apps) may reduce operational costs compared to managing your own container clusters (AKS) or VMs, but the pricing models (consumption vs. reserved instances) can vary significantly.

- **Future-Proofing:**  
  Consider how each option aligns with long-term product strategy. For example, AKS and containerized architectures are gaining popularity for their portability and ease of integration with CI/CD pipelines.

- **Performance and Latency Requirements:**  
  Some workloads might benefit from the granular control offered by Service Fabric or AKS, while others might be well served by the ease-of-use and autoscaling features of Functions or Web Apps.

---

## Next steps

**Assess your workloads:**  
Identify which components of your solution are best suited for each architecture—consider both compute-bound and stateful components.

**Prototype migration:**  
   Consider building a proof-of-concept for the top two or three options. This can help uncover unforeseen challenges and better estimate migration complexity.

**Cost-benefit analysis:**  
Include not only migration effort and operational overhead but also long-term benefits such as ease of updates, resilience, and scalability.

**Stakeholder review:**  
   Share the decision matrix with your team and other stakeholders for feedback and to align on priorities.

**Review migration guides:**  
  - [Comprehensive migration guide](cloud-services-to-service-fabric-migration-guide.md) - Detailed steps and best practices for migrating from Azure Cloud Services to Service Fabric
  - [Web Role migration example](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/MigrationGuides/WebRole_Migration_Example.md) - Complete example of migrating an ASP.NET Web Role to a Service Fabric Stateless Service
  - [Worker Role migration example](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/MigrationGuides/WorkerRole_Migration_Example.md) - Detailed guide for transforming Worker Role background processing to Service Fabric
  - [State Management migration example](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/MigrationGuides/StateManagement_Migration_Example.md) - Technical guide for migrating application state management to Service Fabric
