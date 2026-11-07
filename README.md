# AZ-Lifecycle-of-scaling-an-application

https://github.com/udacity/CD14673-Deploy-and-Manage-Azure-Compute-Resources-Public1/tree/main

Stage 1: Container Registry (ACR)
Requirements
1. Resource Group
Create a resource group with the following configuration:

Name: my_rg
Region: East US
All Stage 1 resources must be deployed in this resource group.

2. Azure Container Registry (ACR)
Provision an Azure Container Registry with the following configuration:

Region: East US
SKU: Standard
Availability Zones: Disabled
Public network access: Enabled
Role assignment permissions mode: RBAC Registry Permissions (default)
Encryption: Microsoft-managed keys (Customer-managed keys disabled)
The registry name must be globally unique and consistently used across all deliverables.

3. Container Image Deployment
Using the provided Docker image (available in this repository(opens in a new tab)):

Build and push the image to your Azure Container Registry.
The repository must contain at least one image with a tag.
The image must be visible in the Repositories blade of the Azure Portal under your ACR instance.
You may use either:

Azure CLI (az acr build, az acr login, etc.), or
Docker CLI (docker build, docker tag, docker push)
4. Infrastructure-as-Code (IaC)
Export the ACR deployment as a Bicep template.

The exported .bicep file must:

Contain a resource definition for Microsoft.ContainerRegistry/registries
Include key configuration properties (name, SKU, location, etc.)
Compile successfully using Bash
The template must represent a valid, redeployable configuration.



Stage 2: Startup Deployment (ACI)
The startup requires a rapid proof-of-concept deployment of the containerized application. You will deploy the image from Azure Container Registry (Stage 1) to Azure Container Instances (ACI), monitor performance under load, and validate failure behavior when resource limits are exceeded.

All resources must be deployed in East US and within the existing resource group: my_rg

Requirements
1. Azure Container Instance Deployment
Deploy an Azure Container Instance using the container image stored in your Azure Container Registry from Stage 1.

The container group must meet the following configuration:

Region: East US
Container image source: Azure Container Registry
Operating System: Linux
vCPU: 1
Memory: 1.5 GiB
GPU: None
Networking type: Public
Exposed port: 80 (TCP)
Availability Zones: None
The container must successfully start and be reachable via its public IP address.

2. Authentication to ACR
Configure the ACI deployment to authenticate to Azure Container Registry.

The container must successfully pull the image from ACR without manual image transfer.

3. Monitoring and Logging
Configure monitoring to observe:

CPU utilization
Memory utilization
Container lifecycle events
You must configure logging sufficient to capture container restart events caused by resource exhaustion.

4. Resource Stress Testing
Using the application’s built-in controls:

Simulate CPU and memory load.
Increase memory usage progressively until the container exceeds its configured memory limit.
Observe container restart behavior.
The container must demonstrate failure due to exceeding memory limits (Out Of Memory condition).

5. Infrastructure-as-Code (IaC)
Export the Azure Container Instance deployment as a Bicep template.

The exported .bicep file must:

Contain a resource definition for:
Microsoft.ContainerInstance/containerGroups
Include container image reference from your ACR
Reflect configured CPU and memory limits
Compile successfully using az bicep build
The template must represent a valid and redeployable configuration.



Stage 3: Scalable Containers with Azure Container Apps (ACA)
Scenario
The company now requires automatic scaling and cost efficiency. The application must scale down to zero when idle and automatically scale out under load. You will migrate the containerized application to Azure Container Apps (ACA), configure autoscaling, and validate scaling behavior and resource limits.

All resources must be deployed in:

Resource Group: my_rg
Region: East US
The container image must be pulled from your Azure Container Registry created in Stage 1.
Requirements
1. Deploy Azure Container App
Deploy an Azure Container App with the following configuration:
General Configuration

Region: East US
Deployment source: Container image
Image source: Azure Container Registry (Stage 1 registry)
Operating system: Linux
Public ingress: Enabled
Ingress type: HTTP
Target port: 80
Workload Profile

Workload profile: Consumption
Minimum replicas: 0
Maximum replicas: 4
Container Resource Configuration (Per Replica)

CPU: 0.5 vCPU
Memory: 1 GiB
The container must successfully start and be accessible via its public URL.

2. Validate Scale-to-Zero Behavior
Allow the application to become idle and confirm:

Replica count reduces to 0.
The application scales from 0 to 1 when traffic is reintroduced.
3. Validate Autoscaling Under Load
Using the application’s built-in load controls:

Generate sustained CPU and memory load.
Confirm that:
CPU and/or Memory utilization increases.
Replica count increases above 1.
The number of replicas does not exceed 4.
Scaling behavior must align with configured minimum and maximum replica settings.

