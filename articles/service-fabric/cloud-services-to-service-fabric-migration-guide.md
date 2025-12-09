---
title: Migrate from Azure Cloud Services to Service Fabric 
description: This guide provides detailed steps and best practices for migrating applications from Azure Cloud Services to Azure Service Fabric.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 09/29/2025
# Customer intent: As a cloud architect currently using Cloud Services, I want to know the steps involved in migrating from Cloud Services to Service Fabric, so that I can efficiently migrate my existing cloud architecture to Service Fabric.
---

# Migrate from Azure Cloud Services to Service Fabric

This guide provides detailed steps and best practices for migrating applications from Azure Cloud Services to Azure Service Fabric. Throughout this guide, we recommend using [Service Fabric Managed Clusters](overview-managed-cluster.md) as they provide simplified cluster management, enhanced security, and automated patching.

You should review the [decision matrix for migrating from Cloud Services](cloud-services-migration-decision-matrix.md) to make sure you're choosing the right Azure services from your architecture.

## Pre-migration assessment

Before migrating from Azure Cloud Services to Service Fabric, conduct a thorough assessment:

### Application inventory
- Document all Web and Worker roles
- Identify dependencies and integration points
- Map storage requirements (local disk, Azure Storage, etc.)
- Document scaling requirements

### Traffic patterns and scaling requirements
- Analyze current traffic patterns
- Document scaling triggers and rules
- Assess autoscaling requirements

### State management
- Identify stateful components
- Document data persistence mechanisms
- Assess cache dependencies

### Identify application constraints
- Startup dependencies
- Role communication patterns
- Deployment requirements
- Authentication and security constraints

### Production readiness assessment
Review the [Service Fabric Production Readiness Checklist](service-fabric-production-readiness-checklist.md) to ensure your future Service Fabric application meets production standards.

## Architecture planning

### Service Fabric Managed Cluster vs. Classic Cluster

Service Fabric offers two deployment models:

- **Service Fabric Managed Clusters (recommended)**: Simplified cluster resource model where Microsoft manages underlying cluster infrastructure.
  - Automated OS patching
  - Simplified deployment and management
  - Reduced operational overhead
  - Built-in security best practices
  - [Learn more about Service Fabric Managed Clusters](overview-managed-cluster.md)

- **Traditional Service Fabric Clusters**: Customizable but requires more operational management.

We strongly recommend using **Service Fabric Managed Clusters** for migrations from Cloud Services to simplify operations and ensure better security posture.

### Service Fabric architecture patterns

Map your Cloud Services components to Service Fabric architectural patterns:

| Cloud Services Component | Service Fabric Equivalent |
|--------------------------|---------------------------|
| Web Role | Stateless Service with ASP.NET Core |
| Worker Role | Stateless Service with background processing |
| Role Instances | Service Instances and Partitions |
| Role Environment | Service Fabric Application Context |
| Local Storage | Service Fabric local storage volumes |
| RoleEntryPoint | ServiceInstanceListener or RunAsync method |

### Service Fabric Cluster structure for Managed Clusters

For setting up a Service Fabric Managed Cluster, refer to the official ARM templates available in the [Azure Quickstart Templates repository](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.servicefabric).

