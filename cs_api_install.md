---

copyright:
  years: 2014, 2019
lastupdated: "2019-09-03"

keywords: kubernetes, iks, ibmcloud, ic, ks, kubectl, api

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

# Setting up the API
{: #cs_api_install}

You can use the {{site.data.keyword.containerlong}} API to create and manage your community Kubernetes or OpenShift clusters. To use the CLI, see [Setting up the CLI](/docs/containers?topic=containers-cs_cli_install).
{:shortdesc}


## About the API
{: #api_about}

The {{site.data.keyword.containerlong_notm}} API automates the provisioning and management of {{site.data.keyword.cloud_notm}} infrastructure resources for your [Kubernetes or OpenShift clusters](/docs/containers?topic=containers-faqs#container_platforms) so that your apps have the compute, networking, and storage resources that they need to serve your users.
{:shortdesc}

The API is versioned to support the different infrastructure providers that are available for you to create clusters. For more information, see [Overview of Classic and VPC infrastructure providers](/docs/containers?topic=containers-infrastructure_providers). 

You can use the version two (`v2`) API to manage both classic and VPC on Classic clusters. The `v2` API is designed to avoid breaking existing functionality when possible. However, make sure that you review the following differences between the `v1` and `v2` API.

<table summary="The rows are read from left to right, with the area of comparison in column one, version 1 API in column two, and version 2 API in column three.">
<caption>{{site.data.keyword.containerlong_notm}} API versions</caption>
<col width="20%">
<col width="40%">
<col width="40%">
 <thead>
 <th>Area</th>
 <th>v1 API</th>
 <th>v2 API</th>
 </thead>
 <tbody>
 <tr>
   <td>API endpoint</td>
   <td>https://containers.cloud.ibm.com/global/v1</td>
   <td>https://containers.cloud.ibm.com/global/v2</td>
 </tr>
 <tr>
   <td>API reference docs</td>
   <td colspan="2">[`https://containers.cloud.ibm.com/global/swagger-global-api/` ![External link icon](../icons/launch-glyph.svg "External link icon")](https://containers.cloud.ibm.com/global/swagger-global-api/)</td>
 </tr>
 <tr>
   <td>API architectural style</td>
   <td>Representational state transfer (REST) that focuses on resources that you interact with through HTTP methods such as `GET`, `POST`, `PUT`, `PATCH`, and `DELETE`.</td>
   <td>Remote procedure calls (RPC) that focus on actions through only `GET` and `POST` HTTP methods.</td>
 </tr>
 <tr>
    <td>Supported container platforms</td>
    <td>Use the {{site.data.keyword.containerlong_notm}} API to manage your {{site.data.keyword.cloud_notm}} infrastructure resources, such as worker nodes, for **both community Kubernetes and OpenShift clusters**.</td>
    <td>VPC clusters are available for **only community Kubernetes clusters**, not OpenShift clusters. As such, provider-specific `v2` API calls against OpenShift clusters fail.</td>
 </tr>
 <tr>
  <td>Kubernetes API</td>
  <td colspan="2">To use the Kubernetes API to manage Kubernetes resources within the cluster, such as pods or namespaces, see [Working with your cluster by using the Kubernetes API](#kube_api).</td>
 </tr>
 <tr>
   <td>Supported infrastructure providers</td>
   <td>`classic`</td>
   <td>`vpc` and `classic`<ul>
   <li>The `vpc` provider is designed to support multiple VPC subproviders. The supported VPC subprovider is `vpc-classic`, which corresponds to a VPC on Classic cluster.</li>
   <li>Provider-specific requests have a path parameter in the URL, such as `v2/vpc/createCluster`. Some APIs are only available to a particular provider, such as `GET vlan` for classic or `GET vpcs` for VPC.</li>
   <li>Provider-agnostic requests can include a provider-specific body parameter that you specify, usually in JSON, such as `{"provider": "vpc"}`, if you want to return responses for only the specified provider.</li></ul></td>
 </tr>
 <tr>
   <td>`GET` responses</td>
   <td>The `GET` method for a collection of resources (such as `GET v1/clusters`) returns the same details for each resource in the list as a `GET` method for an individual resource (such as `GET v1/clusters/{idOrName}`).</td>
   <td>To return responses faster, the v2 `GET` method for a collection of resources (such as `GET v2/clusters`) returns only a subset of information that is detailed in a `GET` method for an individual resource (such as `GET v2/clusters/{idOrName}`). 
   <br><br>Some list responses include a providers property to identify whether the returned item applies to classic or VPC infrastructure. For example, the `GET zones` list returns some results such as `mon01` that are available only in the classic infrastructure provider, while other results such as `us-south-01` are available only in the VPC infrastructure provider.</td>
 </tr>
 <tr>
   <td>Cluster, worker node, and worker-pool responses</td>
   <td>Responses include only information that is specific to the classic infrastructure provider, such as the VLANs in `GET` cluster and worker responses.</td>
   <td>The information that is returned varies depending on the infrastructure provider. For such provider-specific responses, you can specify the provider in your request. For example, VPC clusters do not return VLAN information since they do not have VLANs. Instead, they return subnet and CIDR network information.</td>
 </tr>
</tbody>
</table>

<br />


## Automating cluster deployments with the API
{: #cs_api}

You can use the {{site.data.keyword.containerlong_notm}} API to automate the creation, deployment, and management of your Kubernetes clusters.
{:shortdesc}

The {{site.data.keyword.containerlong_notm}} API requires header information that you must provide in your API request and that can vary depending on the API that you want to use. To determine what header information is needed for your API, see the [{{site.data.keyword.containerlong_notm}} API documentation ![External link icon](../icons/launch-glyph.svg "External link icon")](https://us-south.containers.cloud.ibm.com/swagger-api).

To authenticate with {{site.data.keyword.containerlong_notm}}, you must provide an {{site.data.keyword.cloud_notm}} Identity and Access Management (IAM) token that is generated with your {{site.data.keyword.cloud_notm}} credentials and that includes the {{site.data.keyword.cloud_notm}} account ID where the cluster was created. Depending on the way you authenticate with {{site.data.keyword.cloud_notm}}, you can choose between the following options to automate the creation of your {{site.data.keyword.cloud_notm}} IAM token.

You can also use the [API swagger JSON file ![External link icon](../icons/launch-glyph.svg "External link icon")](https://containers.cloud.ibm.com/global/swagger-global-api/swagger.json) to generate a client that can interact with the API as part of your automation work.
{: tip}

<table summary="ID types and options with the input parameter in column 1 and the value in column 2.">
<caption>ID types and options</caption>
<thead>
<th>{{site.data.keyword.cloud_notm}} ID</th>
<th>My options</th>
</thead>
<tbody>
<tr>
<td>Unfederated ID</td>
<td><ul><li><strong>Generate an {{site.data.keyword.cloud_notm}} API key:</strong> As an alternative to using the {{site.data.keyword.cloud_notm}} user name and password, you can <a href="/docs/iam?topic=iam-userapikey#create_user_key" target="_blank">use {{site.data.keyword.cloud_notm}} API keys</a>. {{site.data.keyword.cloud_notm}} API keys are dependent on the {{site.data.keyword.cloud_notm}} account they are generated for. You cannot combine your {{site.data.keyword.cloud_notm}} API key with a different account ID in the same {{site.data.keyword.cloud_notm}} IAM token. To access clusters that were created with an account other than the one your {{site.data.keyword.cloud_notm}} API key is based on, you must log in to the account to generate a new API key.</li>
<li><strong>{{site.data.keyword.cloud_notm}} user name and password:</strong> You can follow the steps in this topic to fully automate the creation of your {{site.data.keyword.cloud_notm}} IAM access token.</li></ul>
</tr>
<tr>
<td>Federated ID</td>
<td><ul><li><strong>Generate an {{site.data.keyword.cloud_notm}} API key:</strong> <a href="/docs/iam?topic=iam-userapikey#create_user_key" target="_blank">{{site.data.keyword.cloud_notm}} API keys</a> are dependent on the {{site.data.keyword.cloud_notm}} account they are generated for. You cannot combine your {{site.data.keyword.cloud_notm}} API key with a different account ID in the same {{site.data.keyword.cloud_notm}} IAM token. To access clusters that were created with an account other than the one your {{site.data.keyword.cloud_notm}} API key is based on, you must log in to the account to generate a new API key.</li>
<li><strong>Use a one-time passcode: </strong> If you authenticate with {{site.data.keyword.cloud_notm}} by using a one-time passcode, you cannot fully automate the creation of your {{site.data.keyword.cloud_notm}} IAM token because the retrieval of your one-time passcode requires a manual interaction with your web browser. To fully automate the creation of your {{site.data.keyword.cloud_notm}} IAM token, you must create an {{site.data.keyword.cloud_notm}} API key instead.</ul></td>
</tr>
</tbody>
</table>

1.  Create your {{site.data.keyword.cloud_notm}} IAM access token. The body information that is included in your request varies based on the {{site.data.keyword.cloud_notm}} authentication method that you use.

    ```
    POST https://iam.bluemix.net/identity/token
    ```
    {: codeblock}

    <table summary="Input parameters to retrieve IAM tokens with the input parameter in column 1 and the value in column 2.">
    <caption>Input parameters to get IAM tokens.</caption>
    <thead>
        <th>Input parameters</th>
        <th>Values</th>
    </thead>
    <tbody>
    <tr>
    <td>Header</td>
    <td><ul><li>Content-Type: application/x-www-form-urlencoded</li> <li>Authorization: Basic Yng6Yng=<p><strong>Note</strong>: <code>Yng6Yng=</code> equals the URL-encoded authorization for the user name <strong>bx</strong> and the password <strong>bx</strong>.</p></li></ul>
    </td>
    </tr>
    <tr>
    <td>Body for {{site.data.keyword.cloud_notm}} user name and password</td>
    <td><ul><li>`grant_type: password`</li>
    <li>`response_type: cloud_iam uaa`</li>
    <li>`username`: Your {{site.data.keyword.cloud_notm}} user name.</li>
    <li>`password`: Your {{site.data.keyword.cloud_notm}} password.</li>
    <li>`uaa_client_id: cf`</li>
    <li>`uaa_client_secret:`</br><strong>Note</strong>: Add the <code>uaa_client_secret</code> key with no value specified.</li></ul></td>
    </tr>
    <tr>
    <td>Body for {{site.data.keyword.cloud_notm}} API keys</td>
    <td><ul><li>`grant_type: urn:ibm:params:oauth:grant-type:apikey`</li>
    <li>`response_type: cloud_iam uaa`</li>
    <li>`apikey`: Your {{site.data.keyword.cloud_notm}} API key</li>
    <li>`uaa_client_id: cf`</li>
    <li>`uaa_client_secret:` </br><strong>Note</strong>: Add the <code>uaa_client_secret</code> key with no value specified.</li></ul></td>
    </tr>
    <tr>
    <td>Body for {{site.data.keyword.cloud_notm}} one-time passcode</td>
      <td><ul><li><code>grant_type: urn:ibm:params:oauth:grant-type:passcode</code></li>
    <li>`response_type: cloud_iam uaa`</li>
    <li>`passcode`: Your {{site.data.keyword.cloud_notm}} one-time passcode. Run `ibmcloud login --sso` and follow the instructions in your CLI output to retrieve your one-time passcode by using your web browser.</li>
    <li>`uaa_client_id: cf`</li>
    <li>`uaa_client_secret:` </br><strong>Note</strong>: Add the <code>uaa_client_secret</code> key with no value specified.</li></ul>
    </td>
    </tr>
    </tbody>
    </table>

    Example output for using an API key:

    ```
    {
    "access_token": "<iam_access_token>",
    "refresh_token": "<iam_refresh_token>",
    "uaa_token": "<uaa_token>",
    "uaa_refresh_token": "<uaa_refresh_token>",
    "token_type": "Bearer",
    "expires_in": 3600,
    "expiration": 1493747503
    "scope": "ibm openid"
    }

    ```
    {: screen}

    You can find the {{site.data.keyword.cloud_notm}} IAM token in the **access_token** field of your API output. Note the {{site.data.keyword.cloud_notm}} IAM token to retrieve additional header information in the next steps.

2.  Retrieve the ID of the {{site.data.keyword.cloud_notm}} account that you want to work with. Replace `<iam_access_token>` with the {{site.data.keyword.cloud_notm}} IAM token that you retrieved from the **access_token** field of your API output in the previous step. In your API output, you can find the ID of your {{site.data.keyword.cloud_notm}} account in the **resources.metadata.guid** field.

    ```
    GET https://accountmanagement.ng.bluemix.net/v1/accounts
    ```
    {: codeblock}

    <table summary="Input parameters to get {{site.data.keyword.cloud_notm}} account ID with the input parameter in column 1 and the value in column 2.">
    <caption>Input parameters to get an {{site.data.keyword.cloud_notm}} account ID.</caption>
    <thead>
  	<th>Input parameters</th>
  	<th>Values</th>
    </thead>
    <tbody>
  	<tr>
  		<td>Headers</td>
      <td><ul><li><code>Content-Type: application/json</code></li>
        <li>`Authorization: bearer <iam_access_token>`</li>
        <li><code>Accept: application/json</code></li></ul></td>
  	</tr>
    </tbody>
    </table>

    Example output:

    ```
    {
      "total_results": 3,
      "total_pages": 1,
      "prev_url": null,
      "next_url": null,
      "resources":
        {
          "metadata": {
            "guid": "<account_ID>",
            "url": "/v1/accounts/<account_ID>",
            "created_at": "2016-01-07T18:55:09.726Z",
            "updated_at": "2017-04-28T23:46:03.739Z",
            "origin": "BSS"
    ...
    ```
    {: screen}

3.  Generate a new {{site.data.keyword.cloud_notm}} IAM token that includes your {{site.data.keyword.cloud_notm}} credentials and the account ID that you want to work with.

    If you use an {{site.data.keyword.cloud_notm}} API key, you must use the {{site.data.keyword.cloud_notm}} account ID the API key was created for. To access clusters in other accounts, log into this account and create an {{site.data.keyword.cloud_notm}} API key that is based on this account.
    {: note}

    ```
    POST https://iam.bluemix.net/identity/token
    ```
    {: codeblock}

    <table summary="Input parameters to retrieve IAM tokens with the input parameter in column 1 and the value in column 2.">
    <caption>Input parameters to get IAM tokens.</caption>
    <thead>
        <th>Input parameters</th>
        <th>Values</th>
    </thead>
    <tbody>
    <tr>
    <td>Header</td>
    <td><ul><li>`Content-Type: application/x-www-form-urlencoded`</li> <li>`Authorization: Basic Yng6Yng=`<p><strong>Note</strong>: <code>Yng6Yng=</code> equals the URL-encoded authorization for the user name <strong>bx</strong> and the password <strong>bx</strong>.</p></li></ul>
    </td>
    </tr>
    <tr>
    <td>Body for {{site.data.keyword.cloud_notm}} user name and password</td>
    <td><ul><li>`grant_type: password`</li>
    <li>`response_type: cloud_iam uaa`</li>
    <li>`username`: Your {{site.data.keyword.cloud_notm}} user name. </li>
    <li>`password`: Your {{site.data.keyword.cloud_notm}} password. </li>
    <li>`uaa_client_ID: cf`</li>
    <li>`uaa_client_secret:` </br><strong>Note</strong>: Add the <code>uaa_client_secret</code> key with no value specified.</li>
    <li>`bss_account`: The {{site.data.keyword.cloud_notm}} account ID that you retrieved in the previous step.</li></ul>
    </td>
    </tr>
    <tr>
    <td>Body for {{site.data.keyword.cloud_notm}} API keys</td>
    <td><ul><li>`grant_type: urn:ibm:params:oauth:grant-type:apikey`</li>
    <li>`response_type: cloud_iam uaa`</li>
    <li>`apikey`: Your {{site.data.keyword.cloud_notm}} API key.</li>
    <li>`uaa_client_ID: cf`</li>
    <li>`uaa_client_secret:` </br><strong>Note</strong>: Add the <code>uaa_client_secret</code> key with no value specified.</li>
    <li>`bss_account`: The {{site.data.keyword.cloud_notm}} account ID that you retrieved in the previous step.</li></ul>
      </td>
    </tr>
    <tr>
    <td>Body for {{site.data.keyword.cloud_notm}} one-time passcode</td>
    <td><ul><li>`grant_type: urn:ibm:params:oauth:grant-type:passcode`</li>
    <li>`response_type: cloud_iam uaa`</li>
    <li>`passcode`: Your {{site.data.keyword.cloud_notm}} passcode. </li>
    <li>`uaa_client_ID: cf`</li>
    <li>`uaa_client_secret:` </br><strong>Note</strong>: Add the <code>uaa_client_secret</code> key with no value specified.</li>
    <li>`bss_account`: The {{site.data.keyword.cloud_notm}} account ID that you retrieved in the previous step.</li></ul></td>
    </tr>
    </tbody>
    </table>

    Example output:

    ```
    {
      "access_token": "<iam_token>",
      "refresh_token": "<iam_refresh_token>",
      "token_type": "Bearer",
      "expires_in": 3600,
      "expiration": 1493747503
    }

    ```
    {: screen}

    You can find the {{site.data.keyword.cloud_notm}} IAM token in the **access_token** and the refresh token in the **refresh_token** field of your API output.

4.  List available {{site.data.keyword.containerlong_notm}} regions and select the region that you want to work in. Use the IAM access token and refresh token from the previous step to build your header information.
    ```
    GET https://containers.cloud.ibm.com/v1/regions
    ```
    {: codeblock}

    <table summary="Input parameters to retrieve {{site.data.keyword.containerlong_notm}} regions with the input parameter in column 1 and the value in column 2.">
    <caption>Input parameters to retrieve {{site.data.keyword.containerlong_notm}} regions.</caption>
    <thead>
    <th>Input parameters</th>
    <th>Values</th>
    </thead>
    <tbody>
    <tr>
    <td>Header</td>
    <td><ul><li>`Authorization: bearer <iam_token>`</li>
    <li>`X-Auth-Refresh-Token: <refresh_token>`</li></ul></td>
    </tr>
    </tbody>
    </table>

    Example output:
    ```
    {
    "regions": [
        {
            "name": "ap-north",
            "alias": "jp-tok",
            "cfURL": "api.au-syd.bluemix.net",
            "freeEnabled": false
        },
        {
            "name": "ap-south",
            "alias": "au-syd",
            "cfURL": "api.au-syd.bluemix.net",
            "freeEnabled": true
        },
        {
            "name": "eu-central",
            "alias": "eu-de",
            "cfURL": "api.eu-de.bluemix.net",
            "freeEnabled": true
        },
        {
            "name": "uk-south",
            "alias": "eu-gb",
            "cfURL": "api.eu-gb.bluemix.net",
            "freeEnabled": true
        },
        {
            "name": "us-east",
            "alias": "us-east",
            "cfURL": "api.ng.bluemix.net",
            "freeEnabled": false
        },
        {
            "name": "us-south",
            "alias": "us-south",
            "cfURL": "api.ng.bluemix.net",
            "freeEnabled": true
        }
    ]
    }
    ```
    {: screen}

5.  List all clusters in the {{site.data.keyword.containerlong_notm}} region that you selected. If you want to [run Kubernetes API requests against your cluster](#kube_api), make sure to note the **id** and **region** of your cluster.

     ```
     GET https://containers.cloud.ibm.com/v1/clusters
     ```
     {: codeblock}

     <table summary="Input parameters to work with the {{site.data.keyword.containerlong_notm}} API with the input parameter in column 1 and the value in column 2.">
     <caption>Input parameters to work with the {{site.data.keyword.containerlong_notm}} API.</caption>
     <thead>
     <th>Input parameters</th>
     <th>Values</th>
     </thead>
     <tbody>
     <tr>
     <td>Header</td>
     <td><ul><li>`Authorization: bearer <iam_token>`</li>
       <li>`X-Auth-Refresh-Token: <refresh_token>`</li><li>`X-Region: <region>`</li></ul></td>
     </tr>
     </tbody>
     </table>

5.  Review the [{{site.data.keyword.containerlong_notm}} API documentation ![External link icon](../icons/launch-glyph.svg "External link icon")](https://containers.cloud.ibm.com/global/swagger-global-api) to find a list of supported APIs.

<br />


## Working with your cluster by using the Kubernetes API
{: #kube_api}

You can use the [Kubernetes API ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/reference/using-api/api-overview/) to interact with your cluster in {{site.data.keyword.containerlong_notm}}.
{: shortdesc}

The following instructions require public network access in your cluster to connect to the public service endpoint of your Kubernetes master.
{: note}

1. Follow the steps in [Automating cluster deployments with the API](#cs_api) to retrieve your {{site.data.keyword.cloud_notm}} IAM access token, refresh token, the ID of the cluster where you want to run Kubernetes API requests, and the {{site.data.keyword.containerlong_notm}} region where your cluster is located.

2. Retrieve an {{site.data.keyword.cloud_notm}} IAM delegated refresh token.
   ```
   POST https://iam.bluemix.net/identity/token
   ```
   {: codeblock}

   <table summary="Input parameters to get an IAM delegated refresh token with the input parameter in column 1 and the value in column 2.">
   <caption>Input parameters to get an IAM delegated refresh token. </caption>
   <thead>
   <th>Input parameters</th>
   <th>Values</th>
   </thead>
   <tbody>
   <tr>
   <td>Header</td>
   <td><ul><li>`Content-Type: application/x-www-form-urlencoded`</li> <li>`Authorization: Basic Yng6Yng=`</br><strong>Note</strong>: <code>Yng6Yng=</code> equals the URL-encoded authorization for the user name <strong>bx</strong> and the password <strong>bx</strong>.</li><li>`cache-control: no-cache`</li></ul>
   </td>
   </tr>
   <tr>
   <td>Body</td>
   <td><ul><li>`delegated_refresh_token_expiry: 600`</li>
   <li>`receiver_client_ids: kube`</li>
   <li>`response_type: delegated_refresh_token` </li>
   <li>`refresh_token`: Your {{site.data.keyword.cloud_notm}} IAM refresh token. </li>
   <li>`grant_type: refresh_token`</li></ul></td>
   </tr>
   </tbody>
   </table>

   Example output:
   ```
   {
    "delegated_refresh_token": <delegated_refresh_token>
   }
   ```
   {: screen}

3. Retrieve an {{site.data.keyword.cloud_notm}} IAM ID, IAM access, and IAM refresh token by using the delegated refresh token from the previous step. In your API output, you can find the IAM ID token in the **id_token** field, the IAM access token in the **access_token** field, and the IAM refresh token in the **refresh_token** field.
   ```
   POST https://iam.bluemix.net/identity/token
   ```
   {: codeblock}

   <table summary="Input parameters to get IAM ID and IAM access tokens with the input parameter in column 1 and the value in column 2.">
   <caption>Input parameters to get IAM ID and IAM access tokens.</caption>
   <thead>
   <th>Input parameters</th>
   <th>Values</th>
   </thead>
   <tbody>
   <tr>
   <td>Header</td>
   <td><ul><li>`Content-Type: application/x-www-form-urlencoded`</li> <li>`Authorization: Basic a3ViZTprdWJl`</br><strong>Note</strong>: <code>a3ViZTprdWJl</code> equals the URL-encoded authorization for the user name <strong><code>kube</code></strong> and the password <strong><code>kube</code></strong>.</li><li>`cache-control: no-cache`</li></ul>
   </td>
   </tr>
   <tr>
   <td>Body</td>
   <td><ul><li>`refresh_token`: Your {{site.data.keyword.cloud_notm}} IAM delegated refresh token. </li>
   <li>`grant_type: urn:ibm:params:oauth:grant-type:delegated-refresh-token`</li></ul></td>
   </tr>
   </tbody>
   </table>

   Example output:
   ```
   {
    "access_token": "<iam_access_token>",
    "id_token": "<iam_id_token>",
    "refresh_token": "<iam_refresh_token>",
    "token_type": "Bearer",
    "expires_in": 3600,
    "expiration": 1553629664,
    "scope": "ibm openid containers-kubernetes"
   }
   ```
   {: screen}

4. Retrieve the public URL of your Kubernetes master by using the IAM access token, the IAM ID token, the IAM refresh token, and the {{site.data.keyword.containerlong_notm}} region that your cluster is in. You can find the URL in the **`publicServiceEndpointURL`** of your API output.
   ```
   GET https://containers.cloud.ibm.com/v1/clusters/<cluster_ID>
   ```
   {: codeblock}

   <table summary="Input parameters to get the public service endpoint for your Kubernetes master with the input parameter in column 1 and the value in column 2.">
   <caption>Input parameters to get the public service endpoint for your Kubernetes master.</caption>
   <thead>
   <th>Input parameters</th>
   <th>Values</th>
   </thead>
   <tbody>
   <tr>
   <td>Header</td>
     <td><ul><li>`Authorization`: Your {{site.data.keyword.cloud_notm}} IAM access token.</li><li>`X-Auth-Refresh-Token`: Your {{site.data.keyword.cloud_notm}} IAM refresh token.</li><li>`X-Region`: The {{site.data.keyword.containerlong_notm}} region of your cluster that you retrieved with the `GET https://containers.cloud.ibm.com/v1/clusters` API in [Automating cluster deployments with the API](#cs_api). </li></ul>
   </td>
   </tr>
   <tr>
   <td>Path</td>
   <td>`<cluster_ID>:` The ID of your cluster that you retrieved with the `GET https://containers.cloud.ibm.com/v1/clusters` API in [Automating cluster deployments with the API](#cs_api).      </td>
   </tr>
   </tbody>
   </table>

   Example output:
   ```
   {
    "location": "Dallas",
    "dataCenter": "dal10",
    "multiAzCapable": true,
    "vlans": null,
    "worker_vlans": null,
    "workerZones": [
        "dal10"
    ],
    "id": "1abc123b123b124ab1234a1a12a12a12",
    "name": "mycluster",
    "region": "us-south",
    ...
    "publicServiceEndpointURL": "https://c7.us-south.containers.cloud.ibm.com:27078"
   }
   ```
   {: screen}

5. Run Kubernetes API requests against your cluster by using the IAM ID token that you retrieved earlier. For example, list the Kubernetes version that runs in your cluster.

   If you enabled SSL certificate verification in your API test framework, make sure to disable this feature.
   {: tip}

   ```
   GET <publicServiceEndpointURL>/version
   ```
   {: codeblock}

   <table summary="Input parameters to view the Kubernetes version that runs in your cluster with the input parameter in column 1 and the value in column 2. ">
   <caption>Input parameters to view the Kubernetes version that runs in your cluster. </caption>
   <thead>
   <th>Input parameters</th>
   <th>Values</th>
   </thead>
   <tbody>
   <tr>
   <td>Header</td>
   <td>`Authorization: bearer <id_token>`</td>
   </tr>
   <tr>
   <td>Path</td>
   <td>`<publicServiceEndpointURL>`: The **`publicServiceEndpointURL`** of your Kubernetes master that you retrieved in the previous step.      </td>
   </tr>
   </tbody>
   </table>

   Example output:
   ```
   {
    "major": "1",
    "minor": "13",
    "gitVersion": "v1.13.4+IKS",
    "gitCommit": "c35166bd86eaa91d17af1c08289ffeab3e71e11e",
    "gitTreeState": "clean",
    "buildDate": "2019-03-21T10:08:03Z",
    "goVersion": "go1.11.5",
    "compiler": "gc",
    "platform": "linux/amd64"
   }
   ```
   {: screen}

6. Review the [Kubernetes API documentation ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/reference/kubernetes-api/) to find a list of supported APIs for the latest Kubernetes version. Make sure to use the API documentation that matches the Kubernetes version of your cluster. If you do not use the latest Kubernetes version, append your version at the end of the URL. For example, to access the API documentation for version 1.12, add `v1.12`.


## Refreshing {{site.data.keyword.cloud_notm}} IAM access tokens and obtaining new refresh tokens with the API
{: #cs_api_refresh}

Every {{site.data.keyword.cloud_notm}} Identity and Access Management (IAM) access token that is issued via the API expires after one hour. You must refresh your access token regularly to assure access to the {{site.data.keyword.cloud_notm}} API. You can use the same steps to obtain a new refresh token.
{:shortdesc}

Before you begin, make sure that you have an {{site.data.keyword.cloud_notm}} IAM refresh token or an {{site.data.keyword.cloud_notm}} API key that you can use to request a new access token.
- **Refresh token:** Follow the instructions in [Automating the cluster creation and management process with the {{site.data.keyword.cloud_notm}} API](#cs_api).
- **API key:** Retrieve your [{{site.data.keyword.cloud_notm}} ![External link icon](../icons/launch-glyph.svg "External link icon")](https://cloud.ibm.com/) API key as follows.
   1. From the menu bar, click **Manage** > **Access (IAM)**.
   2. Click the **Users** page and then select yourself.
   3. In the **API keys** pane, click **Create an IBM Cloud API key**.
   4. Enter a **Name** and **Description** for your API key and click **Create**.
   4. Click **Show** to see the API key that was generated for you.
   5. Copy the API key so that you can use it to retrieve your new {{site.data.keyword.cloud_notm}} IAM access token.

Use the following steps if you want to create an {{site.data.keyword.cloud_notm}} IAM token or if you want to obtain a new refresh token.

1.  Generate a new {{site.data.keyword.cloud_notm}} IAM access token by using the refresh token or the {{site.data.keyword.cloud_notm}} API key.
    ```
    POST https://iam.bluemix.net/identity/token
    ```
    {: codeblock}

    <table summary="Input parameters for new IAM token with the input parameter in column 1 and the value in column 2.">
    <caption>Input parameters for a new {{site.data.keyword.cloud_notm}} IAM token</caption>
    <thead>
    <th>Input parameters</th>
    <th>Values</th>
    </thead>
    <tbody>
    <tr>
    <td>Header</td>
    <td><ul><li>`Content-Type: application/x-www-form-urlencoded`</li>
      <li>`Authorization: Basic Yng6Yng=`</br></br><strong>Note:</strong> <code>Yng6Yng=</code> equals the URL-encoded authorization for the user name <strong>bx</strong> and the password <strong>bx</strong>.</li></ul></td>
    </tr>
    <tr>
    <td>Body when using the refresh token</td>
    <td><ul><li>`grant_type: refresh_token`</li>
    <li>`response_type: cloud_iam uaa`</li>
    <li>`refresh_token:` Your {{site.data.keyword.cloud_notm}} IAM refresh token. </li>
    <li>`uaa_client_ID: cf`</li>
    <li>`uaa_client_secret:`</li>
    <li>`bss_account:` Your {{site.data.keyword.cloud_notm}} account ID. </li></ul><strong>Note</strong>: Add the <code>uaa_client_secret</code> key with no value specified.</td>
    </tr>
    <tr>
      <td>Body when using the {{site.data.keyword.cloud_notm}} API key</td>
      <td><ul><li>`grant_type: urn:ibm:params:oauth:grant-type:apikey`</li>
    <li>`response_type: cloud_iam uaa`</li>
    <li>`apikey:` Your {{site.data.keyword.cloud_notm}} API key. </li>
    <li>`uaa_client_ID: cf`</li>
        <li>`uaa_client_secret:`</li></ul><strong>Note:</strong> Add the <code>uaa_client_secret</code> key with no value specified.</td>
    </tr>
    </tbody>
    </table>

    Example API output:

    ```
    {
      "access_token": "<iam_token>",
      "refresh_token": "<iam_refresh_token>",
      "uaa_token": "<uaa_token>",
      "uaa_refresh_token": "<uaa_refresh_token>",
      "token_type": "Bearer",
      "expires_in": 3600,
      "expiration": 1493747503
    }

    ```
    {: screen}

    You can find your new {{site.data.keyword.cloud_notm}} IAM token in the **access_token**, and the refresh token in the **refresh_token** field of your API output.

2.  Continue working with the [{{site.data.keyword.containerlong_notm}} API documentation ![External link icon](../icons/launch-glyph.svg "External link icon")](https://containers.cloud.ibm.com/global/swagger-global-api) by using the token from the previous step.

<br />


## Refreshing {{site.data.keyword.cloud_notm}} IAM access tokens and obtaining new refresh tokens with the CLI
{: #cs_cli_refresh}

When you start a new CLI session, or if 24 hours has expired in your current CLI session, you must set the context for your cluster by running `ibmcloud ks cluster config --cluster <cluster_name>`. When you set the context for your cluster with this command, the `kubeconfig` file for your Kubernetes cluster is downloaded. Additionally, an {{site.data.keyword.cloud_notm}} Identity and Access Management (IAM) ID token and a refresh token are issued to provide authentication.
{: shortdesc}

**ID token**: Every IAM ID token that is issued via the CLI expires after one hour. When the ID token expires, the refresh token is sent to the token provider to refresh the ID token. Your authentication is refreshed, and you can continue to run commands against your cluster.

**Refresh token**: Refresh tokens expire every 30 days. If the refresh token is expired, the ID token cannot be refreshed, and you are not able to continue running commands in the CLI. You can get a new refresh token by running `ibmcloud ks cluster config --cluster <cluster_name>`. This command also refreshes your ID token.
