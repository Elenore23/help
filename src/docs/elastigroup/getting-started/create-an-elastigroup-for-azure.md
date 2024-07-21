# Create an Elastigroup for Azure

This procedure describes how to create an Elastigroup from scratch in Azure.

## Prerequisite

Before you start, [connect your Azure account to Spot](connect-your-cloud-provider/azure-account).

## Get Started

1. In the Spot Console, go to **Elastigroup** > **Groups**.
2. Click **Create** > **Start From Scratch**.

   <details>
     <summary markdown="span">View image</summary>

     <img src="https://github.com/user-attachments/assets/d945cc86-7bdb-47d7-993d-96529f7029f5"/>

   </details>

These are the steps in the Creation Wizard:

1. General
2. Compute
3. Scaling
4. Review

## Step 1: General

### Basic Settings

- Elastigroup Name. can use a naming convention based on the specific workload the Elastigroup will manage, for example, <i>dev-eu1-worker</i>.
- Description.

### Capacity Settings

- **Target** is the number of low priority VMs in your Elastigroup.
- **Minimum** is the minimum number of low priority VMs that must run in the group for a scale-down policy action. The minimum value is <i>0</i>.
- **Maximum** is the maximum number of low priority VMs allowed in the group for a scal-up policy action. The minimum value is <i>0</i>.

Choose one of these:

- **On-Demand Count** is the number of on-demand VMs to include in the Elastigroup.
- **Spot-VMs %** is the percentage of low priority VMs to include in the Elastigroup. The rest will be on-demand instances.

### Availability Settings

- **Draining Timeout** is the time in seconds that Elastigroup will allow to deregister and drain VMs before termination.
- **Cluster Orientation** is the prediction algorithm strategy:
  - **Availability**: VM selection ensures the best market availability within your Elastigroup.
  - **Cost**: VM types are prioritized by cost and market.
- **Fallback to On-Demand** lets Elastigroup automatically fall back to an on-demand VM if no spot VMs are available.
- **Continuous Optimization** lets Elastigroup move workloads from on demand to spot VMs:
  - **Once Available**: Elastigroup moves the workloads when your spot VM types become available.
  - **Custom**: define when to allow the move.

## Step 2: Compute

### Operating System and VM Sizes

- **Resource Group**.
- **Operating System** to run on your VMs.
- **Region** the VMS will run in.
- **Availability Zones** where your VMs are allowed to run. You can select multiple availability zones to diversify the spot markets available to the Elastigroup.
- **On-Demand VM-Sizes** lets you select the regular priority VMs sizes within the Elastigroup. They are used if no low-priority VMs are available in the sizes requested. Make sure the VMs are available in the relevant region.
- **Spot-VM Sizes** are the low-priority VM sizes that need to be available for the Elastigroup. Make sure the VM sizes are available in the relevant region.

> **Tip**: To maximize cost savings, provide the Elastigroup with all possible low-priority VM sizes compatible with the expected workload. The more VM sizes, the better the odds Elastigroup will find an available low-priority VM to run on.

### Image

Choose a type of images:

- **[Marketplace](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/cli-ps-findimage)** is an image available in the Azure Marketplace:
  - **Publisher** is the organization that created the image.
  - **Offer** is the name of a group of related images created by a publisher, such as <i>UbuntuServer</i>, <i>WindowsServer</i>.
  - **SKU** is an instance of an offer, such as a major release of a distribution. For exmaple, <i>18.04-LTS</i>, <i>2019-Datacenter</i>.