A basic managed cluster ARM template looks like this (as shown in the [official documentation](quickstart-managed-cluster-template.md)):

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "clusterName": {
      "type": "string",
      "defaultValue": "GEN-UNIQUE",
      "metadata": {
        "description": "Name of your cluster - Between 3 and 23 characters. Letters and numbers only."
      }
    },
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Remote desktop user Id"
      }
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Remote desktop user password. Must be a strong password"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location of the Cluster"
      }
    }
  },
  "variables": {
    "sfManagedClusterName": "[parameters('clusterName')]",
    "managedNodeType": "NT1"
  },
  "resources": [
    {
      "apiVersion": "2021-11-01-preview",
      "type": "Microsoft.ServiceFabric/managedClusters",
      "name": "[variables('sfManagedClusterName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Basic"
      },
      "properties": {
        "dnsName": "[variables('sfManagedClusterName')]",
        "adminUserName": "[parameters('adminUsername')]",
        "adminPassword": "[parameters('adminPassword')]"
      }
    },
    {
      "apiVersion": "2021-11-01-preview",
      "type": "Microsoft.ServiceFabric/managedClusters/nodeTypes",
      "name": "[concat(variables('sfManagedClusterName'), '/', variables('managedNodeType'))]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.ServiceFabric/managedClusters/', variables('sfManagedClusterName'))]"
      ],
      "sku": {
        "name": "Standard",
        "tier": "Standard"
      },
      "properties": {
        "isPrimary": true,
        "vmInstanceCount": 5,
        "dataDiskSizeGB": 100,
        "vmSize": "Standard_D2s_v3"
      }
    }
  ]
}
```

For detailed setup instructions, see [Quickstart: Deploy a Service Fabric managed cluster using ARM templates](quickstart-managed-cluster-template.md).

### Security considerations

Service Fabric Managed Clusters automatically handle most security configurations, including:
- Automatic cluster certificate rotation
- Default secure node-to-node communication
- Encryption of cluster configuration data
- Built-in network security rules

For client authentication to clusters, you need to configure client certificates:
- **Client Certificates**: For accessing cluster and Service Fabric Explorer
  - Use either self-signed certificates (for development/testing) or
  - CA-issued certificates (recommended for production)
  - Add client certificates to your managed cluster configuration

- **Application Security**: Implement [Application and Service Security](service-fabric-application-and-service-security.md) recommendations
- **Network Security**: Configure NSGs and firewalls according to [Service Fabric Best Practices for Security](service-fabric-best-practices-security.md)

With Managed Clusters, you can focus primarily on application-level security and controlled access to your cluster, while Microsoft handles the underlying cluster security infrastructure.

## Migration strategy

### Choose a migration approach

#### Lift and shift
Minimal changes to application architecture, focusing on adapting existing code to run in Service Fabric.

**Pros:**
- Faster migration timeline
- Lower initial development effort
- Reduced risk of functional changes

**Cons:**
- Doesn't fully leverage Service Fabric capabilities
- May require future refactoring to optimize

**Important Limitation**: Service Fabric Managed Clusters currently **do not support containers**. If your application requires IIS, Windows-specific server components, or other dependencies that would be best containerized, you need to use a traditional Service Fabric cluster with Windows Container support instead of a Managed Cluster. Consider this limitation carefully when planning your lift-and-shift migration approach.  Refer to [Containerize existing Windows applications](/previous-versions/azure/architecture/service-fabric/modernize-app-azure-service-fabric#containerize-existing-windows-applications)

#### Refactor to microservices
Decompose application into microservices for greater scalability and easier maintenance.

**Pros:**
- Full utilization of Service Fabric features
- Improved scalability and resilience
- Better separation of concerns

**Cons:**
- Higher initial development effort
- Requires architectural expertise
- Longer migration timeline

### Migration approaches based on application requirements

#### When to use Managed Clusters:
- Applications built on .NET that can be directly migrated to Service Fabric services
- ASP.NET Core web applications that can run without IIS
- Applications with lightweight dependencies
- New applications written specifically for Service Fabric

#### When to use Classic Service Fabric Clusters:
- Applications requiring Windows Containers
- Workloads with IIS dependencies (should containerize)
- Applications with complex server component dependencies
- Scenarios requiring containerization

### Migration phases

1. **Set up Service Fabric environment**
   - For applications without container dependencies: Create a managed cluster using [Service Fabric Managed Cluster deployment tutorial](tutorial-managed-cluster-deploy.md)
   - For applications requiring containers: Create a traditional Service Fabric cluster with [Windows Container support](service-fabric-get-started-containers.md)
   - Configure networking and security
   - Establish CI/CD pipeline for Service Fabric

2. **Migrate configuration and settings**
   - Map Cloud Service configuration (.cscfg, .csdef) to Service Fabric application manifests
   - Migrate environment settings to Service Fabric parameters

3. **Migrate code**
   - Adapt Web Roles to Stateless Services or containerized applications
   - Adapt Worker Roles to Stateless Services or Reliable Services
   - Migrate Startup Tasks to Service Fabric setup code

4. **Migrate state management**
   - Implement appropriate state management solutions (Reliable Collections)
   - Migrate persistent state from external stores

5. **Implement service communication**
   - Replace role communication with Service Fabric communication patterns
   - Configure service discovery

6. **Test and optimize**
   - Validate functionality and performance
   - Test scaling and failover scenarios
   - Optimize resource usage

## Step-by-step migration process

### 1. Set up a Service Fabric Managed Cluster

To deploy a Service Fabric managed cluster, you can use PowerShell commands as documented in [Tutorial: Deploy a Service Fabric managed cluster](tutorial-managed-cluster-deploy.md).

**Using PowerShell:**

```powershell
# Connect to your Azure account
Login-AzAccount
Set-AzContext -SubscriptionId <your-subscription>

