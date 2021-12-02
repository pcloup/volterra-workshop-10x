Lab 2 - Deploy the app in Azure AKS and expose it on Volterra Global Network
############################################################################

.. image:: ../pictures/lab2/archi-lab2.png
   :align: center

Deploy an AKS in Azure
**********************

.. note:: Check your AZ CLI is up and running in the right Tenant (F5 Sales) and Subscription (EMEA-SE)

* Clone this Git to your laptop : https://github.com/f5devcentral/volterra-workshop-10x
* In the ``dev`` branch, enter to the folder ``labs-content/class3/terraform``
* Modify ``variables.tf`` according your project.

  .. warning:: Volterra does not support objects > 64 characters. So, use short name in your prefix and cluster name. 10 characters max per variable.

  .. code-block:: JSON

    variable "resource_prefix" {
    default = "Matt"
    }

    variable "cluster_name" {
    default     = "akscluster"
    description = "K8S Cluster for Matt"
    }

    variable "location" {
    default     = "westeurope"
    description = "The Azure Region in which all resources in this example should be provisioned"
    }

    variable "vm_size" {
    default     = "Standard_B2ms"
    description = "2 vpus, 8 GiB memory"
    }

* Deploy your Terraform plan

  .. code-block:: Terminal

    terraform init
    terraform plan
    terraform apply

* Get your kubeconfig file from your Azure AKS : 

  .. code-block:: Terminal

    az aks get-credentials --resource-group k8s-demo-ss --name k8s-demo-cluster-ss --file kubeconfig-ss


Publish the App without Colors microservice
*******************************************

* Use your favorite k8s client (kubectl, Lens ...) and connect to your cluster
* Deploy the 2 manifests to publish the sentence app on your AKS
  
  * Deploy labs-content/class3/k8s-deployments/aks-sentence-deployment.yaml
  * Deploy labs-content/class3/k8s-deployments/aks-sentence-deployment-nginx.yaml

.. note:: Wait few seconds, and try to connect to the Azure LB created

.. image:: ../pictures/lab2/colors-only.png
   :align: center

|

Expose the app with F5 Distributed Cloud
****************************************

* DELETE the ``labs-content/class3/k8s-deployments/aks-sentence-deployment-nginx.yaml`` manifest so that we can now push the same ``without`` a LB
* PUSH the ``labs-content/class3/k8s-deployments/aks-sentence-deployment-nginx-private.yaml`` manifest

.. note :: Check the LB service is deleted. If not, delete it manually.

.. note:: Now, Sentence app is not published externally. A voltNode is required to access the app.

Deploy a new Azure Vnet Site
============================

* First, create a new Subnet in your Vnet. The Terraform only created one Subnet (10.240.0.0/16). This subnet is our private subnet.
  
  * Create a new subnet in the same Vnet (10.241.0.0/16). Name it ``aks-subnet-public``.

* Deploy a Volterra Node (Dual NIC) and assign the existing private and public subnets from your AKS Vnet.
  
  * For the Cloud Credentials, select ``azure-emea-se`` 

* WAIT and upgrade the node from the VotlConsole if required.

|

Discover the services
=====================

* Create a service discovery
  * Select your site, upload your kubeconfig file
  * Don't forget to publish full FQDN to VIP

  .. image:: ../pictures/lab2/sd.png
     :align: center

* You should see all services + nginx as a nodeport

  .. image:: ../pictures/lab2/sd-ok.png
     :align: center

|

Create an Global Load Balancer and expose Sentence App
======================================================

* Create an Origin Pool with Nginx Frontend webserver as a member

  * Select k8s service
  * Enter service name (copy paste from service discovery)
  * Select Inside network

* Create an LB to expose the Nginx Frontend webserver

  * Domain : sentence-<myname>.emea-ent.f5demos.com
  * HTTPS
  * Select your Ogirin Pool


.. note :: Test your deployment



