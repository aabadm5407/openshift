---

copyright:
  years: 2014, 2020
lastupdated: "2020-05-13"

keywords: openshift, roks, rhoks, rhos, clusters

subcollection: openshift

---

{:codeblock: .codeblock}
{:deprecated: .deprecated}
{:download: .download}
{:external: target="_blank" .external}
{:faq: data-hd-content-type='faq'}
{:gif: data-image-type='gif'}
{:help: data-hd-content-type='help'}
{:important: .important}
{:new_window: target="_blank"}
{:note: .note}
{:pre: .pre}
{:preview: .preview}
{:screen: .screen}
{:shortdesc: .shortdesc}
{:support: data-reuse='support'}
{:table: .aria-labeledby="caption"}
{:tip: .tip}
{:troubleshoot: data-hd-content-type='troubleshoot'}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}
{:tsSymptoms: .tsSymptoms}


# Creating OpenShift clusters
{: #clusters}

Create a cluster in {{site.data.keyword.openshiftlong}}.
{: shortdesc}

After [getting started](/docs/containers?topic=containers-getting-started), you might want to create a cluster that is customized to your use case and different public and private cloud environments. Consider the following steps to create the right cluster each time.

1.  [Prepare your account to create clusters](/docs/openshift?topic=openshift-clusters#cluster_prepare). This step includes creating a billable account, setting up an API key with infrastructure permissions, making sure that you have Administrator access in {{site.data.keyword.cloud_notm}} IAM, planning resource groups, and setting up account networking.
2.  [Decide on your cluster setup](/docs/openshift?topic=openshift-clusters#prepare_cluster_level). This step includes planning cluster network and HA setup, estimating costs, and if applicable, allowing network traffic through a firewall.
3.  Create your [cluster](#clusters_ui) by following the steps in the {{site.data.keyword.cloud_notm}} console or CLI.

<br />


## Sample commands
{: #cluster_create_samples}

Have you created a cluster before and are just looking for quick example commands? Try these examples.
{: shortdesc}




* Classic cluster, shared virtual machine:
   ```
   ibmcloud oc cluster create classic --name my_cluster --version 4.3_openshift --zone dal10 --flavor b3c.4x16 --hardware shared --workers 3 --public-vlan <public_VLAN_ID> --private-vlan <private_VLAN_ID>
   ```
   {: pre}
*  Classic cluster, bare metal:
   ```
   ibmcloud oc cluster create classic --name my_cluster --version 4.3_openshift --zone dal10 --flavor mb2c.4x32 --hardware dedicated --workers 3 --public-vlan <public_VLAN_ID> --private-vlan <private_VLAN_ID>
   ```
   {: pre}
*  Classic cluster with an IBM Cloud&trade; Pak entitlement for a default worker pool of 3 worker nodes with 4 cores and 16 memory each:
   ```
   ibmcloud oc cluster create classic --name cloud_pak_cluster --version 4.3_openshift --zone dal10 --flavor b3c.4x16 --hardware dedicated --workers 3 --entitlement cloud_pak --public-vlan <public_VLAN_ID> --private-vlan <private_VLAN_ID>
   ```
   {: pre}
*  For a classic multizone cluster, after you created the cluster in a [multizone metro](/docs/openshift?topic=openshift-regions-and-zones#zones), [add zones](/docs/openshift?topic=openshift-add_workers#add_zone):
   ```
   ibmcloud oc zone add classic --zone <zone> --cluster <cluster_name_or_ID> --worker-pool <pool_name> --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
   ```
   {: pre}



<br />


## Prepare to create clusters at the account level
{: #cluster_prepare}

Prepare your {{site.data.keyword.cloud_notm}} account for {{site.data.keyword.containerlong_notm}}. After the account administrator makes these preparations, you might not need to change them each time that you create a cluster. However, each time that you create a cluster, you still want to verify that the current account-level state is what you need it to be.
{: shortdesc}

1. [Create or upgrade your account to a billable account ({{site.data.keyword.cloud_notm}} Pay-As-You-Go or Subscription)](https://cloud.ibm.com/registration/).

2. [Set up an API key for Red Hat OpenShift on IBM Cloud](/docs/openshift?topic=openshift-users#api_key) in the region and resource groups where you want to create clusters. Assign the API key with the appropriate service and infrastructure permissions to create clusters.<p class="tip">Are you the account owner? You already have the necessary permissions! When you create a cluster, the API key for that region and resource group is set with your credentials.</p>

  **Services**:
    * [**Administrator** platform role](/docs/openshift?topic=openshift-users#platform) for {{site.data.keyword.containerlong_notm}} at the account level.
    * [**Writer** or **Manager** service role](/docs/openshift?topic=openshift-users#platform) for {{site.data.keyword.containerlong_notm}}.
    * [**Administrator** platform role](/docs/openshift?topic=openshift-users#platform) for Container Registry at the account level. If your account predates 4 October 2018, you need to [enable {{site.data.keyword.cloud_notm}} IAM policies for {{site.data.keyword.registrylong_notm}}](/docs/Registry?topic=registry-user#existing_users). With IAM policies, you can control access to resources such as registry namespaces.

  **Infrastructure**:
    * **Super User** role or the [minimum required permissions](/docs/openshift?topic=openshift-access_reference#infra) for classic infrastructure.

3. Verify that you as a user (not just the API key) have the **Administrator** platform role for {{site.data.keyword.containerlong_notm}}. To allow your cluster to pull images from the private registry, you also need the **Administrator** platform role for {{site.data.keyword.registrylong_notm}}. If you are the account owner, you already have these permissions.
  1. From the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/){: external} menu bar, click **Manage > Access (IAM)**.
  2. Click the **Users** page, and then from the table, select yourself.
  3. From the **Access policies** tab, confirm that your **Role** is **Administrator**. You can be the **Administrator** for all the resources in the account, or at least for {{site.data.keyword.containerlong_notm}}. **Note**: If you have the **Administrator** role for {{site.data.keyword.containerlong_notm}} in only one resource group or region instead of the entire account, you must have at least the **Viewer** role at the account level to see the account's VLANs.
  <p class="tip">Make sure that your account administrator does not assign you the **Administrator** platform role at the same time as scoping the access policy to a namespace.</p>

4. If your account uses multiple resource groups, figure out your account's strategy for [managing resource groups](/docs/openshift?topic=openshift-users#resource_groups).
  * The cluster is created in the resource group that you target when you log in to {{site.data.keyword.cloud_notm}}. If you do not target a resource group, the default resource group is automatically targeted. Free clusters are created in the `default` resource group.
  * If you want to create a cluster in a different resource group than the default, you need at least the **Viewer** role for the resource group. If you do not have any role for the resource group, your cluster is created in the default resource group.
  * You cannot change a cluster's resource group. Furthermore, if you need to use the `ibmcloud oc cluster service bind` [command](/docs/containers-cli-plugin?topic=containers-cli-plugin-kubernetes-service-cli#cs_cluster_service_bind) to [integrate with an {{site.data.keyword.cloud_notm}} service](/docs/openshift?topic=openshift-service-binding#bind-services), that service must be in the same resource group as the cluster. Services that do not use resource groups like {{site.data.keyword.registrylong_notm}} or that do not need service binding like {{site.data.keyword.la_full_notm}} work even if the cluster is in a different resource group.
  * Consider giving clusters unique names across resource groups and regions in your account to avoid naming conflicts. You cannot rename a cluster.

<br />


## Deciding on your cluster setup
{: #prepare_cluster_level}
{: help}
{: support}

After you set up your account to create clusters, decide on the setup for your cluster. You must make these decisions every time that you create a cluster. You can click on the options in the following decision tree image for more information, such as comparisons of free and standard, Kubernetes and OpenShift, or VPC and classic clusters.
{: shortdesc}

<p class="note">OpenShift clusters are available only as standard clusters. You cannot get a [free](/docs/openshift?topic=openshift-faqs#faq_free) OpenShift cluster.</p>


<img usemap="#cluster-plan-map" border="0" class="image" src="images/cluster-plan-dt.png" alt="This image walks you through choosing the setup that you want for your cluster."/>
<map name="cluster-plan-map">
    <area target="" alt="Free and standard cluster comparison" title="Free and standard cluster comparison" href="/docs/containers?topic=containers-cs_ov#cluster_types" coords="43,9,361,106" shape="rect">
    <area target="" alt="OpenShift and Kubernetes comparison" title="OpenShift and Kubernetes comparison" href="/docs/openshift?topic=openshift-cs_ov#openshift_kubernetes" coords="110,128,467,224" shape="rect">
    <area target="" alt="VPC and classic infrastructure comparison" title="VPC and classic infrastructure comparison" href="/docs/containers?topic=containers-infrastructure_providers" coords="60,252,398,352" shape="rect">
    <area target="" alt="Locations" title="Locations" href="/docs/containers?topic=containers-regions-and-zones#zones" coords="101,377,564,456" shape="rect">
    <area target="" alt="Virtual Machines" title="Virtual Machines" href="/docs/containers?topic=containers-planning_worker_nodes#vm" coords="105,488,564,538" shape="rect">
    <area target="" alt="Bare metal machines" title="Bare metal machines" href="/docs/containers?topic=containers-planning_worker_nodes#bm" coords="566,569,372,546" shape="rect">
    <area target="" alt="VPC scenarios" title="VPC scenarios" href="/docs/containers?topic=containers-plan_clusters#vpc-scenarios" coords="104,597,298,675" shape="rect">
    <area target="" alt="Classic scenarios" title="Classic scenarios" href="/docs/containers?topic=containers-plan_clusters#classic-scenarios" coords="369,596,566,674" shape="rect">
    <area target="" alt="Classic firewall" title="Classic firewall" href="/docs/containers?topic=containers-firewall" coords="369,681,564,704" shape="rect">
    <area target="" alt="VPC ACLs and firewall" title="VPC ACLs and firewall" href="/docs/containers?topic=containers-firewall" coords="103,680,298,704" shape="rect">
    <area target="" alt="Estimate costs (cluster create page)" title="Estimate costs (cluster create page)" href="https://cloud.ibm.com/kubernetes/catalog/create" coords="248,732,426,776" shape="rect">
</map>


<br />




## Creating a standard classic cluster in the console
{: #clusters_ui}

Create your single zone or multizone classic OpenShift cluster by using the {{site.data.keyword.cloud_notm}} console.
{: shortdesc}

1. Make sure that you complete the prerequisites to [prepare your account](#cluster_prepare) and decide on your [cluster setup](#prepare_cluster_level). Keep in mind that you need a cluster with a minimum of 2 worker nodes of the `4x16` flavor so that default OpenShift components can deploy.
2. From the [{{site.data.keyword.cloud_notm}} Red Hat OpenShift on IBM Cloud Clusters console](https://cloud.ibm.com/kubernetes/clusters?platformType=openshift){: external}, click **Create cluster**.
3. Configure your cluster environment.
   4. Give your cluster a unique name. The name must start with a letter, can contain letters, numbers, and hyphen (-), and must be 35 characters or fewer. Use a name that is unique across regions. The cluster name and the region in which the cluster is deployed form the fully qualified domain name for the Ingress subdomain. To ensure that the Ingress subdomain is unique within a region, the cluster name might be truncated and appended with a random value within the Ingress domain name.
 **Note**: Changing the unique ID or domain name that is assigned during cluster creation blocks the OpenShift master from managing your cluster.
   5. **Optional**: Add a [tag](/docs/resources?topic=resources-tag) to your cluster to help manage {{site.data.keyword.cloud_notm}} resources, such as the team or billing department that uses the cluster.
   6. Select a resource group in which to create your cluster.
      * A cluster can be created in only one resource group, and after the cluster is created, you can't change its resource group.
      * To create clusters in a resource group other than the default, you must have at least the [**Viewer** role](/docs/openshift?topic=openshift-users#platform) for the resource group.
4. Select **Single zone** or **Multizone** availability. In a multizone cluster, the OpenShift master is deployed in a multizone-capable zone and three replicas of your master are spread across zones.
5. Select a geography in which to deploy your cluster.
6. Enter your metro city and zone details.
  * Multizone clusters:
    1. Select a metro location. For the best performance, select the metro location that is physically closest to you. Your choices might be limited by geography.
    2. Select the specific zones in which you want to host your cluster. You must select at least one zone but you can select as many as you like. If you select more than one zone, the worker nodes are spread across the zones that you choose which gives you higher availability. If you select only one zone, you can [add zones to your cluster](/docs/openshift?topic=openshift-add_workers#add_zone) after the cluster is created.
  * Single zone clusters: Select a zone in which you want to host your cluster. For the best performance, select the zone that is physically closest to you. Your choices might be limited by geography.
7. For each of the selected zones, choose your public and private VLANs. Worker nodes communicate with each other by using the private VLAN, and can communicate with the OpenShift master by using the public or the private VLAN. If you do not have a public or private VLAN in this zone, a public and a private VLAN are automatically created for you. You can use the same VLAN for multiple clusters.
9. Configure your default worker pool. Worker pools are groups of worker nodes that share the same configuration. You can always add more worker pools to your cluster later.
   1. Select the **OpenShift version** that you want to use in your cluster. For differences across versions, see [Comparison between OpenShift 3.11 and 4.3 clusters](/docs/openshift?topic=openshift-cs_ov#3.11_vs_4.3).
   1. Filter the worker flavors by selecting a machine type. Virtual is billed hourly and bare metal is billed monthly.
      - **Bare metal**: Bare metal servers are provisioned manually by IBM Cloud infrastructure after you order, and can take more than one business day to complete. Bare metal is best suited for high-performance applications that need more resources and host control. Be sure that you want to provision a bare metal machine. Because it is billed monthly, if you cancel it immediately after an order by mistake, you are still charged the full month.
      - **Virtual - shared**: Infrastructure resources, such as the hypervisor and physical hardware, are shared across you and other IBM customers, but each worker node is accessible only by you. Although this option is less expensive and sufficient in most cases, you might want to verify your performance and infrastructure requirements with your company policies.
      - **Virtual - dedicated**: Your worker nodes are hosted on infrastructure that is devoted to your account. Your physical resources are completely isolated.
   2. Select a flavor. The flavor defines the amount of virtual CPU, memory, and disk space that is set up in each worker node and made available to the containers. Available bare metal and virtual machines types vary by the zone in which you deploy the cluster. For more information, see [Planning your worker node setup](/docs/openshift?topic=openshift-planning_worker_nodes). After you create your cluster, you can add different flavors by adding a worker node or worker pool to the cluster.
   3. Specify the number of worker nodes that you need in the cluster. The number of workers that you enter is replicated across the number of zones that you selected. For example, if you selected two zones and want to create three worker nodes, a total of six worker nodes are provisioned in your cluster with three worker nodes in each zone.
10. Click **Create cluster**. A worker pool is created with the number of workers that you specified. You can see the progress of the worker node deployment in the **Worker nodes** tab.
    *   Your cluster might take some time to provision the OpenShift master and all worker nodes and enter a **Normal** state. Note that even if the cluster is ready, some parts of the cluster that are used by other services, such as Ingress secrets or registry image pull secrets, might still be in process. Wait until the cluster is ready before continuing to the next step by checking that the **Ingress subdomain** follows a pattern of `<cluster_name>.<region>.containers.appdomain.cloud`.
    *   Every worker node is assigned a unique worker node ID and domain name that must not be changed manually after the cluster is created. Changing the ID or domain name prevents the OpenShift master from managing your cluster.<p class="tip">Is your cluster not in a **Normal** state? Check out the [Debugging clusters](/docs/openshift?topic=openshift-cs_troubleshoot) guide for help. For example, if your cluster is provisioned in an account that is protected by a firewall gateway appliance, you must [configure your firewall settings to allow outgoing traffic to the appropriate ports and IP addresses](/docs/openshift?topic=openshift-firewall#firewall_outbound).</p>
11. After your cluster is created, you can [begin working with your cluster by configuring your CLI session](/docs/openshift?topic=openshift-access_cluster). For more possibilities, review the [Next steps](/docs/openshift?topic=openshift-clusters#next_steps).

<br />


## Creating a standard classic cluster in the CLI
{: #clusters_cli_steps}

Create your single zone or multizone classic cluster by using the {{site.data.keyword.cloud_notm}} CLI.
{: shortdesc}

**Before you begin**:
* Make sure that you complete the prerequisites to [prepare your account](#cluster_prepare) and decide on your [cluster setup](#prepare_cluster_level). Keep in mind that you need a cluster with a minimum of 2 worker nodes of the `4x16` flavor so that default OpenShift components can deploy.
* Install the {{site.data.keyword.cloud_notm}} CLI and the [Red Hat OpenShift on IBM Cloud plug-in](/docs/openshift?topic=openshift-openshift-cli).

<br>

**To create a classic cluster from the CLI**:

1. Log in to the {{site.data.keyword.cloud_notm}} CLI.
   1. Log in and enter your {{site.data.keyword.cloud_notm}} credentials when prompted. If you have a federated ID, use `ibmcloud login --sso` to log in to the {{site.data.keyword.cloud_notm}} CLI.
      ```
      ibmcloud login [--sso]
      ```
      {: pre}

   2. If you have multiple {{site.data.keyword.cloud_notm}} accounts, select the account where you want to create your Kubernetes cluster.
   3. To create clusters in a resource group other than default, target that resource group.
      * A cluster can be created in only one resource group, and after the cluster is created, you can't change its resource group.
      * You must have at least the [**Viewer** role](/docs/openshift?topic=openshift-users#platform) for the resource group.

      ```
      ibmcloud target -g <resource_group_name>
      ```
      {: pre} 

3. Review the zones where you can create your cluster. In the output of the following command, zones have a **Location Type** of `dc`. To span your cluster across zones, you must create the cluster in a [multizone-capable zone](/docs/openshift?topic=openshift-regions-and-zones#zones). Multizone-capable zones have a metro value in the **Multizone Metro** column. If you want to create a multizone cluster, you can use the {{site.data.keyword.cloud_notm}} console, or [add more zones](/docs/openshift?topic=openshift-add_workers#add_zone) to your cluster after the cluster is created.
    ```
    ibmcloud oc locations
    ```
    {: pre}
    When you select a zone that is located outside your country, keep in mind that you might require legal authorization before data can be physically stored in a foreign country.
    {: note}

4. Review the worker node flavors that are available in that zone. The flavor determines the amount of virtual CPU, memory, and disk space that is set up in each worker node and made available to your apps. Worker nodes in classic clusters can be created as virtual machines on shared or dedicated infrastructure, or as bare metal machines that are dedicated to you. For more information, see [Planning your worker node setup](/docs/openshift?topic=openshift-planning_worker_nodes). After you create your cluster, you can add different flavors by [adding a worker pool](/docs/openshift?topic=openshift-add_workers#add_pool).

   Before you create a bare metal machine, be sure that you want to provision one. Bare metal machines are billed monthly. If you order a bare metal machine by mistake, you are charged for the entire month, even if you cancel the machine immediately.  
   {:tip}

   ```
   ibmcloud oc flavors --zone <zone>
   ```
   {: pre}

5. Check if you have existing VLANs in the zones that you want to include in your cluster, and note the ID of the VLAN. If you do not have a public or private VLAN in one of the zones that you want to use in your cluster, {{site.data.keyword.containerlong_notm}} automatically creates these VLANs for you when you create the cluster.
   ```
   ibmcloud oc vlan ls --zone <zone>
   ```
   {: pre}

   Example output:
   ```
   ID        Name   Number   Type      Router
   1519999   vlan   1355     private   bcr02a.dal10
   1519898   vlan   1357     private   bcr02a.dal10
   1518787   vlan   1252     public    fcr02a.dal10
   1518888   vlan   1254     public    fcr02a.dal10
   ```
   {: screen}
   If a public and private VLAN already exist, note the matching routers. Private VLAN routers always begin with <code>bcr</code> (back-end router) and public VLAN routers always begin with <code>fcr</code> (front-end router). When you create a cluster and specify the public and private VLANs, the number and letter combination after those prefixes must match. In the example output, any private VLAN can be used with any public VLAN because the routers all include `02a.dal10`.

6. Create your standard cluster.
    ```
    ibmcloud oc cluster create classic --zone <zone> --flavor <flavor> --hardware <shared_or_dedicated> --public-vlan <public_VLAN_ID> --private-vlan <private_VLAN_ID> --workers <number> --name <cluster_name> --version <major.minor.patch>_openshift --public-service-endpoint [--private-service-endpoint] [--disable-disk-encrypt]
    ```
    {: pre}

   <table>
   <caption>`cluster create classic` command components</caption>
   <thead>
   <th colspan=2><img src="images/idea.png" alt="Idea icon"/> Understanding this command's components</th>
   </thead>
   <tbody>
   <tr>
   <td><code>cluster-create</code></td>
   <td>The command to create a cluster in your {{site.data.keyword.cloud_notm}} organization.</td>
   </tr>
   <tr>
   <td><code>--zone <em>&lt;zone&gt;</em></code></td>
   <td>Specify the {{site.data.keyword.cloud_notm}} zone ID that you chose earlier and that you want to use to create your cluster.</td>
   </tr>
   <tr>
   <td><code>--flavor <em>&lt;flavor&gt;</em></code></td>
   <td>Specify the flavor for your worker node that you chose earlier.</td>
   </tr>
   <tr>
   <td><code>--hardware <em>&lt;shared_or_dedicated&gt;</em></code></td>
   <td>Specify with the level of hardware isolation for your worker node. Use <code>dedicated</code> to have available physical resources dedicated to you only, or <code>shared</code> to allow physical resources to be shared with other IBM customers. The default is shared. This value is optional for VM standard clusters. For bare metal flavors, specify <code>dedicated</code>.</td>
   </tr>
   <tr>
   <td><code>--public-vlan <em>&lt;public_vlan_id&gt;</em></code></td>
   <td>If you already have a public VLAN set up in your IBM Cloud infrastructure account for that zone, enter the ID of the public VLAN that you retrieved earlier. If you do not have a public VLAN in your account, do not specify this option. {{site.data.keyword.containerlong_notm}} automatically creates a public VLAN for you.<p>Private VLAN routers always begin with <code>bcr</code> (back-end router) and public VLAN routers always begin with <code>fcr</code> (front-end router). When you create a cluster and specify the public and private VLANs, the number and letter combination after those prefixes must match.</p></td>
   </tr>
   <tr>
   <td><code>--private-vlan <em>&lt;private_vlan_id&gt;</em></code></td>
   <td>If you already have a private VLAN set up in your IBM Cloud infrastructure account for that zone, enter the ID of the private VLAN that you retrieved earlier. If you do not have a private VLAN in your account, do not specify this option. {{site.data.keyword.containerlong_notm}} automatically creates a private VLAN for you.<p>Private VLAN routers always begin with <code>bcr</code> (back-end router) and public VLAN routers always begin with <code>fcr</code> (front-end router). When you create a cluster and specify the public and private VLANs, the number and letter combination after those prefixes must match.</p></td>
   </tr>
   <tr>
   <td><code>--name <em>&lt;name&gt;</em></code></td>
   <td>Specify a name for your cluster. The name must start with a letter, can contain letters, numbers, and hyphen (-), and must be 35 characters or fewer. Use a name that is unique across regions. The cluster name and the region in which the cluster is deployed form the fully qualified domain name for the Ingress subdomain. To ensure that the Ingress subdomain is unique within a region, the cluster name might be truncated and appended with a random value within the Ingress domain name.
</td>
   </tr>
   <tr>
   <td><code>--workers <em>&lt;number&gt;</em></code></td>
   <td>Specify the number of worker nodes to include in the cluster. If the <code>--workers</code> option is not specified, one worker node is created.</td>
   </tr>
   <tr>
   <td><code>--version <em>&lt;major.minor.patch&gt;</em></code></td>
   <td>The OpenShift version for the cluster master node. This value is required. When the version is not specified, the cluster is created with the default supported Kubernetes version. If you do not specify a supported OpenShift version, your cluster is created as a community Kubernetes cluster. To see available versions, run <code>ibmcloud oc versions</code>.
</td>
   </tr>
   <tr>
   <td><code>--public-service-endpoint</code></td>
   <td>Enable the public service endpoint so that your OpenShift master can be accessed over the public network, for example to run `oc` commands from your terminal, and so that your OpenShift master and the worker nodes can communicate over the public VLAN. You must enable the public service endpoint, and cannot later disable it.<br><br>After you create the cluster, you can get the endpoint by running `ibmcloud oc cluster get --cluster <cluster_name_or_ID>`.</td>
   </tr>
   <tr>
   <td><code>--private-service-endpoint</code></td>
   <td>**For OpenShift 3.11 clusters only, in [VRF-enabled](/docs/resources?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud) and [service endpoint-enabled](/docs/account?topic=account-vrf-service-endpoint#service-endpoint) accounts**: Enable the private service endpoint so that your OpenShift master and the worker nodes can communicate over the private VLAN. In addition, enable the public service endpoint by using the `--public-service-endpoint` flag to access your cluster over the internet. After you enable a private service endpoint, you cannot later disable it.<br><br>After you create the cluster, you can get the endpoint by running `ibmcloud oc cluster get --cluster <cluster_name_or_ID>`.</td>   
   </tr>
   <tr>
   <td><code>--disable-disk-encrypt</code></td>
   <td>Worker nodes feature AES 256-bit [disk encryption](/docs/containers?topic=containers-security#encrypted_disk) by default. If you want to disable encryption, include this option.</td>
   </tr>
   <tr>
   <td><code><strong>--entitlement cloud_pak</strong></code></td>
   <td>Include this flag only if you use this cluster with an [IBM Cloud Pak&trade;](/docs/openshift?topic=openshift-openshift_cloud_paks) that has an OpenShift entitlement. When you specify the number of workers (`--workers`) and flavor (`--flavor`), make sure to specify only the number and size of worker nodes that you are entitled to use in [IBM Passport Advantage](https://www.ibm.com/software/passportadvantage/index.html){: external}. After your cluster is created, you are not charged the OpenShift license fee for the entitled worker nodes in the `default` worker pool.<p class="important">Do not exceed your entitlement. Keep in mind that your OpenShift Container Platform entitlements can be used with other cloud providers or in other environments. To avoid billing issues later, make sure that you use only what you are entitled to use. For example, you might have an entitlement for the OCP licenses for two worker nodes of 4 CPU and 16 GB memory, and you create this worker pool with two worker nodes of 4 CPU and 16 GB memory. You used your entire entitlement, and you cannot use the same entitlement for other worker pools, cloud providers, or environments.</p></td></tr>
   </tbody></table>

7. Verify that the creation of the cluster was requested. For virtual machines, it can take a few minutes for the worker node machines to be ordered, and for the cluster to be set up and provisioned in your account. Bare metal physical machines are provisioned by manual interaction with IBM Cloud infrastructure, and can take more than one business day to complete.
   ```
   ibmcloud oc cluster ls
   ```
   {: pre}

   When the provisioning of your OpenShift master is completed, the **State** of your cluster changes to `deployed`. After your OpenShift master is ready, the provisioning of your worker nodes is initiated.
   ```
   Name         ID                         State      Created          Workers    Zone      Version     Resource Group Name   Provider
   mycluster    blrs3b1d0p0p2f7haq0g       deployed   20170201162433   3          dal10     4.3.18_xxxx_openshift      Default             classic
   ```
   {: screen}

   Is your cluster not in a `deployed` state? Check out the [Debugging clusters](/docs/openshift?topic=openshift-cs_troubleshoot) guide for help. For example, if your cluster is provisioned in an account that is protected by a firewall gateway appliance, you must [configure your firewall settings to allow outgoing traffic to the appropriate ports and IP addresses](/docs/openshift?topic=openshift-firewall#firewall_outbound).
   {: tip}

8. Check the status of the worker nodes.
   ```
   ibmcloud oc worker ls --cluster <cluster_name_or_ID>
   ```
   {: pre}

   When the worker nodes are ready, the worker node state changes to **normal** and the status changes to **Ready**. When the node status is **Ready**, you can then access the cluster. Note that even if the cluster is ready, some parts of the cluster that are used by other services, such as Ingress secrets or registry image pull secrets, might still be in process. Note that if you created your cluster with a private VLAN only, no **Public IP** addresses are assigned to your worker nodes.
   ```
   ID                                                     Public IP        Private IP     Flavor              State    Status   Zone    Version
   kube-blrs3b1d0p0p2f7haq0g-mycluster-default-000001f7   169.xx.xxx.xxx  10.xxx.xx.xxx   u3c.2x4.encrypted   normal   Ready    dal10   1.16.9
   ```
   {: screen}

   Every worker node is assigned a unique worker node ID and domain name that must not be changed manually after the cluster is created. Changing the ID or domain name prevents the OpenShift master from managing your cluster.
   {: important}

9. **Optional**: If you created your cluster in a [multizone metro location](/docs/openshift?topic=openshift-regions-and-zones#zones), you can [spread the default worker pool across zones](/docs/openshift?topic=openshift-add_workers#add_zone) to increase the cluster's availability.

10. After your cluster is created, you can [begin working with your cluster by configuring your CLI session](/docs/openshift?topic=openshift-access_cluster).

Your cluster is ready for your workloads! You might also want to [add a tag to your cluster](/docs/openshift?topic=openshift-add_workers#cluster_tags), such as the team or billing department that uses the cluster, to help manage {{site.data.keyword.cloud_notm}} resources. For more ideas of what to do with your cluster, review the [Next steps](/docs/openshift?topic=openshift-clusters#next_steps).

<br />




<br />




## Next steps
{: #next_steps}

When the cluster is up and running, you can check out the following cluster administration tasks:
- If you created the cluster in a multizone capable zone, [spread worker nodes by adding a zone to your cluster](/docs/openshift?topic=openshift-add_workers).
- [Deploy an app in your cluster.](/docs/openshift?topic=openshift-deploy_app#app_cli)
- [Set up your own private registry in {{site.data.keyword.cloud_notm}} to store and share Docker images with other users.](/docs/Registry?topic=registry-getting-started)
- [Set up the cluster autoscaler](/docs/openshift?topic=openshift-ca#ca) to automatically add or remove worker nodes from your worker pools based on your workload resource requests.
- Control who can create pods in your cluster with [pod security policies](/docs/containers?topic=containers-psp).

Then, you can check out the following network configuration steps for your cluster setup:
  * Isolate networking workloads to edge worker nodes [in classic clusters without a gateway](/docs/openshift?topic=openshift-edge).
  * Expose your apps with [public networking services](/docs/openshift?topic=openshift-cs_network_planning#openshift_routers) or [private networking services](/docs/openshift?topic=openshift-cs_network_planning#private_access).
  * Connect your cluster with services in private networks outside of your {{site.data.keyword.cloud_notm}} account by setting up [{{site.data.keyword.cloud_notm}} Direct Link](/docs/direct-link?topic=direct-link-get-started-with-ibm-cloud-direct-link) or the [strongSwan IPSec VPN service](/docs/openshift?topic=openshift-vpn).
  * Create Calico host network policies to isolate your cluster on the [public network](/docs/openshift?topic=openshift-network_policies#isolate_workers_public) and on the [private network](/docs/openshift?topic=openshift-network_policies#isolate_workers).