# Create a new resource group
$resourceGroup = "myResourceGroup"
$location = "EastUS2"
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create a Service Fabric managed cluster
$clusterName = "<unique cluster name>"
$password = "Password4321!@#" | ConvertTo-SecureString -AsPlainText -Force
$clientThumbprint = "<certificate thumbprint>"  # Client certificate for authentication
$clusterSku = "Standard"

New-AzServiceFabricManagedCluster -ResourceGroupName $resourceGroup -Location $location -ClusterName $clusterName -ClientCertThumbprint $clientThumbprint -ClientCertIsAdmin -AdminPassword $password -Sku $clusterSKU -Verbose

# Add a primary node type to the cluster
$nodeType1Name = "NT1"
New-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -Name $nodeType1Name -Primary -InstanceCount 5
```

Important notes:
- For production deployments, use the Standard SKU (Basic SKU is only for testing)
- You need to provide a client certificate thumbprint for accessing the cluster
- Every Service Fabric cluster requires one primary node type and may have one or more secondary node types
- [Visualize your cluster with Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)

You can also use the Azure portal or Azure CLI for deployment. For a portal-based setup, follow the [Quickstart: Create a Service Fabric managed cluster](quickstart-managed-cluster-portal.md) tutorial.

### 2. Create Service Fabric application projects

Use Visual Studio to create Service Fabric applications:

**Use Visual Studio:**
1. [Prepare your development environment on Windows](service-fabric-get-started.md)
2. [Create a new Stateless Service - Service Fabric project](service-fabric-reliable-services-quick-start.md)
3. [Create a new Stateful Service - Service Fabric project](service-fabric-create-your-first-application-in-visual-studio.md):

### 3. Migrating Cloud Service Web Roles - [Comprehensive Example](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/MigrationGuides/WebRole_Migration_Example.md)

1. Create a stateless service with ASP.NET Core
2. Migrate controllers and views
3. Configure service endpoints

```csharp
// Service registration in Program.cs
internal sealed class Program
{
    private static void Main()
    {
        try
        {
            ServiceRuntime.RegisterServiceAsync("WebFrontEndType", 
                context => new WebFrontEnd(context)).GetAwaiter().GetResult();

            ServiceEventSource.Current.ServiceTypeRegistered(
                Process.GetCurrentProcess().Id, typeof(WebFrontEnd).Name);

            Thread.Sleep(Timeout.Infinite);
        }
        catch (Exception e)
        {
            ServiceEventSource.Current.ServiceHostInitializationFailed(e.ToString());
            throw;
        }
    }
}

// Service implementation
internal sealed class WebFrontEnd : StatelessService
{
    public WebFrontEnd(StatelessServiceContext context)
        : base(context)
    { }

    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new ServiceInstanceListener[]
        {
            new ServiceInstanceListener(serviceContext =>
                new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                {
                    var builder = WebApplication.CreateBuilder();

                    builder.Services.AddSingleton<StatelessServiceContext>(serviceContext);
                    
                    // Add services to the container
                    builder.Services.AddControllers();
                    builder.Services.AddRazorPages();
                    
                    var app = builder.Build();
                    
                    // Configure middleware
                    if (app.Environment.IsDevelopment())
                    {
                        app.UseDeveloperExceptionPage();
                    }
                    else
                    {
                        app.UseExceptionHandler("/Error");
                        app.UseHsts();
                    }
                    
                    app.UseStaticFiles();
                    app.UseRouting();
                    app.UseAuthorization();
                    
                    app.MapControllers();
                    app.MapRazorPages();
                    
                    return app;
                }))
        };
    }
}
```

### 4. Migrate Cloud Service Worker Roles - [Comprehensive Example](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/MigrationGuides/WorkerRole_Migration_Example.md)

1. Create a stateless service with background processing
2. Move worker logic to RunAsync method
3. Implement service events and timers

```csharp
internal sealed class WorkerBackgroundService : StatelessService
{
    private readonly TimeSpan _interval = TimeSpan.FromSeconds(30);
    
    public WorkerBackgroundService(StatelessServiceContext context)
        : base(context)
    { }

    protected override async Task RunAsync(CancellationToken cancellationToken)
    {
        while (!cancellationToken.IsCancellationRequested)
        {
            try
            {
                // Migrated worker role processing logic
                await ProcessQueueMessagesAsync(cancellationToken);
                await Task.Delay(_interval, cancellationToken);
            }
            catch (Exception ex)
            {
                ServiceEventSource.Current.ServiceMessage(Context, $"Exception in RunAsync: {ex.Message}");
                // Implement appropriate retry logic
            }
        }
    }

