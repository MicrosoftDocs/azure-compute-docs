---
title: Containerize your Azure Service Fabric services on Windows
description: Learn how to containerize your Service Fabric Reliable Services and Reliable Actors services on Windows.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 07/11/2022
# Customer intent: As a developer, I want to containerize my Service Fabric applications on Windows, so that I can deploy and manage them efficiently within a containerized environment.
---

# Containerize your Service Fabric Reliable Services and Reliable Actors on Windows

Service Fabric supports containerizing Service Fabric microservices (Reliable Services, and Reliable Actor based services). For more information, see [service fabric containers](service-fabric-containers-overview.md).

This document provides guidance to get your service running inside a Windows container.

> [!NOTE]
> Currently this feature only works for Windows. To run containers, the cluster must be running on Windows Server with Docker installed. Refer to [Installing Mirantis Runtime](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/Deployment/Mirantis-Installation.md).

## Steps to containerize your Service Fabric Application

1. Open your Service Fabric application in Visual Studio.

2. Add class [SFBinaryLoader.cs](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/code/SFBinaryLoaderForContainers/SFBinaryLoader.cs) to your project. The code in this class is a helper to correctly load the Service Fabric runtime binaries inside your application when running inside a container.

3. For each code package you would like to containerize, initialize the loader at the program entry point. Add the static constructor shown in the following code snippet to your program entry point file.

   ```csharp
   namespace MyApplication
   {
      internal static class Program
      {
          static Program()
          {
              SFBinaryLoader.Initialize();
          }

          /// <summary>
          /// This is the entry point of the service host process.
          /// </summary>
          private static void Main()
          {
   ```

4. Build and [package](service-fabric-package-apps.md#Package-App) your project. To build and create a package, right-click the application project in Solution Explorer and choose the **Package** command.

5. For every code package you need to containerize, run the PowerShell script [CreateDockerPackage.ps1](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/scripts/CodePackageToDockerPackage/CreateDockerPackage.ps1). The usage is as follows:

    Full .NET

      ```powershell
        $codePackagePath = 'Path to the code package to containerize.'
        $dockerPackageOutputDirectoryPath = 'Output path for the generated docker folder.'
        $applicationExeName = 'Name of the Code package executable.'
        CreateDockerPackage.ps1 -CodePackageDirectoryPath $codePackagePath -DockerPackageOutputDirectoryPath $dockerPackageOutputDirectoryPath -ApplicationExeName $applicationExeName
      ```

    .NET Core

      ```powershell
        $codePackagePath = 'Path to the code package to containerize.'
        $dockerPackageOutputDirectoryPath = 'Output path for the generated docker folder.'
        $dotnetCoreDllName = 'Name of the Code package dotnet Core Dll.'
        CreateDockerPackage.ps1 -CodePackageDirectoryPath $codePackagePath -DockerPackageOutputDirectoryPath $dockerPackageOutputDirectoryPath -DotnetCoreDllName $dotnetCoreDllName
      ```

      The script creates a folder with Docker artifacts at $dockerPackageOutputDirectoryPath. Modify the generated Dockerfile to `expose` any ports, run setup scripts and so on. based on your needs.

6. Next you need to [build](service-fabric-get-started-containers.md#Build-Containers) and [push](service-fabric-get-started-containers.md#Push-Containers) your Docker container package to your repository.

7. Modify the ApplicationManifest.xml and ServiceManifest.xml to add your container image, repository information, registry authentication, and port-to-host mapping. For modifying the manifests, see [Create an Azure Service Fabric container application](service-fabric-get-started-containers.md). The code package definition in the service manifest needs to be replaced with corresponding container image. Make sure to change the EntryPoint to a ContainerHost type.

   ```xml
   <!-- Code package is your service executable. -->
   <CodePackage Name="Code" Version="1.0.0">
   <EntryPoint>
    <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
    <ContainerHost>
      <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
    </ContainerHost>
   </EntryPoint>
   <!-- Pass environment variables to your container: -->
   </CodePackage>
   ```

8. Add the port-to-host mapping for your replicator and service endpoint. Since both these ports are assigned at runtime by Service Fabric, the ContainerPort is set to zero to use the assigned port for mapping.

   ```xml
   <Policies>
   <ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="0" EndpointRef="ServiceEndpoint"/>
    <PortBinding ContainerPort="0" EndpointRef="ReplicatorEndpoint"/>
   </ContainerHostPolicies>
   </Policies>
   ```

9. For configuring container isolation mode, see [Configure isolation mode]( ./service-fabric-get-started-containers.md#configure-isolation-mode). Windows supports two isolation modes for containers: process and Hyper-V. The following snippets show how the isolation mode is specified in the application manifest file.

   ```xml
   <Policies>
   <ContainerHostPolicies CodePackageRef="Code" Isolation="process">
   ...
   </ContainerHostPolicies>
   </Policies>
   ```

   ```xml
   <Policies>
   <ContainerHostPolicies CodePackageRef="Code" Isolation="hyperv">
   ...
   </ContainerHostPolicies>
   </Policies>
   ```

10. (Optional) Configure certificate for manual communication between services. By default, the primary cluster certificate is needed for manual communication between different containerized services. This certificate is not installed inside the containers like it is on the node itself. One way to fix this is to include the .pfx file inside the container along with a PowerShell script to install the certificate inside the container. Another way would be to configure the cluster such that the certificate is not required. Learn more about [Service Fabric cluster security scenarios](service-fabric-cluster-security.md).

11. (Optional) Configure Ktl logger to use user mode. Stateful services can experience an error when running in containers because of the differences in the file system.  This error will be visible inside the Service Fabric Explorer as follows:

    ```bash
    System.Runtime.InteropServices.COMException (-2147024463)
    A device which does not exist was specified. (0x800701B1)
    ...
    ```

    The error can be fixed by updating the cluster manifest to set the **UseUserModeKtlLogger** to **true** under the **TransactionalReplicator** section. Follow the steps in this [article](service-fabric-cluster-fabric-settings.md).

    ```json
      {
       "name": "TransactionalReplicator",
       "parameters": [
         {
           "name": "UseUserModeKtlLogger",
           "value": "true"
         }
       ]
      }
    ```

12. To test this application, you need to deploy it to a cluster that is running version 5.7 or higher. For runtime versions 6.1 or lower, you need to edit and update the cluster settings to enable this preview feature. Follow the steps in [this article](service-fabric-cluster-fabric-settings.md) to add the setting shown next.

    ```json
      {
        "name": "Hosting",
        "parameters": [
          {
            "name": "FabricContainerAppsEnabled",
            "value": "true"
          }
        ]
      }
    ```

13. Next [deploy](service-fabric-deploy-remove-applications.md) the edited application package to this cluster.

> [!NOTE]
> A Service Fabric cluster is single tenant by design and hosted applications are considered **trusted**. If you are considering hosting **untrusted container applications**, consider deploying them as [guest containers](service-fabric-containers-overview.md#service-fabric-support-for-containers) and see [Hosting untrusted applications in a Service Fabric cluster](service-fabric-best-practices-security.md#hosting-untrusted-applications-in-a-service-fabric-cluster).

You should now have a containerized Service Fabric application running your cluster.

## Next steps

* Learn more about running [containers on Service Fabric](service-fabric-get-started-containers.md).
* Learn about the Service Fabric [application life-cycle](service-fabric-application-lifecycle.md).
