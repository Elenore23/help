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

Enter the information for your network interface. You can define additional network interfaces as needed.

- Vnet Resource Group. Select the Vnet Resource group you want your Elastigroup Scale Sets to be a part of.
- Virtual Network. Select the specific Virtual Network (VN) for your Elastigroup.
- Set as Primary. The main network interface attached to the VM.
- Subnet ID. Select the specific Subnet inside your VN.
- Resource Group. The resource group where the network interface will be created.
- Network Security Group. The network security group to associate with the VM.

<img src="/elastigroup/_media/gettingstarted-eg-azure-03b.png" width="514" height="301" />

- Application Security Group: Provides security micro-segmentation virtual networks in Azure and enables you to define network security policies based on workloads (i.e., applications) instead of explicit IP addresses. Choose one or more application security groups the Elastigroup should belong to.

  <img src="/elastigroup/_media/gettingstarted-eg-azure-03c.png" width="331" height="171" />

  Note that the items appearing in the list of application security groups depend on the Virtual Network that you choose and may be different in each network.

- Assign Public IP. Mark this checkbox if you want VMs in this Elastigroup to launch with a Public IP. You will then need to choose one or more Static Public IPs from the dropdown list. The list will include IPs only from AZs that you have chosen for the Elastigroup.

<img src="/elastigroup/_media/gettingstarted-eg-azure-03d.png" width="273" height="362" />

#### More on Choosing Public IPs

In order to minimize ad hoc creation of new IPs on VM launchers, the following is recommended:

- Choose IPs that are indicated as No zone/Zone redundant. These will ensure the most AZ flexibility.
- The optimal number of public IPs for the pool is twice your maximum capacity. For example, if your maximum capacity is 6 VMs, then choose at least 12 public IP addresses.
- If you choose zonal IPs (e.g., in zones 1, 2, 3), then distribute them equally across the zones.

### Login

- User Name. Specify the user name you wish to SSH the VM's with.
- Windows Password. The password you use for your Windows login.

> **Tip**: When a Specialized Shared [Image](elastigroup/getting-started/create-an-elastigroup-for-azure?id=image) is specified, you do not need to specify login information.

### Additional Configuration (optional)

- Managed Identity. Select the Managed Identity for your VMSS instances.
- Tags. Add tag keys and values you want associated with the Elastigroup VMs.
- Custom Data. Custom data is useful for launching VMs with all required configurations and software installations. Elastigroup can load custom user data (i.e., custom scripts) during the provisioning of VMs. When a Specialized Shared Image is specified, Custom Data is not available.
- Shutdown Script. You can configure a shutdown script, but this requires an agent to be installed on the instance. [Learn more](elastigroup/features/azure/shutdown-script-in-elastigroup-for-azure).

<img src="/elastigroup/_media/gettingstarted-eg-azure-05.png" />

### Load Balancers (optional)

You can add a load balancer. Elastigroup will automatically register new VMs to the configured load balancer.

<img src="/elastigroup/_media/gettingstarted-eg-azure-03aa.png" />

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