    private async Task ProcessQueueMessagesAsync(CancellationToken cancellationToken)
    {
        // Implement your worker logic here
    }
}
```

### 5. Configuration migration

Service Fabric uses a hierarchical configuration model:

1. **ApplicationManifest.xml**: Application-wide configuration

```xml
<ApplicationManifest ApplicationTypeName="MyApplicationType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="WebFrontEnd_InstanceCount" DefaultValue="-1" />
    <Parameter Name="StorageAccountConnectionString" DefaultValue="" />
    <Parameter Name="ASPNETCORE_ENVIRONMENT" DefaultValue="Production" />
  </Parameters>
  
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="WebFrontEndPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides>
      <ConfigOverride Name="Config">
        <Settings>
          <Section Name="ConnectionStrings">
            <Parameter Name="StorageAccount" Value="[StorageAccountConnectionString]" />
          </Section>
          <Section Name="Environment">
            <Parameter Name="ASPNETCORE_ENVIRONMENT" Value="[ASPNETCORE_ENVIRONMENT]" />
          </Section>
        </Settings>
      </ConfigOverride>
    </ConfigOverrides>
  </ServiceManifestImport>
</ApplicationManifest>
```

2. **ServiceManifest.xml**: Service-specific configuration

```xml
<ServiceManifest Name="WebFrontEndPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ConfigPackage Name="Config" Version="1.0.0" />
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>WebFrontEnd.exe</Program>
        <WorkingFolder>CodeBase</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>
  <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpoint" Protocol="http" Port="8080" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

3. **Settings.xml**: Configuration settings

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Settings xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Section Name="ConnectionStrings">
    <Parameter Name="StorageAccount" Value="" />
  </Section>
  <Section Name="Environment">
    <Parameter Name="ASPNETCORE_ENVIRONMENT" Value="Production" />
  </Section>
</Settings>
```

### 6. Access configuration in Service Fabric

```csharp
// Accessing configuration in a Service Fabric service
public sealed class WebFrontEnd : StatelessService
{
    private readonly IConfiguration _configuration;
    
    public WebFrontEnd(StatelessServiceContext context)
        : base(context)
    {
        // Load Service Fabric configuration
        var configPackagePath = context.CodePackageActivationContext.GetConfigurationPackageObject("Config").Path;
        
        _configuration = new ConfigurationBuilder()
            .SetBasePath(configPackagePath)
            .AddJsonFile("appsettings.json", optional: true)
            .AddXmlFile("Settings.xml")
            .AddEnvironmentVariables()
            .Build();
    }
    
    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        // Create service listeners using configuration
        var connectionString = _configuration.GetSection("ConnectionStrings")["StorageAccount"];
        // Use connection string to configure services
    }
}
```

### 7. Deploy to Service Fabric Managed Cluster

1. Package the Service Fabric application:

```powershell
# Package the Service Fabric application
$appPkgPath = "C:\MyServiceFabricApp\pkg"
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $appPkgPath -CompressPackage -SkipCopy
```

2. Deploy to a managed cluster:

```powershell
# Connect to the cluster
Connect-ServiceFabricCluster -ConnectionEndpoint "mycluster.westus.cloudapp.azure.com:19000"