4. Validate Resource Limits and Failure Behavior
Increase memory usage progressively until the container exceeds its configured memory limit.

Confirm:

Container restart occurs due to memory exhaustion.
Restart is observable through logs or events.
Restart timing aligns with stress testing activity.
5. Infrastructure-as-Code (IaC)
Export the Azure Container App deployment as a Bicep template.

The exported .bicep file must:

Include a resource of type:
Microsoft.App/containerApps
Include:

Image reference from your Stage 1 ACR
Minimum replicas = 0
Maximum replicas = 4
Container CPU and memory limits
Compile successfully using:
az bicep build
The template must represent a valid and redeployable configuration.



Stage 4: App Services with Deployment Slots (Production-Ready Web App)
Scenario
The organization now requires a production-ready deployment model that supports:

Horizontal autoscaling
Monitoring and alerting
Safe deployments using staging slots
Traffic routing for A/B testing
You will deploy the containerized application to Azure App Service, configure autoscaling and monitoring, and implement deployment slots with traffic splitting.

All resources must be deployed in:
Resource Group: my_rg
Region: East US
The container image must be pulled from your Stage 1 Azure Container Registry.
Requirements
1. Deploy App Service Plan
Provision an App Service Plan with the following configuration:

Operating System: Linux
Region: East US
Pricing Tier: Standard S1
Zone Redundancy: Disabled
The plan must support autoscaling.
2. Deploy Web App (Container-Based)
Deploy a Web App using:

Publish method: Container
Operating System: Linux
App Service Plan: The plan created above
Image source: Azure Container Registry
Container image: The image from Stage 1 ACR
Authentication to ACR: Managed identity
The Web App must be publicly accessible and successfully serve traffic.

3. Configure Autoscaling
Configure autoscale settings for the App Service Plan with the following parameters:

Minimum instances: 1
Maximum instances: 3
Default instances: 1
Scale-Out Rule

Trigger: CPU percentage > 70%
Evaluation period: 5 minutes
Action: Increase by 1 instance
Cooldown: 5 minutes
Scale-In Rule

Trigger: CPU percentage < 30%
Evaluation period: 7 minutes
Action: Decrease by 1 instance
Cooldown: 7 minutes
Validate that autoscaling responds appropriately under load.

4. Configure Monitoring and Alerts
Create a metric alert with the following configuration:

Metric: CPU Percentage
Threshold: Greater than 90%
Evaluation period: 10 minutes
Action: Email notification via Action Group
Trigger the alert by generating sufficient CPU load and confirm the notification is received.

5. Configure Deployment Slots
Create a staging deployment slot for the Web App.

Requirements:

Both Production and Staging slots must be visible.
Deploy a modified version of the application to the staging slot (e.g., version identifier change).
Configure traffic routing so that a non-zero percentage of traffic is directed to the staging slot.
Confirm traffic split is active and observable.
6. Validate Scaling and Slot Behavior
Under simulated load:

Confirm CPU utilization increases.
Confirm scale-out occurs from 1 to 2 and then to 3 instances.
Confirm scale-in occurs when load decreases.
Verify per-instance CPU and memory metrics are visible.
Confirm staging slot receives traffic when routing percentage is applied.
7. Infrastructure-as-Code (IaC)
After completing all configurations (autoscale, alerts, slots), export the deployment as a Bicep template.

The exported .bicep file must include:

Microsoft.Web/serverfarms
Microsoft.Web/sites
Deployment slot resource
Autoscale configuration
Alert configuration
The template must compile successfully using:
az bicep build
The template must represent a valid and redeployable configuration.




Stage 5: Virtual Machines (IaaS Deployment and Encryption)
Scenario
The organization now requires full control over the operating system, networking, and storage configuration. You will deploy the application on a Linux Virtual Machine, validate performance scaling, move the resource between resource groups, and implement disk encryption using customer-managed keys.

All resources must initially be deployed in:

Resource Group: my_rg
Region: East US
Requirements
1. Deploy Linux Virtual Machine
Provision a Virtual Machine with the following configuration:

General Configuration

Operating System: Ubuntu Server 22.04 LTS or newer (x64, Gen2)
Region: East US
Availability: Zone 1
Security type: Trusted Launch
VM Size: 1 vCPU, approximately 3–4 GiB memory
Authentication type: SSH public key
Disk Configuration

OS Disk Size: Default (approximately 30 GiB)
OS Disk Type: Premium SSD
Delete disk with VM: Enabled
Networking Configuration

