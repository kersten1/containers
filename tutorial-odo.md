---

copyright: 
  years: 2014, 2022
lastupdated: "2022-05-06"

keywords: kubernetes, odo

subcollection: containers

content-type: tutorial
services: containers, openshift
account-plan: lite
completion-time: 10m

---

{{site.data.keyword.attribute-definition-list}}



# Developing in clusters with the OpenShift Do CLI
{: #odo-tutorial}
{: toc-content-type="tutorial"}
{: toc-services="containers, openshift"}
{: toc-completion-time="10m"}

Accelerate application development on any Kubernetes cluster with the {{site.data.keyword.redhat_openshift_notm}} Do (`odo`) command-line interface (CLI) tool.
{: shortdesc}

Designed to improve the developer experience, the `odo` CLI has a simple syntax to help you get started with developing and deploying apps to a Kubernetes cluster. With version 2.0, `odo` supports any Kubernetes platform, not just OpenShift Container Platform. You can use `odo` for {{site.data.keyword.containerlong}} or {{site.data.keyword.openshiftlong}} clusters.

## Objectives
{: #odo-objectives}

- Create and deploy a Node.js app to a Kubernetes cluster by using the `odo` command line.
- Use the iterative development capabilities of `odo` to push code changes to your cluster in real time.  
- View deployment information and verify that your app is up and running.

## Audience
{: #odo-audience}

This tutorial is intended for software developers who have some familiarity with Kubernetes application development. Instead of configuring a bunch of YAML files and using the `kubectl` CLI to administer the system, you prefer a simpler syntax that is focused more around the developer experience.

## Prerequisites
{: #odo-prereqs}

* [Install the {{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cli-getting-started).
* [Install the `odo` CLI](https://docs.openshift.com/container-platform/4.6/cli_reference/developer_cli_odo/installing-odo.html){: external}.
* [Create](/docs/containers?topic=containers-getting-started) or identify an existing Kubernetes cluster to use. In the CLI, you can list clusters by running `ibmcloud ks cluster ls`.

## Create a microservice
{: #odo-new-microservice}
{: step}

After you complete the [prerequisites](#odo-prereqs) install the CLI and select a cluster, you can start creating your microservice with `odo`.
{: shortdesc}

1. [Log in to your account. If applicable, target the appropriate resource group. Set the context for your cluster.](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure)
2. Verify that the `odo` CLI is able to communicate with your cluster by listing the projects in your cluster. The projects represent Kubernetes namespaces where your apps and other microservice resources run.
    ```sh
    odo project list
    ```
    {: pre}

3. Create a project for your Node.js application. An application, or _app_, is a program that is designed for your users. The app represents the overall experience, not an individual microservice. As such, the app might consist of many microservices, each microservice referred to as a _component_ in `odo`.
    ```sh
    odo project create nodejs-test
    ```
    {: pre}

    Example output

    ```sh
    ✓  Project 'nodejs-test' is ready for use
    ✓  New project created and now using project: nodejs-test
    ```
    {: screen}

4. In your command line, create or change directories to the path where your Node.js app files are stored, or where you want to store the app files.

    If you don't have an existing Node.js app and want to download a start kit in a later step, the directory must be empty.
    {: note}

    ```sh
    cd nodejs-test-app
    ```
    {: pre}

5. Create the Node.js component in your project. The `odo` CLI prompts you to select the language, name, and project for your new component.
    ```sh
    odo create
    ```
    {: pre}

    1. Select the `nodejs` component, as shown in the following example. Example output
        ```sh
        ? Which devfile component type do you wish to create  [Use arrows to move, type to filter]
        java-openliberty
        java-quarkus
        java-springboot
        java-vertx
        > nodejs
        python
        python-django
        ```
        {: screen}

    2. Enter `nodejs-test` for the component name, as shown in the following example.
        ```sh
        ? What do you wish to name the new devfile component nodejs-test
        ```
        {: screen}

    3. Use the **Enter** key or re-enter `nodejs-test` for the project that you previously created, as shown in the following example.
        ```sh
        ? What project do you want the devfile component to be created in nodejs-test
        ```
        {: screen}

    4. Optional: If you don't have an existing Node.js app in your directory, you can download a starter kit, as shown in the following example. You can confirm the files are downloaded by running `ls` from the directory.
        ```sh
        ? Do you want to download a starter project Yes
        ```
        {: screen}

## Push a microservice to the cluster
{: #odo-push-microservice}
{: step}

Now that you have a Node.js component that is based on your local code, you can push the Node.js component to your cluster.
{: shortdesc}

You might wonder, how can the microservice be pushed to the cluster without a Dockerfile to describe the image, a YAML file to describe the Kubernetes resource, a Helm chart, or other similar configuration file? The `odo` command line uses the [{{site.data.keyword.redhat_openshift_notm}} `source-to-image`](https://github.com/openshift/source-to-image){: external} capability to generate all the configuration files that are needed to deploy the microservice to the Kubernetes cluster.

1. Push the Node.js component to your cluster. The `odo` CLI validates the Node.js component and packages the component as a deployable container.
    ```sh
    odo push
    ```
    {: pre}

    Example output

    ```sh
    Validation
    ✓  Validating the devfile [710378ns]

    Creating Kubernetes resources for component nodejs-test
    ✓  Waiting for component to start [33s]

    Applying URL changes
    ✓  URL http-3000: http://http-3000-nodejs-test-nodejs-test.<cluster_name>-<ID>.<region>.containers.appdomain.cloud/ created

    Syncing to component nodejs-test
    ✓  Checking files for pushing [2ms]
    ✓  Syncing files to the component [15s]

    Executing devfile commands for component nodejs-test
    ✓  Executing install command "npm install" [10s]
    ✓  Executing run command "npm start" [8s]

    Pushing devfile component nodejs-test
    ✓  Changes successfully pushed to component

    ```
    {: screen}

2. Verify that the component is running in your cluster. You can also view the component from your cluster's Kubernetes dashboard.
    ```sh
    kubectl get all -n nodejs-test
    ```
    {: pre}

3. Expose your Node.js microservice so that users can access the microservice from outside the cluster.
    1. Get your cluster's Ingress subdomain.
        ```sh
        ibmcloud ks cluster get -c <cluster> | grep Subdomain
        ```
        {: pre}

        Example output
        ```sh
        Ingress Subdomain: <cluster_name>-<ID>.<region>.containers.appdomain.cloud  
        ```
        {: screen}

    2. Create a URL for your microservice on port 3000 with the host Ingress subdomain. The `odo` command creates an Ingress rule to expose the pod that runs the microservice on the cluster's Ingress subdomain.
        ```sh
        odo url create --port 3000 --host <cluster_name>-<ID>.<region>.containers.appdomain.cloud
        ```
        {: pre}

    3. Push your changes to the cluster so that your URL is created.
        ```sh
        odo push
        ```
        {: pre}

4. In a web browser, open the URL of your Node.js app. You see your app, or if you used the starter kit, a message similar to the following.
    ```sh
    Hello from Node.js Starter Application!
    ```
    {: screen}

## What's next?
{: #odo-next-steps}

Now that you have a microservice running in your cluster, you might wonder what comes next.
{: shortdesc}

* **Add more microservices to your app**: You can repeat this process to build out your app by adding microservices.
* **Develop your app iteratively**: Whenever you need to make changes to your microservice, rerun the `odo push` command from the app directory to update the microservice in your cluster.

    Don't want to remember to push each time? Try the `odo watch` command to monitor for local file changes and automatically push the saved updates to your cluster.
    {: tip}

* **Learn more about OpenShift Do**: Learn more about the features of `odo` by visiting the {{site.data.keyword.redhat_openshift_notm}} Do CLI Documentation](https://docs.openshift.com/container-platform/4.9/cli_reference/developer_cli_odo/understanding-odo.html){: external}.