# Register and create the application
Register-ServiceFabricApplicationType -ApplicationPackagePathInImageStore MyServiceFabricApp
New-ServiceFabricApplication -ApplicationName fabric:/MyServiceFabricApp -ApplicationTypeName MyServiceFabricAppType -ApplicationTypeVersion 1.0.0
```

You can also use [Azure Pipelines for automated deployments](how-to-managed-cluster-app-deployment-template.md) to Service Fabric managed clusters.

## Testing and validation

### Functional testing
- Validate all application features
- Test service discovery and communication
- Verify configuration is correctly loaded
- Validate user experience and flows

### Performance testing
- Compare response times with Cloud Services
- Test under expected user load
- Validate autoscaling parameters
- Measure resource usage

### Resilience testing
- Test failover scenarios
- Validate instance recycling behavior
- Test upgrade and rollback processes
- Simulate infrastructure failures

### Validation checklist
- [ ] All features function correctly
- [ ] Performance meets or exceeds Cloud Services
- [ ] Configuration migration is complete
- [ ] Logging and diagnostics work
- [ ] Security requirements are met
- [ ] Deployment pipeline is established
- [ ] Monitoring and alerting are configured
- [ ] Rollback procedures are documented

## Post-migration considerations

### Monitoring and diagnostics

> [!WARNING]
> Application Insights for the Service Fabric SDK is no longer supported.

[Visualize your cluster with Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)

Configure [Service Fabric monitoring and diagnostics](service-fabric-diagnostics-overview.md) for your application:

- Enable Application Insights
- Configure Service Fabric diagnostic collection
- Set up alerts and dashboards
- Implement health reporting

```csharp
// Adding health reporting in your service
var healthClient = new FabricClient().HealthManager;
var healthReport = new HealthReport(
    serviceName: new Uri("fabric:/MyApp/MyService"),
    sourceId: "MyHealthWatcher",
    healthProperty: "Connectivity",
    healthState: HealthState.Ok,
    description: "Service is connected to dependencies"
);
await healthClient.ReportHealthAsync(healthReport);
```

### Scaling and optimizing

Service Fabric managed clusters support [manual scaling](tutorial-managed-cluster-scale.md) and automatic scaling:

```json
{
  "apiVersion": "2021-05-01",
  "type": "Microsoft.ServiceFabric/managedClusters/nodeTypes",
  "name": "[concat(parameters('clusterName'), '/FrontEnd')]",
  "location": "[parameters('location')]",
  "properties": {
    "vmInstanceCount": 5,
    "primaryCount": 5,
    "dataDiskSizeGB": 100,
    "vmSize": "Standard_D2s_v3"
  }
}
```

### Disaster recovery planning

- Configure [Service Fabric backup and restore service](service-fabric-reliable-services-backup-restore.md)
- Implement geo-replication where needed
- Document recovery procedures
- Test disaster recovery scenarios

### Security posture

Follow security best practices:
- Apply [Service Fabric security best practices](service-fabric-best-practices-security.md)
- Regularly update client certificates
- Review network security
- Implement proper authentication and authorization in applications

## Troubleshooting guide

### Deployment issues
- Verify application manifest is correct
- Check cluster health and capacity
- Validate service package versions
- Review deployment logs

### Runtime errors
- Check service logs
- Verify configuration settings
- Validate service communication
- Review health events

### Performance issues
- Analyze resource usage
- Check [partition load](/powershell/module/servicefabric/get-servicefabricpartitionloadinformation?view=azureservicefabricps&preserve-view=true)
- Validate scaling policies
- Review service code for bottlenecks

### Common error scenarios and resolutions

| Error | Possible Cause | Resolution |
|-------|----------------|------------|
| Service activation failed | Missing dependencies or configuration | Verify all dependencies and configuration values are included in service package |
| Communication failures | Network/firewall issues | Check LoadBalancer Rules/Probes, NSG rules, and service endpoints |
| Configuration errors | Parameter mismatches | Validate configuration settings across all layers |
| Scaling issues | Cluster capacity | Review node resource utilization and increase capacity if needed |

## Common migration scenarios

### Web Role migration
For a comprehensive step-by-step guide on migrating ASP.NET Web Roles to Service Fabric Stateless Services, see [Web Role Migration Example](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/MigrationGuides/WebRole_Migration_Example.md). This guide provides detailed code comparisons between original Web Roles and Service Fabric implementations, covering project structure, configuration files, middleware migration, and deployment strategies with side-by-side code samples.

### Worker Role migration
For a detailed guide on transforming Worker Role background processing to Service Fabric, see [Worker Role Migration Example](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/MigrationGuides/WorkerRole_Migration_Example.md). This guide demonstrates the architectural transition from Cloud Services background processing to reliable task execution in Service Fabric, including implementations for reliable timers, queue processing, and state persistence with practical code examples.

### State management migration
For in-depth guidance on migrating application state management to Service Fabric, see [State Management Migration Example](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/MigrationGuides/StateManagement_Migration_Example.md). This technical guide covers transitioning to Reliable Collections with implementation patterns for session management, workflow processing, and caching, as well as data migration strategies, backup/restore procedures, and hybrid approaches combining Service Fabric state management with external stores.

## Additional resources

- [Azure Service Fabric Documentation](index.yml)
- [Service Fabric Managed Clusters Overview](overview-managed-cluster.md)
- [Connect to a Service Fabric managed cluster](how-to-managed-cluster-connect.md)
- [Service Fabric Programming Models](service-fabric-choose-framework.md)
- [Service Fabric Architecture](service-fabric-architecture.md)
- [Service Fabric Production Readiness Checklist](service-fabric-production-readiness-checklist.md)
- [Service Fabric Best Practices for Security](service-fabric-best-practices-security.md)
- [Service Fabric Application and Service Security](service-fabric-application-and-service-security.md)
- [Service Fabric Cluster Security](service-fabric-cluster-security.md)
- [Microsoft Training: Introduction to Azure Service Fabric](/training/modules/intro-to-azure-service-fabric/)
- [Service Fabric Sample Applications](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started) 
- [Service Fabric managed cluster configuration options](how-to-managed-cluster-configuration.md)