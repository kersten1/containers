---

copyright:
  years: 2014, 2019
lastupdated: "2019-10-02"

keywords: kubernetes, iks, clusters, worker nodes, worker pools, delete

subcollection: containers

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:deprecated: .deprecated}
{:download: .download}
{:preview: .preview}
{:gif: data-image-type='gif'}

# Adding worker nodes and zones to clusters
{: #add_workers}

To increase the availability of your apps, you can add worker nodes to an existing zone or multiple existing zones in your cluster. To help protect your apps from zone failures, you can add zones to your cluster.
{:shortdesc}

When you create a cluster, the worker nodes are provisioned in a worker pool. After cluster creation, you can add more worker nodes to a pool by resizing it or by adding more worker pools. By default, the worker pool exists in one zone. Clusters that have a worker pool in only one zone are called single zone clusters. When you add more zones to the cluster, the worker pool exists across the zones. Clusters that have a worker pool that is spread across more than one zone are called multizone clusters.

<p class="tip">If you have a multizone cluster, keep its worker node resources balanced. Make sure that all the worker pools are spread across the same zones, and add or remove workers by resizing the pools instead of adding individual nodes.</br></br>
After you set up your worker pool, you can [set up the cluster autoscaler](/docs/containers?topic=containers-ca#ca) to automatically add or remove worker nodes from your worker pools based on your workload resource requests.</p>

## Adding worker nodes by resizing an existing worker pool
{: #resize_pool}

You can add or reduce the number of worker nodes in your cluster by resizing an existing worker pool, regardless of whether the worker pool is in one zone or spread across multiple zones.
{: shortdesc}

For example, consider a cluster with one worker pool that has three worker nodes per zone.
* If the cluster is single zone and exists in `dal10`, then the worker pool has three worker nodes in `dal10`. The cluster has a total of three worker nodes.
* If the cluster is multizone and exists in `dal10` and `dal12`, then the worker pool has three worker nodes in `dal10` and three worker nodes in `dal12`. The cluster has a total of six worker nodes.

For bare metal worker pools, keep in mind that billing is monthly. If you resize up or down, it impacts your costs for the month.
{: tip}

Before you begin, make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).

To resize the worker pool, change the number of worker nodes that the worker pool deploys in each zone:

1. Get the name of the worker pool that you want to resize.
    ```
    ibmcloud ks worker-pool ls --cluster <cluster_name_or_ID>
    ```
    {: pre}

2. Resize the worker pool by designating the number of worker nodes that you want to deploy in each zone. The minimum value is 1.
    ```
    ibmcloud ks worker-pool resize --cluster <cluster_name_or_ID> --worker-pool <pool_name>  --size-per-zone <number_of_workers_per_zone>
    ```
    {: pre}

3. Verify that the worker pool is resized.
    ```
    ibmcloud ks worker ls --cluster <cluster_name_or_ID> --worker-pool <pool_name>
    ```
    {: pre}

    Example output for a worker pool that is in two zones, `dal10` and `dal12`, and is resized to two worker nodes per zone:
    ```
    ID                                                 Public IP        Private IP      Machine Type      State    Status  Zone    Version
    kube-dal10-crb20b637238ea471f8d4a8b881aae4962-w7   169.xx.xxx.xxx   10.xxx.xx.xxx   b3c.4x16          normal   Ready   dal10   1.14.7
    kube-dal10-crb20b637238ea471f8d4a8b881aae4962-w8   169.xx.xxx.xxx   10.xxx.xx.xxx   b3c.4x16          normal   Ready   dal10   1.14.7
    kube-dal12-crb20b637238ea471f8d4a8b881aae4962-w9   169.xx.xxx.xxx   10.xxx.xx.xxx   b3c.4x16          normal   Ready   dal12   1.14.7
    kube-dal12-crb20b637238ea471f8d4a8b881aae4962-w10  169.xx.xxx.xxx   10.xxx.xx.xxx   b3c.4x16          normal   Ready   dal12   1.14.7
    ```
    {: screen}

<br />



## Adding worker nodes in VPC clusters
{: #vpc_pools}

Add worker nodes to your VPC cluster.
{: shortdesc}

### Creating a new worker pool
{: #vpc_add_pool}

You can add worker nodes to your VPC cluster by creating a new worker pool.
{:shortdesc}

Before you begin, make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).

1. Retrieve the **VPC ID** and **Worker Zones** of your cluster and choose the zone where you want to deploy the worker nodes in your worker pool. You can choose any of the existing **Worker Zones** of your cluster, or add one of the [multizone metro locations](/docs/containers?topic=containers-regions-and-zones#zones) for the region that your cluster is in. You can list available zones by running `ibmcloud ks zone ls --provider vpc-classic`.
   ```
   ibmcloud ks cluster get --cluster <cluster_name_or_ID>
   ```
   {: pre}

   Example output:
   ```
   ...
   VPC ID:        <VPC_ID>
   ...
   Worker Zones: us-south-1, us-south-2, us-south-3
   ```
   {: screen}

2. For each zone, note the ID of VPC subnet that you want to use for the worker pool. If you do not have a VPC subnet in the zone, [create a VPC subnet](/docs/infrastructure/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference#subnet-create). VPC subnets provide IP addresses for your worker nodes and load balancer services in the cluster, so create a VPC subnet with enough IP addresses, such as 256.
   ```
   ibmcloud ks subnets --zone <zone> --provider vpc-classic --vpc-id <VPC_ID>
   ```
   {: pre}

3.  For each zone, review the [available flavors for worker nodes](/docs/containers?topic=containers-planning_worker_nodes#vm).

    ```
    ibmcloud ks flavors --zone <zone>
    ```
    {: pre}

4. Create a worker pool. Include the `--label` option to automatically label worker nodes that are in the pool with the label `key=value`.
   ```
   ibmcloud ks worker-pool create vpc-classic --name <pool_name> --cluster <cluster_name_or_ID> --flavor <flavor> --size-per-zone <number_of_workers_per_zone> --vpc-id <VPC_ID> [--label <key=value>]
   ```
   {: pre}

5. Verify that the worker pool is created.
   ```
   ibmcloud ks worker-pool ls --cluster <cluster_name_or_ID>
   ```
   {: pre}

6. By default, adding a worker pool creates a pool with no zones. To deploy worker nodes in a zone, you must add the zones that you previously retrieved to the worker pool. If you want to spread your worker nodes across multiple zones, repeat this command for each zone.
   ```
   ibmcloud ks zone add vpc-classic --zone <zone> --cluster <cluster_name_or_ID> --worker-pool <pool_name> --subnet-id <VPC_subnet_ID>
   ```
   {: pre}

7. Verify that worker nodes provision in the zone that you added. Your worker nodes are ready when the **State** changes from `provisioning` to `normal`.
   ```
   ibmcloud ks worker ls --cluster <cluster_name_or_ID> --worker-pool <pool_name>
   ```
   {: pre}

   Example output:
   ```
   ID                                                     Primary IP     Flavor   State          Status                                        Zone       Version   
   kube-<ID_string>-<cluster_name>-<pool_name>-00000002   10.xxx.xx.xxx   c2.2x4   provisioning   Infrastructure instance status is 'pending'   us-south-1   -   
   kube-<ID_string>-<cluster_name>-<pool_name>-00000003   10.xxx.xx.xxx   c2.2x4   normal   Ready   us-south-1   1.15.1_1511   
   ```
   {: screen}

### Adding a zone to a worker pool
{: #vpc_add_zone}

You can span your VPC cluster across multiple zones within one region by adding a zone to your existing worker pool.
{:shortdesc}

When you add a zone to a worker pool, the worker nodes that are defined in your worker pool are provisioned in the new zone and considered for future workload scheduling. {{site.data.keyword.containerlong_notm}} automatically adds the `failure-domain.beta.kubernetes.io/region` label for the region and the `failure-domain.beta.kubernetes.io/zone` label for the zone to each worker node. The Kubernetes scheduler uses these labels to spread pods across zones within the same region.

If you have multiple worker pools in your cluster, add the zone to all of them so that worker nodes are spread evenly across your cluster.

Before you begin, make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).

1. Get the **Location** of your cluster, and note the existing **Worker Zones** and **VPC ID**.
   ```
   ibmcloud ks cluster get --cluster <cluster_name_or_ID>
   ```
   {: pre}

   Example output:
   ```
   ...
   VPC ID:        <VPC_ID>
   Workers:       3
   Worker Zones:  us-south-1
   ...
   Location:      Dallas   
   ```
   {: screen}

2. List available zones for your cluster's location to see what other zones you can add.
   ```
   ibmcloud ks zone ls --provider vpc-classic | grep <location>
   ```
   {: pre}

3. List available VPC subnets for each zone that you want to add. If you do not have a VPC subnet in the zone, [create a VPC subnet](/docs/infrastructure/vpc-on-classic?topic=vpc-infrastructure-cli-plugin-vpc-reference#subnet-create). VPC subnets provide IP addresses for your worker nodes and load balancer services in the cluster, so create a VPC subnet with enough IP addresses, such as 256. You cannot change the number of IPs that a VPC subnet has later.
   ```
   ibmcloud ks subnets --zone <zone> --provider vpc-classic --vpc-id <VPC_ID>
   ```
   {: pre}

4. List the worker pools in your cluster and note their names.
   ```
   ibmcloud ks worker-pool ls --cluster <cluster_name_or_ID>
   ```
   {: pre}

5. Add the zone to your worker pool. Repeat this step for each zone that you want to add to your worker pool. If you have multiple worker pools, add the zone to all your worker pools so that your cluster is balanced in all zones. Include the `--worker-pool` flag for each worker pool.

   If you want to use different VPC subnets for different worker pools, repeat this command for each subnet and its corresponding worker pools. Any new worker nodes are added to the VPC subnets that you specify, but the VPC subnets for any existing worker nodes are not changed.
   {: tip}
   ```
   ibmcloud ks zone add vpc-classic --zone <zone> --cluster <cluster_name_or_ID> -w <pool_name> [-w <pool2_name>] --subnet-id
   ```
   {: pre}

6. Verify that the zone is added to your cluster. Look for the added zone in the **Worker Zones** field of the output. Note that the total number of workers in the **Workers** field has increased as new worker nodes are provisioned in the added zone.
  ```
  ibmcloud ks cluster get --cluster <cluster_name_or_ID>
  ```
  {: pre}

  Example output:
  ```
  Workers:       9
  Worker Zones:  us-south-1, us-south-2, us-south-3
  ```
  {: screen}

<br />


## Adding worker nodes in classic clusters
{: #classic_pools}

Add worker nodes to your classic cluster.
{: shortdesc}

### Creating a new worker pool
{: #add_pool}

You can add worker nodes to your classic cluster by creating a new worker pool.
{:shortdesc}

Before you begin, make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).

1. Retrieve the **Worker Zones** of your cluster and choose the zone where you want to deploy the worker nodes in your worker pool. If you have a single zone cluster, you must use the zone that you see in the **Worker Zones** field. For multizone clusters, you can choose any of the existing **Worker Zones** of your cluster, or add one of the [multizone metro locations](/docs/containers?topic=containers-regions-and-zones#zones) for the region that your cluster is in. You can list available zones by running `ibmcloud ks zone ls`.
   ```
   ibmcloud ks cluster get --cluster <cluster_name_or_ID>
   ```
   {: pre}

   Example output:
   ```
   ...
   Worker Zones: dal10, dal12, dal13
   ```
   {: screen}

2. For each zone, list available private and public VLANs. Note the private and the public VLAN that you want to use. If you do not have a private or a public VLAN, the VLAN is automatically created for you when you add a zone to your worker pool.
   ```
   ibmcloud ks vlan ls --zone <zone>
   ```
   {: pre}

3.  For each zone, review the [available flavors for worker nodes](/docs/containers?topic=containers-planning_worker_nodes#planning_worker_nodes).

    ```
    ibmcloud ks flavors --zone <zone>
    ```
    {: pre}

4. Create a worker pool. Include the `--label` option to automatically label worker nodes that are in the pool with the label `key=value`. If you provision a bare metal worker pool, specify `--hardware dedicated`.
   ```
   ibmcloud ks worker-pool create classic --name <pool_name> --cluster <cluster_name_or_ID> --machine-type <flavor> --size-per-zone <number_of_workers_per_zone>
   ```
   {: pre}

5. Verify that the worker pool is created.
   ```
   ibmcloud ks worker-pool ls --cluster <cluster_name_or_ID>
   ```
   {: pre}

6. By default, adding a worker pool creates a pool with no zones. To deploy worker nodes in a zone, you must add the zones that you previously retrieved to the worker pool. If you want to spread your worker nodes across multiple zones, repeat this command for each zone.
  ```
  ibmcloud ks zone add classic --zone <zone> --cluster <cluster_name_or_ID> --worker-pool <pool_name> --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
  ```
  {: pre}

7. Verify that worker nodes provision in the zone that you added. Your worker nodes are ready when the status changes from **provision_pending** to **normal**.
   ```
   ibmcloud ks worker ls --cluster <cluster_name_or_ID> --worker-pool <pool_name>
   ```
   {: pre}

   Example output:
   ```
   ID                                                 Public IP        Private IP      Machine Type      State    Status  Zone    Version
   kube-dal10-crb20b637238ea471f8d4a8b881aae4962-w7   169.xx.xxx.xxx   10.xxx.xx.xxx   b3c.4x16          provision_pending   Ready   dal10   1.14.7
   kube-dal10-crb20b637238ea471f8d4a8b881aae4962-w8   169.xx.xxx.xxx   10.xxx.xx.xxx   b3c.4x16          provision_pending   Ready   dal10   1.14.7
   ```
   {: screen}

<br />


### Adding a zone to a worker pool
{: #add_zone}



You can span your classic cluster across multiple zones within one region by adding a zone to your existing worker pool.
{:shortdesc}

When you add a zone to a worker pool, the worker nodes that are defined in your worker pool are provisioned in the new zone and considered for future workload scheduling. {{site.data.keyword.containerlong_notm}} automatically adds the `failure-domain.beta.kubernetes.io/region` label for the region and the `failure-domain.beta.kubernetes.io/zone` label for the zone to each worker node. The Kubernetes scheduler uses these labels to spread pods across zones within the same region.

If you have multiple worker pools in your cluster, add the zone to all of them so that worker nodes are spread evenly across your cluster.

Before you begin:
*  To add a zone to your worker pool, your worker pool must be in a [multizone-capable zone](/docs/containers?topic=containers-regions-and-zones#zones). If your worker pool is not in a multizone-capable zone, consider [creating a new worker pool](#add_pool).
*  Make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).
*  In classic clusters, if you have multiple VLANs for your cluster, multiple subnets on the same VLAN, or a multizone classic cluster, you must enable a [Virtual Router Function (VRF)](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud) for your IBM Cloud infrastructure account so your worker nodes can communicate with each other on the private network. To enable VRF, [contact your IBM Cloud infrastructure account representative](/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#how-you-can-initiate-the-conversion). To check whether a VRF is already enabled, use the `ibmcloud account show` command. If you cannot or do not want to enable VRF, enable [VLAN spanning](/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning). To perform this action, you need the **Network > Manage Network VLAN Spanning** [infrastructure permission](/docs/containers?topic=containers-users#infra_access), or you can request the account owner to enable it. To check whether VLAN spanning is already enabled, use the `ibmcloud ks vlan spanning get --region <region>` [command](/docs/containers?topic=containers-cli-plugin-kubernetes-service-cli#cs_vlan_spanning_get).

To add a zone with worker nodes to your worker pool:

1. List available zones and pick the zone that you want to add to your worker pool. The zone that you choose must be a multizone-capable zone.
   ```
   ibmcloud ks zone ls
   ```
   {: pre}

2. List available VLANs in that zone. If you do not have a private or a public VLAN, the VLAN is automatically created for you when you add a zone to your worker pool.
   ```
   ibmcloud ks vlan ls --zone <zone>
   ```
   {: pre}

3. List the worker pools in your cluster and note their names.
   ```
   ibmcloud ks worker-pool ls --cluster <cluster_name_or_ID>
   ```
   {: pre}

4. Add the zone to your worker pool. If you have multiple worker pools, add the zone to all your worker pools so that your cluster is balanced in all zones.

    A private and a public VLAN must exist before you can add a zone to multiple worker pools. If you do not have a private and a public VLAN in that zone, add the zone to one worker pool first so that a private and a public VLAN is created for you. Then, you can add the zone to other worker pools by specifying the private and the public VLAN that was created for you.
    {: note}

   If you want to use different VLANs for different worker pools, repeat this command for each VLAN and its corresponding worker pools. Any new worker nodes are added to the VLANs that you specify, but the VLANs for any existing worker nodes are not changed.
   {: tip}
   ```
   ibmcloud ks zone add classic --zone <zone> --cluster <cluster_name_or_ID> -w <pool_name> [-w <pool2_name>] --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
   ```
   {: pre}

5. Verify that the zone is added to your cluster. Look for the added zone in the **Worker zones** field of the output. Note that the total number of workers in the **Workers** field has increased as new worker nodes are provisioned in the added zone.
  ```
  ibmcloud ks cluster get --cluster <cluster_name_or_ID>
  ```
  {: pre}

  Example output:
  ```
  Name:                           mycluster
  ID:                             df253b6025d64944ab99ed63bb4567b6
  State:                          normal
  Created:                        2018-09-28T15:43:15+0000
  Location:                       dal10
  Master URL:                     https://c3.<region>.containers.cloud.ibm.com:30426
  Public Service Endpoint URL:    https://c3.<region>.containers.cloud.ibm.com:30426
  Private Service Endpoint URL:   https://c3-private.<region>.containers.cloud.ibm.com:31140
  Master Location:                Dallas
  Master Status:                  Ready (21 hours ago)
  Ingress Subdomain:              mycluster.us-south.containers.appdomain.cloud
  Ingress Secret:                 mycluster
  Workers:                        6
  Worker Zones:                   dal10, dal12
  Version:                        1.11.3_1524
  Owner:                          owner@email.com
  Resource Group ID:              a8a12accd63b437bbd6d58fb6a462ca7
  Resource Group Name:            Default
  ```
  {: screen}

<br />


## Adding worker nodes in gateway-enabled classic clusters
{: #gateway_pools}

By default, gateway-enabled classic clusters are created with a `default` pool of compute worker nodes and a `gateway` pool of gateway worker nodes. These pools are created with specific labels for their respective compute or gateway functionality. Add compute or gateway worker nodes depending on your use case.
{: shortdesc}

* Compute: If you need more compute resources to run your app workloads, [resize]#resize_pool) or [add zones](#add_gateway_zone) to the `default` compute worker pool. Only create a new compute worker pool if you need worker nodes of a different flavor than the existing compute worker nodes.
* Gateway: If you need more network throughput for your cluster, you can [resize]#resize_pool) or [add zones](#add_gateway_zone) to the `gateway` worker pool. Only create new gateway worker pool if you need worker nodes of a different flavor than the existing gateway worker nodes, which are created on shared virtual machines with the `u3c.2x4` flavor by default.

### Creating a new compute worker pool
{: #gateway_compute}

By default, gateway-enabled classic clusters are created with a `default` pool of compute worker nodes and a `gateway` pool of gateway worker nodes. After cluster creation, you can add more compute worker nodes by creating a new compute worker pool.
{:shortdesc}

Before you begin, make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).

1. Retrieve all of the existing **Worker Zones** of your cluster.
   ```
   ibmcloud ks cluster get --cluster <cluster_name_or_ID>
   ```
   {: pre}

   Example output:
   ```
   ...
   Worker Zones: dal10, dal12
   ```
   {: screen}

2. For each zone, list available private VLANs. Note the private VLAN that you want to use. If you do not have a private VLAN, the VLAN is automatically created for you when you add a zone to your worker pool.
   ```
   ibmcloud ks vlan ls --zone <zone>
   ```
   {: pre}

3. For each zone, review the [available flavors for worker nodes](/docs/containers?topic=containers-planning_worker_nodes#planning_worker_nodes).
   ```
   ibmcloud ks flavors --zone <zone>
   ```
   {: pre}

4. Create a worker pool. Ensure that you include the `--labels node-role.kubernetes.io/compute=true,ibm-cloud.kubernetes.io/private-cluster-role=worker` option to create a worker pool with compute functionality. If you provision a bare metal worker pool, specify `--hardware dedicated`.
   ```
   ibmcloud ks worker-pool create classic --cluster <cluster_name_or_ID> --name <pool_name> --machine-type <flavor> --size-per-zone <number_of_workers_per_zone> --labels node-role.kubernetes.io/compute=true,ibm-cloud.kubernetes.io/private-cluster-role=worker
   ```
   {: pre}

5. Verify that the worker pool is created.
   ```
   ibmcloud ks worker-pool ls --cluster <cluster_name_or_ID>
   ```
   {: pre}

6. By default, adding a worker pool creates a pool with no zones. To deploy compute worker nodes, you must add the zones that you retrieved in step 1 to the worker pool. Repeat this command for each zone.
  ```
  ibmcloud ks zone add classic --zone <zone> --cluster <cluster_name_or_ID> --worker-pool <pool_name> --private-vlan <private_VLAN_ID> --private-only
  ```
  {: pre}

7. Verify that worker nodes provision in the zone that you added. Your worker nodes are ready when the **State** changes from `provision_pending` to `normal`.
   ```
   ibmcloud ks worker ls --cluster <cluster_name_or_ID> --worker-pool <pool_name>
   ```
   {: pre}

   Example output:
   ```
   ID                                                     Public IP     Private IP      Machine Type      State    Status  Zone    Version
   kube-blrs3b1d0p0p2f7haq0g-mycluster-default-000001f7   -             10.xxx.xx.xxx   b3c.4x16          provision_pending   Ready   dal10   1.14.7
   kube-blrs3b1d0p0p2f7haq0g-mycluster-default-000004ea   -             10.xxx.xx.xxx   b3c.4x16          provision_pending   Ready   dal12   1.14.7
   ```
   {: screen}

### Creating a new gateway worker pool
{: #gateway_replace}

By default, gateway-enabled classic clusters are created with a `default` pool of compute worker nodes and a `gateway` pool of gateway worker nodes. If you need more network throughput for your cluster after cluster creation, you can [resize]#resize_pool) or [add zones](#add_gateway_zone) to the `gateway` worker pool. Only create new gateway worker pool if you need worker nodes of a different flavor than the existing gateway worker nodes, which are created on shared virtual machines with the `u3c.2x4` flavor by default.
{:shortdesc}

Before you begin, make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).

1. Retrieve all of the existing **Worker Zones** of your cluster.
   ```
   ibmcloud ks cluster get --cluster <cluster_name_or_ID>
   ```
   {: pre}

   Example output:
   ```
   ...
   Worker Zones: dal10, dal12
   ```
   {: screen}

2. For each zone, list available private and public VLANs. Note the private and the public VLAN that you want to use. If you do not have a private or a public VLAN, the VLAN is automatically created for you when you add a zone to your worker pool.
   ```
   ibmcloud ks vlan ls --zone <zone>
   ```
   {: pre}

3. For each zone, review the [available flavors for worker nodes](/docs/containers?topic=containers-planning_worker_nodes#planning_worker_nodes).
   ```
   ibmcloud ks flavors --zone <zone>
   ```
   {: pre}

4. Create a worker pool. Ensure that you include the `--labels dedicated=gateway,node-role.kubernetes.io/gateway=true,ibm-cloud.kubernetes.io/private-cluster-role=gateway` option of following command to create a worker pool with gateway functionality. If you provision a bare metal worker pool, specify `--hardware dedicated`.
   ```
   ibmcloud ks worker-pool create classic --cluster <cluster_name_or_ID> --name <pool_name> --machine-type <flavor> --size-per-zone <number_of_workers_per_zone> --labels dedicated=gateway,node-role.kubernetes.io/gateway=true,ibm-cloud.kubernetes.io/private-cluster-role=gateway
   ```
   {: pre}

5. Verify that the worker pool is created.
   ```
   ibmcloud ks worker-pool ls --cluster <cluster_name_or_ID>
   ```
   {: pre}

6. By default, adding a worker pool creates a pool with no zones. To deploy gateway worker nodes, you must add the zones that you retrieved in step 1 to the worker pool. Repeat this command for each zone.
  ```
  ibmcloud ks zone add classic --zone <zone> --cluster <cluster_name_or_ID> --worker-pool <pool_name> --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
  ```
  {: pre}

7. Verify that worker nodes provision in the zone that you added. Your worker nodes are ready when the **State** changes from `provision_pending` to `normal`.
   ```
   ibmcloud ks worker ls --cluster <cluster_name_or_ID> --worker-pool <pool_name>
   ```
   {: pre}

   Example output:
   ```
   ID                                                     Public IP        Private IP      Machine Type      State    Status  Zone    Version
   kube-blrs3b1d0p0p2f7haq0g-mycluster-default-000001f7   169.xx.xxx.xxx   10.xxx.xx.xxx   b3c.4x16          provision_pending   Ready   dal10   1.14.7
   kube-blrs3b1d0p0p2f7haq0g-mycluster-default-000004ea   169.xx.xxx.xxx   10.xxx.xx.xxx   b3c.4x16          provision_pending   Ready   dal12   1.14.7
   ```
   {: screen}

8. Optional: If you created a gateway worker pool to replace the default `gateway` worker pool, delete the `gateway` worker pool.
  ```
  ibmcloud ks worker-pool rm --worker-pool gateway --cluster <cluster_name_or_ID>
  ```
  {: pre}

### Adding a zone to compute and gateway worker pools
{: #add_gateway_zone}

You can span your classic gateway-enabled cluster across multiple zones within one region by adding a zone to your existing worker pools.
{:shortdesc}

When you add a zone to a worker pool, the worker nodes that are defined in your worker pool are provisioned in the new zone and considered for future workload scheduling. {{site.data.keyword.containerlong_notm}} automatically adds the `failure-domain.beta.kubernetes.io/region` label for the region and the `failure-domain.beta.kubernetes.io/zone` label for the zone to each worker node. The Kubernetes scheduler uses these labels to spread pods across zones within the same region.

If you have multiple worker pools in your cluster, add the zone to all of them so that worker nodes are spread evenly across your cluster.

Before you begin:
* To add a zone to your worker pools, your compute and gateway worker pools must be in a [multizone-capable zone](/docs/containers?topic=containers-regions-and-zones#zones). If your worker pools are not in a multizone-capable zone, consider [creating a new compute worker pool](#gateway_compute) and [creating a new gateway worker pool](#gateway_replace).
* Make sure that you have the [**Operator** or **Administrator** {{site.data.keyword.cloud_notm}} IAM platform role](/docs/openshift?topic=openshift-users#platform).

1. List available zones and pick the zone that you want to add to your worker pool. The zone that you choose must be a multizone-capable zone.
   ```
   ibmcloud ks zone ls
   ```
   {: pre}

2. List available VLANs in that zone. If you do not have a private or a public VLAN, the VLAN is automatically created for you when you add a zone to your worker pool.
   ```
   ibmcloud ks vlan ls --zone <zone>
   ```
   {: pre}

3. List the worker pools in your cluster and note their names.
   ```
   ibmcloud ks worker-pool ls --cluster <cluster_name_or_ID>
   ```
   {: pre}

4. Add the zone to your compute and gateway worker pools. Compute worker pools are connected to a private VLAN only, but gateway worker pools are connected to both a public and a private VLAN.
  1. Add the zone to your compute worker pools. If you have multiple compute worker pools, add the zone to all your worker pools so that your cluster is balanced in all zones.

      A private VLAN must exist before you can add a zone to multiple compute worker pools. If you do not have a private VLAN in that zone, add the zone to one worker pool first so that a private VLAN is created for you. Then, you can add the zone to other compute worker pools by specifying the private VLAN that was created for you.
      {: note}

     If you want to use different private VLANs for different worker pools, repeat this command for each VLAN and its corresponding worker pools. Any new worker nodes are added to the VLAN that you specify, but the VLAN for any existing worker nodes are not changed.
     {: tip}
     ```
     ibmcloud ks zone add classic --zone <zone> --cluster <cluster_name_or_ID> -p <compute_pool_name> [-p <compute_pool2_name>] --private-vlan <private_VLAN_ID> --private-only
     ```
     {: pre}

  2. Add the zone to your gateway worker pools. If you have multiple gateway worker pools, add the zone to all your worker pools so that your cluster is balanced in all zones.

      A public VLAN and private VLAN must exist before you can add a zone to multiple compute worker pools. If you do not have a public VLAN and private VLAN in that zone, add the zone to one worker pool first so that public and private VLANs are created for you. Then, you can add the zone to other compute worker pools by specifying the public VLAN and private VLAN that was created for you.
      {: note}

     If you want to use different private VLANs for different worker pools, repeat this command for each VLAN and its corresponding worker pools. Any new worker nodes are added to the VLANs that you specify, but the VLANs for any existing worker nodes are not changed.
     {: tip}
     ```
     ibmcloud ks zone add classic --zone <zone> --cluster <cluster_name_or_ID> -p <gateway_pool_name> [-p <gateway_pool2_name>] --private-vlan <private_VLAN_ID> --public-vlan <public_VLAN_ID>
     ```
     {: pre}

5. Verify that the zone is added to your cluster. Look for the added zone in the **Worker zones** field of the output. Note that the total number of workers in the **Workers** field has increased as new worker nodes are provisioned in the added zone.
  ```
  ibmcloud ks cluster get --cluster <cluster_name_or_ID>
  ```
  {: pre}

  Example output:
  ```
  ...
  Worker Zones:                   dal10, dal12
  ...
  ```
  {: screen}

<br />




## Deprecated: Adding stand-alone worker nodes
{: #standalone}

If you have a cluster that was created before worker pools were introduced, you can use the deprecated commands to add stand-alone worker nodes.
{: deprecated}

If you have a cluster that was created after worker pools were introduced, you cannot add stand-alone worker nodes. Instead, you can [create a worker pool](#add_pool), [resize an existing worker pool](#resize_pool), or [add a zone to a worker pool](#add_zone) to add worker nodes to your cluster.
{: note}

1. List available zones and pick the zone where you want to add worker nodes.
   ```
   ibmcloud ks zone ls
   ```
   {: pre}

2. List available VLANs in that zone and note their ID.
   ```
   ibmcloud ks vlan ls --zone <zone>
   ```
   {: pre}

3. List available flavors in that zone.
   ```
   ibmcloud ks flavors --zone <zone>
   ```
   {: pre}

4. Add stand-alone worker nodes to the cluster. For bare metal flavors, specify `dedicated`.
   ```
   ibmcloud ks worker add --cluster <cluster_name_or_ID> --workers <number_of_worker_nodes> --public-vlan <public_VLAN_ID> --private-vlan <private_VLAN_ID> --machine-type <flavor> --hardware <shared_or_dedicated>
   ```
   {: pre}

5. Verify that the worker nodes are created.
   ```
   ibmcloud ks worker ls --cluster <cluster_name_or_ID>
   ```
   {: pre}

<br />


## Adding labels to existing worker pools
{: #worker_pool_labels}

You can assign a worker pool a label when you [create the worker pool](#add_pool), or later by following these steps. After a worker pool is labeled, all existing and subsequent worker nodes get this label. You might use labels to deploy specific workloads only to worker nodes in the worker pool, such as [edge nodes for load balancer network traffic](/docs/containers?topic=containers-edge).
{: shortdesc}

Before you begin: [Log in to your account. If applicable, target the appropriate resource group. Set the context for your cluster.](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)

1.  List the worker pools in your cluster.
    ```
    ibmcloud ks worker-pool ls --cluster <cluster_name_or_ID>
    ```
    {: pre}
2.  To label the worker pool with a `key=value` label, use the [PATCH worker pool API ![External link icon](../icons/launch-glyph.svg "External link icon")](https://containers.cloud.ibm.com/global/swagger-global-api/#/clusters/PatchWorkerPool). Format the body of the request as in the following JSON example. <p class="important">You can also rename an existing label by assigning the same key a new value. However, do not modify the worker pool labels that are provided by default because these labels are required for worker pools to function properly. Modify only custom labels that you previously added.</p>
    ```
    {
      "labels": {"key":"value"},
      "state": "labels"
    }
    ```
    {: codeblock}
3.  **Optional**: To remove a label from a worker pool, run the [PATCH worker pool API ![External link icon](../icons/launch-glyph.svg "External link icon")](https://containers.cloud.ibm.com/global/swagger-global-api/#/clusters/PatchWorkerPool) again with the label's key field included but the value field empty.<p class="important">Do not remove the worker pool labels that are provided by default because these labels are required for worker pools to function properly. Remove only custom labels that you previously added.</p>
    ```
    {
      "labels": {"key":""},
      "state": "labels"
    }
    ```
    {: codeblock}
4.  Verify that the worker pool and worker node have the `key=value` label that you assigned.
    *   To check worker pools:
        ```
        ibmcloud ks worker-pool get --cluster <cluster_name_or_ID> --worker-pool <worker_pool_name_or_ID>
        ```
        {: pre}
    *   To check worker nodes:
        1.  List the worker nodes in the worker pool and note the **Private IP**.
            ```
            ibmcloud ks worker ls --cluster <cluster_name_or_ID> --worker-pool <worker_pool_name_or_ID>
            ```
            {: pre}
        2.  Review the **Labels** field of the output.
            ```
            kubectl describe node <worker_node_private_IP>
            ```
            {: pre}

            Example output for an added label:
            ```
            Labels:   app=test
                      arch=amd64
                      ...
            ```
            {: screen}

            Example output for a removed label (the `app=test` label is gone):
            ```
            Labels:   arch=amd64
                      ...
            ```
            {: screen}

After you label your worker pool, you can use the [label in your app deployments](/docs/containers?topic=containers-app#label) so that your workloads run on only these worker nodes, or [taints ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) to prevent deployments from running on these worker nodes.

<br />


## Autorecovery for your worker nodes
{: #planning_autorecovery}

Critical components, such as `containerd`, `kubelet`, `kube-proxy`, and `calico`, must be functional to have a healthy Kubernetes worker node. Over time these components can break and might leave your worker node in a nonfunctional state. Nonfunctional worker nodes decrease total capacity of the cluster and can result in downtime for your app.
{:shortdesc}

You can [configure health checks for your worker node and enable Autorecovery](/docs/containers?topic=containers-health#autorecovery). If Autorecovery detects an unhealthy worker node based on the configured checks, Autorecovery triggers a corrective action like an OS reload on the worker node. For more information about how Autorecovery works, see the [Autorecovery blog ![External link icon](../icons/launch-glyph.svg "External link icon")](https://www.ibm.com/cloud/blog/autorecovery-utilizes-consistent-hashing-high-availability).



