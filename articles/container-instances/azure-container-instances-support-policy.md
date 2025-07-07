---
title: Support Policy for Azure Container Instances
description: Outline of the support policies on Azure Container Instances
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.topic: conceptual
ms.date: 04/16/2025
---
# Support policy for Azure Container Instances

This article explains the support policies for **Azure Container Instances (ACI)**.  
It explains how Microsoft manages the container runtime and infrastructure, clarifies what customers are responsible for, and defines what is and isn't supported for networking, third-party software, and security or patch management in a serverless setup.

## Azure Container Instances (ACI) Overview

Here's a summary of ACI:

- Platform-as-a-Service (PaaS): ACI abstracts away infrastructure management. You only provide the container image and resource definitions—ACI handles provisioning, scaling, and securing the runtime environment.

- No Host-Level Responsibility: There’s no shared responsibility model for the host. You don’t need to patch operating systems, manage VMs, or configure load balancers.

- Ideal Use Cases: ACI is best suited for lightweight, event-driven, or short-lived workloads that benefit from fast startup and minimal operational overhead.

- Limited Customization by Design: While you can configure container group properties (for example, CPU, memory, networking), you cannot modify the host OS or runtime. This ensures consistency, security, and high availability.

Not a Full Orchestrator: ACI doesn’t offer the orchestration features of AKS (Azure Kubernetes Service), but it excels at rapid, scalable container deployment without the complexity of managing clusters.

## Managed features in ACI

Azure Container Instances (ACI) provide a simplified, serverless environment for running containers without the need to manage any underlying virtual machines or orchestrators. Unlike traditional IaaS where you manage compute, networking, and storage, ACI offers a streamlined experience with minimal infrastructure management.

With ACI, you can deploy containers in seconds using a fully managed platform. Microsoft manages the runtime and compute, so you don't need to handle VMs, scaling, or Kubernetes setup.

Microsoft manages these parts of ACI:

- Container runtime (such as Docker-compatible runtimes)
- Compute and networking infrastructure necessary to run your containers
- Storage integration (when attached) such as Azure File shares
- Container scheduling and isolation to ensure resource fairness and security
- DNS resolution and automatic public IP assignment for container groups
- Security patches and updates for the underlying OS and runtime

## Shared responsibility
When you deploy containers using Azure Container Instances (ACI), Microsoft handles the underlying infrastructure—including compute, networking, and the operating system. Your responsibility is limited to managing the container image, the application inside it, and the data it processes.

Because ACI abstracts away the virtual machine layer, Microsoft Support cannot access your container’s file system, logs, or runtime data unless you explicitly configure diagnostics (for example, sending logs to Azure Monitor or a storage account). You retain full control over the container image and its behavior.

Since you don’t manage the host environment, simulating host-level changes (as you might in a VM or Kubernetes cluster) is not supported. Instead, all configuration should be done within the container image or through supported environment variables and ACI resource definitions.

While you can apply custom tags and metadata to the ACI resource, modifying system-managed settings or trying to bypass platform constraints may lead to unsupported or unstable states.