Public IP assigned
Inbound rules must allow:
SSH (Port 22) from your IP address
HTTP (Port 8000) from your IP address
The VM must be reachable via SSH.

2. Deploy and Run Application
Deploy the application inside the VM and ensure:

The application listens on port 8000.
The application is accessible from a browser using the VM’s public IP address.
The application can generate CPU and memory load for testing.
3. Monitor Performance
Using Azure Monitor:

Capture CPU utilization metrics.
Capture Memory utilization metrics.
Record metrics under load while running the application.
4. Resize the Virtual Machine
Resize the VM to a larger instance with:

At least 2 vCPUs
Increased memory capacity
After resizing:

Confirm the application is still operational.
Capture CPU and Memory metrics again under load.
Compare performance behavior before and after resizing.
5. Export ARM Template (Initial Deployment)
Export the ARM template of the Virtual Machine deployment before moving the resource to another resource group.

The exported template must include:

Microsoft.Compute/virtualMachines
Associated networking resources
6. Move Virtual Machine to a New Resource Group
Move the Virtual Machine and its dependent resources to a newly created resource group.

After the move:

Export the ARM template again.
Provide a short written note describing which properties changed as a result of the move.
7. Configure Disk Encryption with Customer-Managed Keys
Implement disk encryption using a Disk Encryption Set.

Requirements:

Create a Key Vault for key management.
Create a Disk Encryption Set referencing the Key Vault key.
Deploy a new managed data disk:
Minimum size: 8 GiB
Encryption: Server-side encryption with Customer-Managed Key (SSE with CMK)
Attach the encrypted disk to the Virtual Machine.

8. Validate Encrypted Disk Inside VM
Inside the VM:

Confirm the encrypted disk is visible.
Mount the disk.
Verify it is accessible for use.

Stage 6: Virtual Machine Scale Sets (Elastic Scaling)
Scenario
The organization now requires automatic, elastic scaling to handle traffic spikes. You will create a reusable VM image from Stage 5, deploy a Virtual Machine Scale Set (VMSS), configure autoscaling rules, and validate scale-out and scale-in behavior under real workload conditions.

All resources must be deployed in:

Region: East US
The VM image used for VMSS must originate from the Virtual Machine created in Stage 5.
Requirements
1. Create Azure Compute Gallery Image
From the Virtual Machine created in Stage 5:

Create an image version in an Azure Compute Gallery.
The image must capture the configured application so that new instances automatically run the web service on startup.
The image must be reusable for VMSS deployment.
2. Deploy Virtual Machine Scale Set (VMSS)
Deploy a VMSS using the Azure Compute Gallery image created above.

General Configuration

Region: East US
Security type: Trusted Launch
Operating System: Ubuntu 22.04 LTS or newer
VM Architecture: x64
VM Size: Standard_B1s (or equivalent 1 vCPU instance)
Authentication: SSH public key
Capacity Configuration

Minimum instances: 1
Maximum instances: 6
Default instances: 1
3. Configure Networking
Configure a Load Balancer to provide a single public IP address for accessing the application.

Requirements:

Public frontend IP
Backend pool linked to VMSS
Health probe configured
Load balancing rule mapping:
Frontend port 80
Backend port 8000
The application must be accessible via the Load Balancer public IP address.

4. Configure Autoscale Rules
Configure autoscale rules with the following parameters:

Scale-Out Rule

Trigger: CPU Percentage > 70%
Evaluation period: 5 minutes
Action: Increase instance count by 1
Cooldown: 5 minutes
Scale-In Rule

Trigger: CPU Percentage < 30%
Evaluation period: 7 minutes
Action: Decrease instance count by 1
Cooldown: 7 minutes
Autoscale limits must respect:

Minimum = 1
Maximum = 6
5. Validate Scaling Behavior
Using the application to generate load:

Under Load

Confirm CPU utilization rises above 70%.
Confirm instance count increases above 1.
Confirm scaling does not exceed 6 instances.
After Load Reduction

Reduce CPU utilization below 30%.
Confirm instance count decreases toward 1.
Confirm scale-in behavior aligns with cooldown settings.
Metrics must clearly show:

CPU percentage
Instance count
Correlation between CPU threshold crossing and scaling event must be visible.

6. Infrastructure-as-Code (IaC)
After autoscale configuration is complete, export the VMSS deployment as an ARM template.

The exported template must include:

Microsoft.Compute/virtualMachineScaleSets
Capacity configuration showing:
min = 1
max = 6
default = 1
Autoscale rules with:
70% scale-out threshold
30% scale-in threshold
Image reference pointing to the Azure Compute Gallery image created from Stage 5
The template must represent a valid and redeployable configuration.
