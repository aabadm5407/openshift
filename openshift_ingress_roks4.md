---

copyright:
  years: 2014, 2020
lastupdated: "2020-05-14"

keywords: openshift, roks, rhoks, rhos, nginx, ingress controller

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


# Setting up Ingress in OpenShift version 4.3 or later
{: #ingress-roks4}

Expose multiple apps in your {{site.data.keyword.openshiftlong}} cluster by creating Ingress resources that are managed by the Ingress controller.
{:shortdesc}

<img src="images/icon-version-43.png" alt="Version 4.3 icon" width="30" style="width:30px; border-style: none"/> This information is for clusters that run OpenShift version 4.3 or later only. To set up Ingress for clusters that run OpenShift version 3.11, see [Setting up Ingress in OpenShift version 3.11](/docs/openshift?topic=openshift-ingress).
{: note}

## Prerequisites
{: #ingress-roks4-prereqs}

Before you get started with Ingress, review the following prerequisites.
{:shortdesc}

- Setting up Ingress requires the following [{{site.data.keyword.cloud_notm}} IAM roles](/docs/openshift?topic=openshift-users#platform):
    - **Administrator** platform role for the cluster
    - **Manager** service role in all projects
- If a zone fails, you might see intermittent failures in requests to apps that are exposed by the Ingress controller and router in that zone.
- To ensure high availability, at least two worker nodes per zone are recommended.
* Enable a [Virtual Router Function (VRF)](/docs/resources?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud) for your IBM Cloud infrastructure account. To enable VRF, [contact your IBM Cloud infrastructure account representative](/docs/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#benefits-of-moving-to-vrf). To check whether a VRF is already enabled, use the `ibmcloud account show` command. If you cannot or do not want to enable VRF, enable [VLAN spanning](/docs/vlans?topic=vlans-vlan-spanning#vlan-spanning). When a VRF or VLAN spanning is enabled, the Ingress controller can route packets to various subnets in the account.

<br />


## Planning networking for single or multiple projects
{: #multiple_projects}

One Ingress resource is required per project where you have apps that you want to expose.
{:shortdesc}

### All apps are in one project
{: #one-project}

If the apps in your cluster are all in the same project, you must create one Ingress resource to define the routing rules for the apps that you want to expose.
{: shortdesc}

For example, if you have `app1` and `app2` exposed by services in a development project, you can create an Ingress resource in the project. The resource specifies `domain.net` as the host and registers the paths that each app listens on with `domain.net`.

<img src="images/cs_ingress_single_ns.png" width="270" alt="One resource is required per project." style="width:270px; border-style: none"/>

Note that if you want to use different domains for the apps within the same project, you can create one resource per domain.

### Apps are in multiple projects
{: #multi-project}

If the apps in your cluster are in different projects, you must create one Ingress resource for each project to define the app's routing rules.
{: shortdesc}

You can either define the same domain in multiple resources, or use a wildcard domain to specify different subdomains within the Ingress resource for each project.

{: #wildcard_tls}
When a wildcard domain is registered, multiple subdomains can all resolve to the same host. The IBM-provided Ingress subdomain wildcard, `*.<cluster_name>.<globally_unique_account_HASH>-0000.<region>.containers.appdomain.cloud`, is registered by default for your cluster. The IBM-provided TLS certificate is a wildcard certificate and can be used for the wildcard subdomain. If you want to use a custom domain, you must register the custom domain as a wildcard domain such as `*.custom_domain.net`. To use TLS, you must get a wildcard certificate.
{: note}

For example, consider the following scenario in which you might want to use a wildcard subdomain:
* You have two versions of the same app, `app1` and `app3`, for testing purposes.
* You deploy the apps in two different projects within the same cluster: `app1` into the development project, and `app3` into the staging project.

To use different subdomains to manage traffic to these apps, you create the following services and resources:
* A Kubernetes service in the development project to expose `app1`.
* An Ingress resource in the development project that specifies the host as `dev.domain.net`.
* A Kubernetes service in the staging project to expose `app3`.
* An Ingress resource in the staging project that specifies the host as `stage.domain.net`.
</br>
<img src="images/cs_ingress_multi_ns.png" width="625" alt="Within a project, use subdomains in one or multiple resources" style="width:625px; border-style: none"/>

Now, both URLs resolve to the same domain. However, because the resource in the staging project is registered with the `stage` subdomain, the Ingress controller configures the router to correctly proxy requests from the `stage.domain.net/app3` URL to only `app3`.

### Multiple domains within a project
{: #multi-domains-project}

Within an individual project, you can use one domain to access all the apps in the project. If you want to use different domains for the apps within an individual project, use a wildcard domain. When a wildcard domain is registered, multiple subdomains all resolve to the same host. Then, you can use one resource to specify multiple subdomain hosts within that resource. Alternatively, you can create multiple Ingress resources in the project and specify a different subdomain in each Ingress resource.
{: shortdesc}

<img src="images/cs_ingress_single_ns_multi_subs.png" width="625" alt="One resource is required per project." style="width:625px; border-style: none"/>

The IBM-provided Ingress subdomain wildcard, `*.<cluster_name>.<globally_unique_account_HASH>-0000.<region>.containers.appdomain.cloud`, is registered by default for your cluster. The IBM-provided TLS certificate is a wildcard certificate and can be used for the wildcard subdomain. If you want to use a custom domain, you must register the custom domain as a wildcard domain such as `*.custom_domain.net`. To use TLS, you must get a wildcard certificate.
{: note}

<br />


## Exposing apps that are inside your cluster to the public
{: #ingress-roks4-public}

Expose apps that are inside your cluster to the public by using the public Ingress controller.
{:shortdesc}

**Before you begin**:
* Review the Ingress [prerequisites](#ingress-roks4-prereqs).
* [Access your OpenShift cluster](/docs/openshift?topic=openshift-access_cluster).

### Step 1: Deploy apps and create app services
{: #ingress-roks4-public-1}

Start by deploying your apps and creating Kubernetes services to expose them.
{: shortdesc}

1.  [Deploy your app to the cluster](/docs/openshift?topic=openshift-openshift_apps). Ensure that you add a label to your deployment in the metadata section of your configuration file, such as `app: code`. This label is needed to identify all pods where your app runs so that the pods can be included in the Ingress load balancing.

2.   For each app deployment that you want to expose, create a Kubernetes `ClusterIP` service. Your app must be exposed by a Kubernetes service to be included in the Ingress load balancing.
      ```
      oc expose deploy <app_deployment_name> --name my-app-svc --port <app_port> -n <namespace>
      ```
      {: pre}


### Step 2: Select an app domain and TLS termination
{: #ingress-roks4-public-2}

Choose the domain that you use to access your apps and the TLS termination for the app.
{: shortdesc}

You can use the IBM-provided domain, such as `mycluster-<hash>-0000.us-south.containers.appdomain.cloud/myapp`, to access your app from the internet. To use a custom domain instead, you can set up a CNAME record to map your custom domain to the IBM-provided domain, and configure a custom Ingress controller with the domain.

The Ingress controller load balances HTTP network traffic to the apps in your cluster. To load balance incoming HTTPS connections, you can use a TLS certificate so that the Ingress controller can decrypt the network traffic and forward the decrypted request to the apps that are exposed in your cluster.<p class="note">Currently, when you configure TLS termination for Ingress, only HTTPS connections are permitted.</p>

**To use the IBM-provided Ingress domain and TLS secret:**

Get the IBM-provided domain to use in subsequent steps.
```
ibmcloud oc cluster get --cluster <cluster_name_or_ID> | grep Ingress
```
{: pre}

Example output:
```
Ingress Subdomain:      mycluster-<hash>-0000.us-south.containers.appdomain.cloud
Ingress Secret:         mycluster-<hash>-0000
```
{: screen}

The Ingress controller for your app is already registered with the IBM-provided TLS certificate, which is stored as the `Ingress secret` in the `openshift-ingress` project. IBM-provided TLS certificates are signed by LetsEncrypt and are fully managed by IBM. The certificates expire every 90 days and are automatically renewed 37 days before they expire.

**To use a custom domain and TLS secret:**

1.  Get your custom domain ready.
    1. Work with your Domain Name Service (DNS) provider or [{{site.data.keyword.cloud_notm}} DNS](/docs/dns?topic=dns-getting-started) to register your custom domain. If you want to use different subdomains for your apps, register the custom domain as a wildcard domain, such as `*.custom_domain.net`. Note that domains are limited to 255 characters or fewer.
    2.  Define an alias for your custom domain by specifying the IBM-provided domain as a Canonical Name record (CNAME). To find the IBM-provided Ingress domain, run `ibmcloud oc cluster get --cluster <cluster_name>` and look for the **Ingress subdomain** field.

2.  If you want to configure TLS termination, get your custom TLS secret ready.
  * To use a TLS certificate that is stored in {{site.data.keyword.cloudcerts_long_notm}}:
      1. Import the certificate to your cluster. When you import a certificate, a secret that holds the TLS certificate and key is automatically created in the `ibm-cert-store` project. <p class="note">Do not create the secret with the same name as the IBM-provided Ingress secret, which you can find by running `ibmcloud oc cluster get --cluster <cluster_name_or_ID> | grep Ingress`.</p>
        ```
        ibmcloud oc alb cert deploy --secret-name <secret_name> --cluster <cluster_name_or_ID> --cert-crn <certificate_crn>
        ```
        {: pre}
      2. Copy the secret into the project where your app service is deployed.
        ```
        oc get secret <secret_name> -n ibm-cert-store -o yaml | sed 's/ibm-cert-store/<new-project>/g' | oc -n <new-project> create -f -
        ```
        {: pre}
  * To create a TLS certificate:
      1. Generate a certificate authority (CA) cert and key from your certificate provider.
          * If you have your own domain, purchase an official TLS certificate for your domain.
          * Make sure the [CN](https://support.dnsimple.com/articles/what-is-common-name/){: external} is different for each certificate.
          * If you registered a wildcard domain, generate a wildcard certificate.
          * TLS certificates that contain pre-shared keys (TLS-PSK) are not supported.
      2. Encode the cert and key into base64 and save the base64 encoded value in a new file.
        ```
        openssl base64 -in tls.key -out tls.key.base64
        ```
        {: pre}
      3. View the base64 encoded value for your cert and key.
        ```
        cat tls.key.base64
        ```
        {: pre}
      4. Create a Kubernetes secret for your certificate in the project where your app services are deployed. Do not create the secret with the same name as the IBM-provided Ingress secret, which you can find by running `ibmcloud oc cluster get --cluster <cluster_name_or_ID> | grep Ingress`.
         ```
         oc create secret tls <secret_name> -n <project> --cert=<tls.crt> --key=<tls.key>
         ```
         {: pre}
         

### Step 3: Create the Ingress resource
{: #ingress-roks4-public-3}

Ingress resources define the routing rules that the Ingress controller uses to route traffic to your app service.
{: shortdesc}

1.  Define an Ingress resource configuration file that uses the IBM-provided domain or your custom domain to route incoming network traffic to the services that you created earlier.
    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: myingressresource
    spec:
      tls:
      - hosts:
        - <custom_domain>
        secretName: <custom_secret_name>
      rules:
      - host: <domain>
        http:
          paths:
          - path: /<app1_path>
            backend:
              serviceName: <app1_service>
              servicePort: 80
          - path: /<app2_path>
            backend:
              serviceName: <app2_service>
              servicePort: 80
    ```
    {: codeblock}

    <table>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="Idea icon"/> Understanding the YAML file components</th>
    </thead>
    <tbody>
    <tr>
    <td><code>tls</code></td>
    <td><p class="note">If you use the IBM-provided Ingress subdomain, do not include a `spec.tls` section in your Ingress resource. The default Ingress controller is already registered with the IBM-provided TLS certificate, which is stored as the `Ingress secret` in the `openshift-ingress` project. The Ingress controller can access and apply the default certificate in any project where you create this Ingress resource.</p>If you use a custom domain and want to use TLS, include this TLS section in your resource with the following fields:<ul><li>Replace <em>&lt;domain&gt;</em> with your custom domain. Do not use &ast; for your host or leave the host property empty to avoid failures during Ingress creation.</li><li>Replace <em>&lt;tls_secret_name&gt;</em> with the secret that you created earlier that holds your custom TLS certificate and key.</li></ul></td>
    </tr>
    <tr>
    <td><code>host</code></td>
    <td>Replace <em>&lt;domain&gt;</em> with the IBM-provided Ingress subdomain or your custom domain.<ul><li>If your cluster has multiple projects where apps are exposed, one Ingress resource is required per project. You can use the same subdomain in each resource or different subdomains in each resource. For example, if you use a wildcard domain, you can append a wildcard subdomain to the beginning of the domain, such as `subdomain1.custom_domain.net` or `subdomain1.mycluster-<hash>-0000.us-south.containers.appdomain.cloud`.</li><li>Do not use &ast; for your host or leave the host property empty to avoid failures during Ingress creation.</li></ul></td>
    </tr>
    <tr>
    <td><code>path</code></td>
    <td>Replace <em>&lt;app_path&gt;</em> with a slash or the path that your app is listening on. The path is appended to the IBM-provided or your custom domain to create a unique route to your app. When you enter this route into a web browser, network traffic is routed to the Ingress controller. The Ingress controller looks up the associated service, and the router sends network traffic to the service. The service then forwards the traffic to the pods where the app runs.
    </br></br>
    Many apps do not listen on a specific path, but use the root path and a specific port. In this case, define the root path as <code>/</code> and do not specify an individual path for your app. Examples: <ul><li>For <code>http://domain/</code>, enter <code>/</code> as the path.</li><li>For <code>http://domain/app1_path</code>, enter <code>/app1_path</code> as the path.</li></ul></td>
    </tr>
    <tr>
    <td><code>serviceName</code></td>
    <td>Replace <em>&lt;app1_service&gt;</em> and <em>&lt;app2_service&gt;</em>, and so on, with the name of the services you created to expose your apps. If your apps are exposed by services in different projects in the cluster, include only app services that are in the same project. You must create one Ingress resource for each project where you have apps that you want to expose.</td>
    </tr>
    <tr>
    <td><code>servicePort</code></td>
    <td>The port that your service listens to. Use the same port that you defined when you created the Kubernetes service for your app.</td>
    </tr>
    </tbody></table>

3.  Create the Ingress resource for your cluster. Ensure that the resource deploys into the same project as the app services that you specified in the resource.
    ```
    oc apply -f myingressresource.yaml -n <project>
    ```
    {: pre}
4.   Verify that the Ingress resource was created successfully. If messages in the events describe an error in your resource configuration, change the values in your resource file and reapply the file for the resource.

    ```
    oc describe ingress myingressresource
    ```
    {: pre}


Your Ingress resource is created in the same project as your app services, and your apps are registered with the Ingress controller.

### Step 4: Access your app from the internet
{: #ingress-roks4-public-4}

In a web browser, enter the URL of the app service to access.
{: shortdesc}

```
https://<domain>/<app1_path>
```
{: codeblock}

If you exposed multiple apps, access those apps by changing the path that is appended to the URL.

```
https://<domain>/<app2_path>
```
{: codeblock}

If you use a wildcard domain, access those apps with their own subdomains.

```
http://<subdomain1>.<domain>/<app1_path>
```
{: codeblock}

```
http://<subdomain2>.<domain>/<app1_path>
```
{: codeblock}


<br />


## Exposing apps that are outside your cluster to the public
{: #ingress-roks4-external}

Expose apps that are outside your cluster to the public by including them in public Ingress load balancing. Incoming public requests on the IBM-provided or your custom domain are forwarded automatically to the external app.
{: shortdesc}

**Before you begin**:
* Review the Ingress [prerequisites](#ingress-roks4-prereqs).
* Ensure that the external app that you want to include into the cluster load balancing can be accessed by using a public IP address.
* [Access your OpenShift cluster](/docs/openshift?topic=openshift-access_cluster).

To expose apps that are outside of your cluster to the public:
1.  Define a Kubernetes service configuration file for the app that the Ingress controller will expose. This service that forwards incoming requests to an external endpoint that you create in subsequent steps.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: myexternalservice
    spec:
      ports:
       - protocol: TCP
         port: <app_port>
    ```
    {: codeblock}

2.  Create the service in your cluster.
    ```
    oc apply -f myexternalservice.yaml
    ```
    {: pre}

3.  Define an external endpoint configuration file. Include all public IP addresses and ports that you can use to access your external app. Note that the name of the endpoint must be the same as the name of the service that you defined in the previous step, `myexternalservice`.
    ```yaml
    kind: Endpoints
    apiVersion: v1
    metadata:
      name: myexternalservice
    subsets:
      - addresses:
          - ip: <external_IP1>
          - ip: <external_IP2>
        ports:
          - port: <external_port>
    ```
    {: codeblock}

    <table>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="Idea icon"/> Understanding the YAML file components</th>
    </thead>
    <tbody>
    <tr>
    <td><code>name</code></td>
    <td>Replace <em><code>&lt;myexternalendpoint&gt;</code></em> with the name of the Kubernetes service that you created earlier.</td>
    </tr>
    <tr>
    <td><code>ip</code></td>
    <td>Replace <em>&lt;external_IP&gt;</em> with the public IP addresses to connect to your external app.</td>
     </tr>
     <td><code>port</code></td>
     <td>Replace <em>&lt;external_port&gt;</em> with the port that your external app listens to.</td>
     </tbody></table>

4.  Create the endpoint in your cluster.
    ```
    oc apply -f myexternalendpoint.yaml
    ```
    {: pre}

5. Continue with the steps in "Exposing apps that are inside your cluster to the public", [Step 2: Select an app domain and TLS termination](#ingress-roks4-public-2).

<br />


## Exposing apps to a private network
{: #ingress-roks4-private}

Expose apps to a private network by using a private Ingress controller.
{:shortdesc}

To use private Ingress load balancing, you must first create a private Ingress controller. Then you must register your Ingress controller with a custom domain and, optionally, import your own TLS certificate.

**Before you begin**:
* Review the Ingress [prerequisites](#ingress-roks4-prereqs). 
* [Access your OpenShift cluster](/docs/openshift?topic=openshift-access_cluster).

### Step 1: Deploy apps and create app services
{: #ingress-roks4-private-1}

Start by deploying your apps and creating Kubernetes services to expose them.
{: shortdesc}

1.  [Deploy your app to the cluster](/docs/openshift?topic=openshift-openshift_apps). Ensure that you add a label to your deployment in the metadata section of your configuration file, such as `app: code`. This label is needed to identify all pods where your app runs so that the pods can be included in the Ingress load balancing.

2.   For each app deployment that you want to expose, create a Kubernetes `ClusterIP` service. Your app must be exposed by a Kubernetes service to be included in the Ingress load balancing.
      ```
      oc expose deploy <app_deployment_name> --name my-app-svc --port <app_port> -n <namespace>
      ```
      {: pre}


</br>

### Step 2: Register your custom domain and TLS certificate
{: #ingress-roks4-private-2}

When you configure the private Ingress controller, you must expose your app by using a custom domain and setting up a CNAME record to map your custom domain to the IBM-provided domain.
{: shortdesc}

The Ingress controller load balances HTTP network traffic to the apps in your cluster. To  load balance incoming HTTPS connections, you can add a TLS certificate to your Ingress resource in the next section so that the Ingress controller can decrypt the network traffic and forward the decrypted request to the apps that are exposed in your cluster.<p class="note">Currently, when you configure TLS termination for Ingress, only HTTPS connections are permitted.</p>


1.  Work with your Domain Name Service (DNS) provider or [{{site.data.keyword.cloud_notm}} DNS](/docs/dns?topic=dns-getting-started) to register your custom domain. If you want to use different subdomains for your apps, register the custom domain as a wildcard domain, such as `*.custom_domain.net`. Note that domains are limited to 255 characters or fewer.

2.  If you want to configure TLS termination, get your custom TLS secret ready.
  * To use a TLS certificate that is stored in {{site.data.keyword.cloudcerts_long_notm}}:
      1. Import the certificate to your cluster. When you import a certificate, a secret that holds the TLS certificate and key is automatically created in the `ibm-cert-store` project.<p class="note">Do not create the secret with the same name as the IBM-provided Ingress secret, which you can find by running `ibmcloud oc cluster get --cluster <cluster_name_or_ID> | grep Ingress`.</p>
        ```
        ibmcloud oc alb cert deploy --secret-name <secret_name> --cluster <cluster_name_or_ID> --cert-crn <certificate_crn>
        ```
        {: pre}
      2. Copy the secret into the project where your app service is deployed.
        ```
        oc get secret <secret_name> -n ibm-cert-store -o yaml | sed 's/ibm-cert-store/<new-project>/g' | oc -n <new-project> create -f -
        ```
        {: pre}
  * To create TLS certificate:
      1. Generate a certificate authority (CA) cert and key from your certificate provider.
          * If you have your own domain, purchase an official TLS certificate for your domain.
          * Make sure the [CN](https://support.dnsimple.com/articles/what-is-common-name/){: external} is different for each certificate.
          * If you registered a wildcard domain, generate a wildcard certificate.
          * TLS certificates that contain pre-shared keys (TLS-PSK) are not supported.
      2. Convert the cert and key into base64.
         1. Encode the cert and key into base64 and save the base64 encoded value in a new file.
            ```
            openssl base64 -in tls.key -out tls.key.base64
            ```
            {: pre}
         2. View the base64 encoded value for your cert and key.
            ```
            cat tls.key.base64
            ```
            {: pre}
      3. Create a Kubernetes secret for your certificate in the project where your app services are deployed.
           ```
           oc create secret tls <secret_name> -n <project> --cert=<tls.crt> --key=<tls.key>
           ```
           {: pre}
           Do not create the secret with the same name as the IBM-provided Ingress secret, which you can find by running `ibmcloud oc cluster get --cluster <cluster_name_or_ID> | grep Ingress`.
           {: note}

### Step 3: Create and configure a private Ingress controller
{: #ingress-roks4-private-3}

After you register your custom domain and TLS certificate, you must create a private Ingress controller and configure the controller with your domain.
{:shortdesc}

1.  Create a configuration file for a private Ingress controller.
    ```yaml
    apiVersion: operator.openshift.io/v1
    kind: IngressController
    metadata:
      name: private-ingress-controller
      namespace: openshift-ingress-operator
    spec:
      replicas: 2
      domain: <custom_domain>
      endpointPublishingStrategy:
        type: LoadBalancerService
        loadBalancer:
          scope: Internal
    ```
    {: codeblock}

2. Create the Ingress controller in your cluster. When you create the Ingress controller, a private router is automatically created and deployed based on the Ingress controller settings. Additionally, a router service is created to expose the router with an IP address.
  ```
  oc create -f private-ingress-controller.yaml -n openshift-ingress-operator
  ```
  {: pre}

3. Get the portable private IP address for the router service. In the output, look for the IP address in the **EXTERNAL IP** field for the router service that is named `router-private-ingress-controller`.
  ```
  oc get svc -n openshift-ingress
  ```
  {: pre}

  Example output:
  ```
  NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
  router-default                        LoadBalancer   172.21.128.174   169.XX.XX.XX   80:32744/TCP,443:31001/TCP   25h
  router-internal-default               ClusterIP      172.21.49.6      <none>         80/TCP,443/TCP,1936/TCP      25h
  router-private-ingress-controller     LoadBalancer   172.21.57.132    10.XX.XX.XX    80/TCP,443/TCP,1940/TCP      3m
  ```
  {: screen}

4. Work with your DNS provider to map your custom domain to the portable private IP address of the router by adding the IP address as an A record.

</br>

### Step 4: Create the Ingress resource
{: #ingress-roks4-private-4}

Ingress resources define the routing rules that the Ingress controller uses to route traffic to your app service.
{: shortdesc}

1.  Define an Ingress resource configuration file that uses the IBM-provided domain or your custom domain to route incoming network traffic to the services that you created earlier.
    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: myingressresource
    spec:
      tls:
      - hosts:
        - <custom_domain>
        secretName: <custom_secret_name>
      rules:
      - host: <domain>
        http:
          paths:
          - path: /<app1_path>
            backend:
              serviceName: <app1_service>
              servicePort: 80
          - path: /<app2_path>
            backend:
              serviceName: <app2_service>
              servicePort: 80
    ```
    {: codeblock}

    <table>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="Idea icon"/> Understanding the YAML file components</th>
    </thead>
    <tbody>
    <tr>
    <tr>
    <td><code>tls</code></td>
    <td>If you want to use TLS, include this TLS section in your resource with the following fields:<ul><li>Replace <em>&lt;domain&gt;</em> with your custom domain. Do not use &ast; for your host or leave the host property empty to avoid failures during Ingress creation.</li><li>Replace <em>&lt;tls_secret_name&gt;</em> with the secret that you created earlier that holds your custom TLS certificate and key.</li></ul></td>
    </tr>
    <td><code>host</code></td>
    <td>Replace <em>&lt;domain&gt;</em> with your custom domain.<ul><li>If your cluster has multiple projects where apps are exposed, one Ingress resource is required per project. You can use the same subdomain in each resource or different subdomains in each resource. For example, if you use a wildcard domain, you can append a wildcard subdomain to the beginning of the domain, such as `subdomain1.custom_domain.net`.</li><li>Do not use &ast; for your host or leave the host property empty to avoid failures during Ingress creation.</li></ul></td>
    </tr>
    <tr>
    <td><code>path</code></td>
    <td>Replace <em>&lt;app_path&gt;</em> with a slash or the path that your app is listening on. The path is appended to the IBM-provided or your custom domain to create a unique route to your app. When you enter this route into a web browser, network traffic is routed to the Ingress controller. The Ingress controller looks up the associated service, and the router sends network traffic to the service. The service then forwards the traffic to the pods where the app runs.
    </br></br>
    Many apps do not listen on a specific path, but use the root path and a specific port. In this case, define the root path as <code>/</code> and do not specify an individual path for your app. Examples: <ul><li>For <code>http://domain/</code>, enter <code>/</code> as the path.</li><li>For <code>http://domain/app1_path</code>, enter <code>/app1_path</code> as the path.</li></ul></td>
    </tr>
    <tr>
    <td><code>serviceName</code></td>
    <td>Replace <em>&lt;app1_service&gt;</em> and <em>&lt;app2_service&gt;</em>, and so on, with the name of the services you created to expose your apps. If your apps are exposed by services in different projects in the cluster, include only app services that are in the same project. You must create one Ingress resource for each project where you have apps that you want to expose.</td>
    </tr>
    <tr>
    <td><code>servicePort</code></td>
    <td>The port that your service listens to. Use the same port that you defined when you created the Kubernetes service for your app.</td>
    </tr>
    </tbody></table>

3.  Create the Ingress resource for your cluster. Ensure that the resource deploys into the same project as the app services that you specified in the resource.
    ```
    oc apply -f myingressresource.yaml -n <project>
    ```
    {: pre}
4.   Verify that the Ingress resource was created successfully. If messages in the events describe an error in your resource configuration, change the values in your resource file and reapply the file for the resource.

    ```
    oc describe ingress myingressresource
    ```
    {: pre}


Your Ingress resource is created in the same project as your app services, and your apps are registered with the Ingress controller.

</br>

### Step 5: Access your app from your private network
{: #ingress-roks4-private-5}

1. Before you can access your app, make sure that you can access a DNS service.  To use the default external DNS provider, you must [configure edge nodes with public access](/docs/openshift?topic=openshift-edge#edge) and [configure a Virtual Router Appliance](https://www.ibm.com/blogs/cloud-archive/2017/07/kubernetes-and-bluemix-container-based-workloads-part4/){: external}.

2. From within your private network firewall, enter the URL of the app service in a web browser.

```
https://<domain>/<app1_path>
```
{: codeblock}

If you exposed multiple apps, access those apps by changing the path that is appended to the URL.

```
https://<domain>/<app2_path>
```
{: codeblock}

If you use a wildcard domain, access those apps with their own subdomains.

```
http://<subdomain1>.<domain>/<app1_path>
```
{: codeblock}

```
http://<subdomain2>.<domain>/<app1_path>
```
{: codeblock}


<br />


## Customizing Ingress routing with annotations
{: #annotations-roks4}

If you want to customize routing rules for your app, you can use [HAProxy annotations for the OpenShift router](https://docs.openshift.com/container-platform/4.3/networking/routes/route-configuration.html#nw-route-specific-annotations_route-configuration){: external} that manages traffic for your app.

These supported annotations are in the format `haproxy.router.openshift.io/<annotation>` or `router.openshift.io/<annotation>`.</br></br>{{site.data.keyword.containerlong_notm}} annotations (`ingress.bluemix.net/<annotation>`) and NGINX annotations (`nginx.ingress.kubernetes.io/<annotation>`) are not supported for the router or the Ingress resource in OpenShift version 4.3 and later.
{: important}

To add annotations to the router:

1. List the services in the `openshift-ingress` project.
  ```
  oc get svc -n openshift-ingress
  ```
  {: pre}

2. In the output, look for the router service for the Ingress controller that you want to annotate. Choose a router service that is named in one of the following formats:
  * To annotate the router for the default Ingress controller, which is registered with your cluster's default Ingress subdomain, look for the router service that is named `router-default`. If you have a multizone cluster, note that the router service in the first zone where you have workers nodes is always named `router-default`, and router services in the zones that you subsequently add to your cluster have names such as `router-dal12`.
  * If you created a custom Ingress controller and registered it with your custom subdomain, look for the router service that is named, for example, `router-custom-ingress-controller-<hash>` or `router-private-ingress-controller-<hash>`.

3. Open the configuration for the router and add [supported HA-proxy router annotations](https://docs.openshift.com/container-platform/4.3/networking/routes/route-configuration.html#nw-route-specific-annotations_route-configuration){: external}. Repeat this process for the router in each zone.
  ```
  oc edit svc router-default -n openshift-ingress
  ```
  {: pre}

4. Save and close the file. Your changes are automatically applied.
