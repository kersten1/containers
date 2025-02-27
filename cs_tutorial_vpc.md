
---

copyright:
  years: 2014, 2019
lastupdated: "2019-10-01"

keywords: kubernetes, iks, vpc

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

# Creating a classic cluster in your Virtual Private Cloud (VPC)
{: #vpc_ks_tutorial}

With the **{{site.data.keyword.containerlong}} clusters in VPC on Classic**, you can create your cluster on classic infrastructure in the next generation of the {{site.data.keyword.cloud_notm}} platform, in your [Virtual Private Cloud](/docs/vpc-on-classic?topic=vpc-on-classic-about). VPC gives you the security of a private cloud environment with the dynamic scalability of a public cloud. VPC uses the next version of {{site.data.keyword.containerlong_notm}} [infrastructure providers](/docs/containers?topic=containers-infrastructure_providers#infrastructure_providers), with a select group of v2 API, CLI, and console functionality. You can create only standard clusters for VPC on Classic.
{: shortdesc}

## Objectives
{: #vpc_ks_objectives}

In the tutorial lessons, you create an {{site.data.keyword.containerlong_notm}} cluster in a Virtual Private Cloud (VPC). Then, you deploy an app and expose the app publicly through a load balancer.

## Time required
{: #vpc_ks_time}
30 minutes

## Audience
{: #vpc_ks_audience}

This tutorial is for administrators who are creating a cluster in {{site.data.keyword.containerlong_notm}} in VPC on Classic for the first time.
{: shortdesc}

## Prerequisites
{: #vpc_ks_prereqs}

Ensure that you have the following {{site.data.keyword.cloud_notm}} IAM access policies.
* VPC on Classic clusters: [**Administrator** platform role for VPC Infrastructure](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources).
* [**Administrator** platform role](/docs/containers?topic=containers-users#platform) for {{site.data.keyword.containerlong_notm}}.
* [**Writer** or **Manager** service role](/docs/containers?topic=containers-users#platform) for {{site.data.keyword.containerlong_notm}}.
* [**Administrator** platform role](/docs/containers?topic=containers-users#platform) for Container Registry.

<br>
Install the command-line tools.
*   [Install the {{site.data.keyword.cloud_notm}} CLI (`ibmcloud`), {{site.data.keyword.containershort_notm}}plug-in (`ibmcloud ks`), and {{site.data.keyword.registryshort_notm}} plug-in (`ibmcloud cr`)](/docs/containers?topic=containers-cs_cli_install#cs_cli_install_steps).
*   Update your {{site.data.keyword.containerlong_notm}} plug-in to the latest version.
    ```
    ibmcloud plugin update container-service
    ```
    {: pre}
*   To work with VPC, install the `infrastructure-service` plug-in. The prefix for running commands is `ibmcloud is`.
    ```
    ibmcloud plugin install infrastructure-service
    ```
    {: pre}
*   Make sure that the [`kubectl` version](/docs/containers?topic=containers-cs_cli_install#kubectl) matches the Kubernetes version of your VPC cluster. This tutorial creates a cluster that runs version 1.15.

<br />


## Lesson 1: Creating a cluster in VPC
{: #vpc_ks_create_vpc_cluster}

Create an {{site.data.keyword.containerlong_notm}} cluster in your {{site.data.keyword.cloud_notm}} Virtual Private Cloud (VPC) environment. For more information about VPC, see [Getting Started with VPC on Classic](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started).
{:shortdesc}

1.  Log in to the {{site.data.keyword.cloud_notm}} region where you want to create your VPC environment. The VPC must be set up in the same multizone metro location where you want to create your cluster. In this tutorial you create a VPC in `us-south`. For other supported regions, see [Multizone metros for VPC clusters](/docs/containers?topic=containers-regions-and-zones#zones). If you have a federated ID, include the `--sso` flag.
    ```
    ibmcloud login -r us-south [--sso]
    ```
    {: pre}
2.  Create a VPC for your cluster. For more information, see the docs for creating a VPC in the [console](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-console) or [CLI](/docs/vpc-on-classic?topic=vpc-on-classic-creating-a-vpc-using-the-ibm-cloud-cli).
    1.  Target the VPC on Classic infrastructure generation.
        ```
        ibmcloud is target --gen 1
        ```
        {: pre}
    2.  Create a VPC that is called `myvpc` and note the **ID** in the output. VPCs provide an isolated environment for your workloads to run within the public cloud. You can use the same VPC for multiple clusters, such as if you plan to have different clusters host separate microservices that need to communicate with each other. If you want to separate your clusters, such as for different departments, you can create a VPC for each cluster.
        ```
        ibmcloud is vpc-create myvpc
        ```
        {: pre}
    3.  Create a subnet for your VPC, and note its **ID**. Consider the following information when you create the VPC subnet:
        *  **Zones**: You must have one VPC subnet for each zone in your cluster. The available zones depend on the metro location that you created the VPC in. To list available zones in the region, run `ibmcloud is zone ls`.
        *  **IP addresses**: VPC subnets provide private IP addresses for your worker nodes and load balancer services in your cluster, so create a VPC subnet with enough IP addresses, such as 256. You cannot change the number of IPs that a VPC subnet has later.
        *  **Public gateways**: You do not need to attach a public gateway to complete this tutorial. Instead, you can keep your worker nodes isolated from public access by using VPC load balancers to expose workloads securely. You might attach a public gateway if your worker nodes need to access a public URL. For more information, see [Planning your cluster network setup](/docs/containers?topic=containers-plan_clusters).
        *  **Network traffic control**: Set up [network access control lists](/docs/vpc-on-classic-network?topic=vpc-on-classic-network-setting-up-network-acls) (ACLs) to control inbound and outbound network traffic at the subnet level. Each subnet includes a default ACL that permits all inbound and outbound traffic.

        ```
        ibmcloud is subnet-create mysubnet1 <vpc_ID> --zone us-south-1 --ipv4-address-count 256
        ```
        {: pre}

3.  Create a cluster in your VPC in the same zone as the subnet. By default, your cluster is created with a public and a private service endpoint. You can use the public service endpoint to access the Kubernetes master, such as to run `kubectl` commands, from your local machine. Your worker nodes can communicate with the master on the private service endpoint. For more information about the command options, see the [`cluster create vpc-classic` CLI reference docs](/docs/containers-cli-plugin?topic=containers-cli-plugin-kubernetes-service-cli#cli_cluster-create-vpc-classic).
    ```
    ibmcloud ks cluster create vpc-classic --name myvpc-cluster --zone us-south-1 --flavor b2.4x16 --workers 1 --vpc-id <vpc_ID> --subnet-id <vpc_subnet_ID>
    ```
    {: pre}
4.  Check the state of your cluster. The cluster might take a few minutes to provision.
    1.  Verify that the cluster **State** is **normal**.
        ```
        ibmcloud ks cluster ls --provider vpc-classic
        ```
        {: pre}
    2.  Download the Kubernetes configuration files.

        ```
        ibmcloud ks cluster config --cluster myvpc-cluster
        ```
        {: pre}

        When the download of the configuration files is finished, a command is displayed that you can use to set the path to the local Kubernetes configuration file as an environment variable.

        Example:

        ```
        export KUBECONFIG=/Users/<user_name>/.bluemix/plugins/kubernetes-service/clusters/<cluster_name>/kube-config-<datacenter>-<cluster_name>.yml
        ```
        {: screen}

    3.  Copy and paste the command that is displayed in your terminal to set the `KUBECONFIG` environment variable.

    4.  Verify that the `kubectl` commands run properly with your cluster by checking the Kubernetes CLI server version.

        ```
        kubectl version  --short
        ```
        {: pre}

        Example output:

        ```
        Client Version: v1.14.7
        Server Version: v1.14.7+IKS
        ```
        {: screen}

<br />


## Lesson 2: Deploying a privately available app
{: #vpc_ks_app}      

Create a Kubernetes deployment to deploy a single app instance as a pod to your worker node in your VPC cluster.
{:shortdesc}

The components that you deploy by completing this lesson are shown in the following diagram.

![Deployment setup](images/cs_app_tutorial_mz-components1.png)

To deploy the app:

1.  Clone the source code for the [Hello world app ![External link icon](../icons/launch-glyph.svg "External link icon")](https://github.com/IBM/container-service-getting-started-wt) to your user home directory. The repository contains different versions of a similar app in folders that each start with `Lab`. Each version contains the following files:
    * `Dockerfile`: The build definitions for the image.
    * `app.js`: The Hello world app.
    * `package.json`: Metadata about the app.

    ```
    git clone https://github.com/IBM/container-service-getting-started-wt.git
    ```
    {: pre}

2.  Navigate to the `Lab 1` directory.

    ```
    cd 'container-service-getting-started-wt/Lab 1'
    ```
    {: pre}

3.  Log in to the {{site.data.keyword.registryshort_notm}} CLI and note the registry region that you are in, such as `us.icr.io`.

    ```
    ibmcloud cr login
    ```
    {: pre}

    Example output:
    ```
    Logged in to 'us.icr.io'.
    ```
    {: screen}
4.  Use an existing registry namespace or create one, such as `vpc`.
    ```
    ibmcloud cr namespace-list
    ```
    {: pre}
    ```
    ibmcloud cr namespace-add vpc
    ```
    {: pre}

5.  Build a Docker image that includes the app files of the `Lab 1` directory, and push the image to the {{site.data.keyword.registryshort_notm}} namespace that you created in the previous tutorial. If you need to change the app in the future, repeat these steps to create another version of the image. **Note**: Learn more about [securing your personal information](/docs/containers?topic=containers-security#pi) when you work with container images.

    Use lowercase alphanumeric characters or underscores (`_`) only in the image name. Don't forget the period (`.`) at the end of the command. The period tells Docker to look inside the current directory for the Dockerfile and build artifacts to build the image.

    ```
    ibmcloud cr build -t us.icr.io/<namespace>/hello-world:1 .
    ```
    {: pre}

    When the build is complete, verify that you see the following success message:

    ```
    Successfully built <image_ID>
    Successfully tagged us.icr.io/<namespace>/hello-world:1
    The push refers to a repository [us.icr.io/vpc/hello-world]
    29042bc0b00c: Pushed
    f31d9ee9db57: Pushed
    33c64488a635: Pushed
    0804854a4553: Layer already exists
    6bd4a62f5178: Layer already exists
    9dfa40a0da3b: Layer already exists
    1: digest: sha256:f824e99435a29e55c25eea2ffcbb84be4b01345e0a3efbd7d9f238880d63d4a5 size: 1576
    ```
    {: screen}

6.  Create a deployment for your app. Deployments are used to manage pods, which include containerized instances of an app. The following command deploys the app in a single pod. For the purposes of this tutorial, the deployment is named **hello-world-deployment**, but you can give the deployment any name that you want.

    ```
    kubectl create deployment hello-world-deployment --image=us.icr.io/vpc/hello-world:1
    ```
    {: pre}

    Example output:

    ```
    deployment "hello-world-deployment" created
    ```
    {: screen}

    Learn more about [securing your personal information](/docs/containers?topic=containers-security#pi) when you work with Kubernetes resources.

7.  Make the app accessible by exposing the deployment as a NodePort service. Because your VPC worker nodes are connected to a private subnet only, the NodePort is assigned only a private IP address and is not exposed on the public network. Other services that run on the private network can access your app by using the private IP address of the NodePort service.

    ```
    kubectl expose deployment/hello-world-deployment --type=NodePort --name=hello-world-service --port=8080 --target-port=8080
    ```
    {: pre}

    Example output:

    ```
    service "hello-world-service" exposed
    ```
    {: screen}

    <table summary=“Information about the expose command parameters. Columns are read left to right, with the first column the command parameter and the second column the description of the parameter.”>
    <caption>More about the expose parameters</caption>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="Idea icon"/> More about the expose parameters</th>
    </thead>
    <tbody>
    <tr>
    <td><code>expose</code></td>
    <td>Expose a resource as a Kubernetes service so that users can access the resource by using the IP address of the service.</td>
    </tr>
    <tr>
    <td><code>deployment/<em>&lt;hello-world-deployment&gt;</em></code></td>
    <td>The resource type and the name of the resource to expose with this service.</td>
    </tr>
    <tr>
    <td><code>--name=<em>&lt;hello-world-service&gt;</em></code></td>
    <td>The name of the service.</td>
    </tr>
    <tr>
    <td><code>--type=NodePort</code></td>
    <td>The service type to create. In this lesson, you create a `NodePort` service. In the following lesson, you create a `LoadBalancer` service.</td>
    </tr>
    <tr>
    <td><code>--port=<em>&lt;8080&gt;</em></code></td>
    <td>The port on which the service listens for external network traffic.</td>
    </tr>
    <tr>
    <td><code>--target-port=<em>&lt;8080&gt;</em></code></td>
    <td>The port that your app listens on and to which the service directs incoming network traffic. In this example, the `target-port` is the same as the `port`, but other apps that you create might use a different port.</td>
    </tr>
    </tbody></table>

8. Now that all the deployment work is done, you can test your app from within the cluster. Get the details to form the private IP address that you can use to access your app.
    1.  Get information about the service to see which NodePort was assigned. The NodePorts are randomly assigned when they are generated with the `expose` command, but within 30000-32767. In this example, the **NodePort** is 30872.

        ```
        kubectl describe service hello-world-service
        ```
        {: pre}

        Example output:

        ```
        Name:                   hello-world-service
        Namespace:              default
        Labels:                 run=hello-world-deployment
        Selector:               run=hello-world-deployment
        Type:                   NodePort
        IP:                     10.xxx.xx.xxx
        Port:                   <unset> 8080/TCP
        NodePort:               <unset> 30872/TCP
        Endpoints:              172.30.xxx.xxx:8080
        Session Affinity:       None
        No events.
        ```
        {: screen}

    2.  List the pods that run your app, and note the pod name.
        ```
        kubectl get pods
        ```
        {: pre}

        Example output:
        ```
        NAME                                     READY     STATUS        RESTARTS   AGE
        hello-world-deployment-d99cddb45-lmj2v   1/1       Running       0          2d
        ```
        {: screen}

    3.  Describe your pod to find out what worker node the pod is running on. In the example output, the worker node that the pod runs on is **172.30.xxx.xxx**.
        ```
        kubectl describe pod hello-world-deployment-d99cddb45-lmj2v
        ```
        {: pre}

        Example output:
        ```
        Name:               hello-world-deployment-d99cddb45-lmj2v
        Namespace:          default
        Priority:           0
        PriorityClassName:  <none>
        Node:               10.xxx.xx.xxx/10.xxx.xx.xxx
        Start Time:         Mon, 22 Apr 2019 12:40:48 -0400
        Labels:             pod-template-hash=d99cddb45
                            run=hello-world-deployment
        Annotations:        kubernetes.io/psp=ibm-privileged-psp
        Status:             Running
        IP:                 172.30.xxx.xxx
        ...
        ```
        {: screen}

9.  Log in to the pod so that you can make a request to your app from within the cluster.
    ```
    kubectl exec -it hello-world-deployment-d99cddb45-lmj2v /bin/sh
    ```
    {: pre}
10. Make a request to the NodePort service by using the worker node private IP address and the node port that you previously retrieved.
    ```
    wget -O - 10.xxx.xx.xxx:30872
    ```
    {: pre}

    Example output:
    ```
    Connecting to 10.xxx.xx.xxx:30872 (10.xxx.xx.xxx:30872)
    Hello world from hello-world-deployment-d99cddb45-lmj2v! Your app is up and running in a cluster!
    -                    100% |*****************************************************************************************|    88   0:00:00 ETA
    ```
    {: screen}

    To close your pod session, enter `exit`.

<br />


## Lesson 3: Setting up a Load Balancer for VPC to expose your app publicly
{: #vpc_ks_vpc_lb}

Set up a VPC load balancer to expose your app on the public network.
{: shortdesc}

When you create a Kubernetes `LoadBalancer` service in your cluster, a load balancer for VPC is automatically created in your VPC outside of your cluster. The load balancer is multizonal and routes requests for your app through the private NodePorts that are automatically opened on your worker nodes. The following diagram illustrates how a user accesses an app's services through the load balancer, even though your worker node is connected to only a private subnet.

<img src="images/vpc_tutorial_lesson4_lb.png" width="800" alt="VPC load balancing for a cluster" style="width:600px; border-style: none"/>

1.  Create a Kubernetes `LoadBalancer` service in your cluster to publicly expose the hello world app.
    ```
    kubectl expose deployment/hello-world-deployment --type=LoadBalancer --name=hw-lb-svc  --port=8080 --target-port=8080
    ```
    {: pre}

    Example output:

    ```
    service "hw-lb-svc" exposed
    ```
    {: screen}

    <table summary=“Information about the expose command parameters. Columns are read left to right, with the first column the command parameter and the second column the description of the parameter.”>
    <caption>More about the expose parameters</caption>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="Idea icon"/> More about the expose parameters</th>
    </thead>
    <tbody>
    <tr>
    <td><code>expose</code></td>
    <td>Expose a resource as a Kubernetes service so that users can access the resource by using the VPC load balancer hostname.</td>
    </tr>
    <tr>
    <td><code>deployment/<em>&lt;hello-world-deployment&gt;</em></code></td>
    <td>The resource type and the name of the resource to expose with this service.</td>
    </tr>
    <tr>
    <td><code>--name=<em>&lt;hello-world-service&gt;</em></code></td>
    <td>The name of the service.</td>
    </tr>
    <tr>
    <td><code>--type=LoadBalancer</code></td>
    <td>The Kubernetes service type to create. In this lesson, you create a `LoadBalancer` service.</td>
    </tr>
    <tr>
    <td><code>--port=<em>&lt;8080&gt;</em></code></td>
    <td>The port on which the service listens for external network traffic.</td>
    </tr>
    <tr>
    <td><code>--target-port=<em>&lt;8080&gt;</em></code></td>
    <td>The port that your app listens on and to which the service directs incoming network traffic. In this example, the `target-port` is the same as the `port`, but other apps that you create might use a different port.</td>
    </tr>
    </tbody></table>

2.  Verify that the Kubernetes `LoadBalancer` service is created successfully in your cluster. When the Kubernetes `LoadBalancer` service is created, the **LoadBalancer Ingress** field is populated with a hostname that is assigned by the VPC load balancer that is automatically created.<p class="note">The VPC load balancer takes a few minutes to provision in your VPC. Until the VPC load balancer is ready, you cannot access the Kubernetes `LoadBalancer` service through its hostname.</p>

    ```
    kubectl describe service hw-lb-svc
    ```
    {: pre}

    Example CLI output:
    ```
    Name:                     hw-lb-svc
    Namespace:                default
    Labels:                   app=hello-world-deployment
    Annotations:              <none>
    Selector:                 app=hello-world-deployment
    Type:                     LoadBalancer
    IP:                       172.21.xxx.xxx
    LoadBalancer Ingress:     1234abcd-us-south.lb.appdomain.cloud
    Port:                     <unset> 8080/TCP
    TargetPort:               8080/TCP
    NodePort:                 <unset> 32040/TCP
    Endpoints:              
    Session Affinity:         None
    External Traffic Policy:  Cluster
    Events:
      Type    Reason                Age   From                Message
      ----    ------                ----  ----                -------
      Normal  EnsuringLoadBalancer  1m    service-controller  Ensuring load balancer
      Normal  EnsuredLoadBalancer   1m    service-controller  Ensured load balancer
    ```
    {: screen}

3.  Verify that the VPC load balancer is created successfully in your VPC. In the output, verify that the VPC load balancer has an **Operating Status** of `online` and a **Provision Status** of `active`.

    The VPC load balancer is named in the format `kube-<cluster_ID>-<kubernetes_lb_service_UID>`. To see your cluster ID, run `ibmcloud ks cluster get --cluster <cluster_name>`. To see the Kubernetes `LoadBalancer` service UID, run `kubectl get svc hw-lb-svc -o yaml` and look for the **metadata.uid** field in the output.
    {: tip}
    ```
    ibmcloud is load-balancers
    ```
    {: pre}

    In the following example CLI output, the VPC load balancer that is named `kube-bh077ne10vqpekt0domg-046e0f754d624dca8b287a033d55f96e` is created for the `hw-lb-svc` Kubernetes `LoadBalancer` service:
    ```
    ID                                     Name                                                         Created          Host Name                              Is Public   Listeners                               Operating Status   Pools                                   Private IPs              Provision Status   Public IPs                    Subnets                                Resource Group   
    06496f64-a689-4693-ba23-320959b7b677   kube-bh077ne10vqpekt0domg-046e0f754d624dca8b287a033d55f96e   8 minutes ago    1234abcd-us-south.lb.appdomain.cloud   yes         95482dcf-6b9b-4c6a-be54-04d3c46cf017    online             717f2122-5431-403c-b21d-630a12fc3a5a    10.1.1.1,10.1.1.2        active             169.1.1.1,169.1.1.2   c6540331-1c1c-40f4-9c35-aa42a98fe0d9   00809211b934565df546a95f86160f62
    ```
    {: screen}

4. Curl the hostname and port of the Kubernetes `LoadBalancer` service that is assigned by the VPC load balancer. Example:
    ```
    curl 1234abcd-us-south.lb.appdomain.cloud:8080
    ```
    {: pre}

    Example output:
    ```
    Hello world from hello-world-deployment-5fd7787c79-sl9hn! Your app is up and running in a cluster!
    ```
    {: screen}

<br />


## What's next?
{: #vpc_ks_next}

Now that you have a VPC cluster, learn more about what you can do.
{: shortdesc}

* [Setting up block storage for your apps](/docs/containers?topic=containers-vpc-block)
* [Overview of the differences between classic and VPC clusters](/docs/containers?topic=containers-infrastructure_providers)
* [VPC cluster limitations](/docs/containers?topic=containers-ibm-cloud-kubernetes-service-technology#vpc_ks_limits)
* [About the v2 API](/docs/containers?topic=containers-cs_api_install#api_about)
* [Comparison of Classic and VPC commands for the CLI](/docs/containers?topic=containers-cli-plugin-kubernetes-service-cli#cli_classic_vpc_about)

Need help, have questions, or want to give feedback on VPC clusters? Try posting in the [internal](https://ibm-argonauts.slack.com/messages/CJ58JHD9C) or [external](https://ibm-container-service.slack.com/messages/C4G6362ER) Slack channels.

If you do not use an IBMid for your {{site.data.keyword.cloud_notm}} account, [request an invitation](https://cloud.ibm.com/kubernetes/slack) to this Slack.
{: tip}