- **Custom** VM image:
  - **Image Resource Group** is a list of Resource Groups associated with your subscription.
  - **[Image Name](https://docs.microsoft.com/en-us/dynamics-nav/how-to--get-the-microsoft-azure-image-name)** is the label of the image.
- **[Shared Image](https://docs.microsoft.com/en-us/azure/virtual-machines/shared-image-galleries)** lets you create an Elastigroup with an image from your organization’s shared image gallery:
  - **Image Resource Group** is a list of resource groups associated with your subscription.
  - **Gallery Name** is the list of shared gallery names associated with the selected resource group.
  - **Image Name** is the list of shared images associated with the gallery.
    
     <details>
       <summary markdown="span">See more about image name</summary>

       All of the image names in the list have the OS that you chose under Operating System and VM Sizes.

       Images are <i>Generalized</i> or <i>Specialized</i>. For a specialized image:
       * You do not specify login information for the image.
       * Custom data is not available.

   </details>
   
  - **Version** is the list of available versions of the selected image in your selected region. If you need the most recent version, select <i>Latest</i>.

### Network

- **Vnet Resource Group** you want your Elastigroup scale sets to be a part of.
- **Virtual Network** (VN) for your Elastigroup.
- Network Interface:
  - **Set as Primary** makes this the main network interface attached to the VM.
  - **Subnet ID** is the specific subnet inside your VN.
  - **Network Resource Group** is  where the network interface will be created.
  - **Network Security Group** to associate with the VM.
  - **Application Security Group** provides security micro-segmentation virtual networks in Azure and lets you define network security policies based on workloads (such as applications) instead of explicit IP addresses.
    The application security groups depend on the virtual network that you choose and may be different in each network.
  - **Assign Public IP** if you want VMs in this Elastigroup to launch with a public IP. The list of static public IPs only includes IPs from availability zones that you set for the Elastigroup.

     <details>
       <summary markdown="span">See more about public IPs</summary>

      You can minimize the number of new IPs created ad hoc on VM launchers:
      * Choose IPs that are <i>no zone</i> or <i>zone redundant</i>. These give the most availability zone flexibility.
      * The ideal number of public IPs for the pool is twice your maximum capacity. For example, if your maximum capacity is 6 VMs, then choose at least 12 public IP addresses.
      * If you choose zonal IPs (for example, in zones 1, 2, 3), distribute them equally across the zones.

   </details>

### Login

- **User Name**.
- **Authentication type**  can be <i>SSH public key</i> or <i>password</i> (the password you use to sign in to Linux or Windows).

When a specialized shared [Image](elastigroup/getting-started/create-an-elastigroup-for-azure?id=image) is used, you do not need to give login information.

### Load Balancers (optional)

You can add a load balancer. Elastigroup will automatically register new VMs to the configured load balancer.
Keep in mind:
* You can have up to one private and one public standard load balancer.
* Standard load balancers are not available if you assign a basic public IP to your network interface or if basic VM sizes are selected.

### Health Checks (optional)

* **Health check types**:
  * **VM state** checks the VM’s current status in Azure.
  * **Application gateway** tests the connection from the application gateway to the VM. It’s available if at least one application gateway is defined in the Elastigroup. You also select the health check grace period in seconds, which is the time to allow a VM to boot and applications to fully start before the first health check.
* **Auto healing** checks the VM health according to the health check types and replaces unhealthy VMs.
  
### Additional Configuration (optional)

- **Managed Identity** for your VMSS instances.
- **Tags** let you add tag keys and values for the Elastigroup VMs.
- **Custom Data** is a script that will run during every VM startup. It's useful for launching VMs with all the preset configurations and software installations. Elastigroup can load custom user data (such as custom scripts) when provisioning VMs. Custom data is not available for specialized shared images.
    Make sure your script doesn't require additional [extensions](https://docs.spot.io/managed-instance/azure/tutorials/extensions). For example, you may need to add an extension for custom data to work.
  
   <details>
     <summary markdown="span">Extension for custom data</summary>

    Add the `customDataRunner` extension:

    1. Go to **group** > **compute** > **launchSpecification** > **extensions**.
    2. Add this extension:
       <pre><code>
       "extensions": [
              {
                "name": "customDataRunner",
                "type": "CustomScriptExtension",
                "publisher": "Microsoft.Compute",
                "apiVersion": "1.10",
                "minorVersionAutoUpgrade": true,
                "publicSettings": {
                  "commandToExecute": "copy C:\\AzureData\\CustomData.bin C:\\AzureData\\CustomData.ps1 & powershell -ExecutionPolicy Unrestricted -File C:\\AzureData\\customData.ps1"
                }
              }
            ]
       </code></pre>
    
   </details>
  
   <details>
     <summary markdown="span">See more about custom data and user data</summary>

    <b>User data</b>

    <a href="https://learn.microsoft.com/en-us/azure/virtual-machines/user-data">User data</a> is a set of scripts or other metadata inserted into an Azure virtual machine during provisioning. After provision, any application on the virtual machine can access the user data from the Azure Instance Metadata Service (IMDS).

    <b>Custom data</b>

    <a href="https://learn.microsoft.com/en-us/azure/virtual-machines/custom-data">Custom data</a> is made available to the VM during provisioning.

    <b>What’s the difference between the two?</b>

    The main difference is that user data is available when OS persistence is on, and custom data is not.
    
   </details>


- **Shutdown Script** requires an agent to be installed on the instance. [Learn more](elastigroup/features/azure/shutdown-script-in-elastigroup-for-azure).

## Step 3: Scaling

Optionally, create scaling policies for your Elastigroup based on Azure Monitor metrics. A policy can be for scale up or scale down.

To create a scaling policy, complete the following steps.

1. Click Add Policy.
2. Set policy name.
3. Specify a Namespace (default is Microsoft.Compute).
4. Set scale based on values: Choose Trigger (Metric Name), Behavior.
5. Set Duration to determine the number of tests (and duration between them) to activate the policy.
6. Choose the action type:
   - Adjustment. Will add instances on up scaling and remove on down scaling policies. You need to set the number of instances.
   - Set the minimum target.
   - Update Capacity. In terms of Target Minimum and Maximum.
   - Percentage Adjustment. Add or Remove a percentage of the group active capacity, for Up and Down scaling respectively.
7. Cool-down.
   - Wait Period is the time (in seconds) that all scaling activities will be suspended after the scaling policy is triggered.

<img src="/elastigroup/_media/gettingstarted-eg-azure-06.png" />

## Step 4: Review

1. Review your configuration in JSON format in the Review tab. You can edit the JSON directly by switching on Edit mode.

<img src="/elastigroup/_media/gettingstarted-eg-azure-07.png" />

2. To create the Elastigroup, click Create.

## What's Next?

Learn how to:

- [Import Existing Azure Resources](elastigroup/azure/getting-started/import-an-existing-azure-resource.md) such as a Scale Set, an Application Gateway, or a VM.
- [Clone an Existing Elastigroup](elastigroup/tutorials/azure/clone-an-existing-elastigroup).
