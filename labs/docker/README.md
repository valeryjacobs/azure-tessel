Using Docker on Linux to host a REST API and connect to it from Tessel
============
Docker is a tool to manage Linux containers. Containers are processes that have isolated storage, networking and compute resources. Docker is the client tool to create, extend, run and deploy containers. Containers offer a runtime for web applications, services, databases, whatever you can run on Linux (and soon also on Windows). 

Containers are run from Docker images (just like a VM has as a source image file). You can write setup files called Dockerfiles that instruct Docker which steps and commands it should run to build up the image.

Why care about Containers and Docker? Docker makes it very simple to setup environments and make efficient use of the resource a VM has to offer. Once you have a host VM running you can run any number of containers on it that can startup very fast (think sub-second!).

Why would I want to run my host VM on Azure?
With Microsoft Azure you get a robust platform that let's you (auto)scale you deployments to any size you need with first class tooling (like Powershell, Puppet, Chef, DSC etc.) and high availability features build right in for both storage and compute. With Azure you also setup for a 99,95% SLA.

So the three selling point for Docker are:
* Simplified deployment: Make your images and distribute them within your organization to run on any machine on any platform that has a modern Linux kernel running, again, a Windows version is coming.
* Efficient use of resources: Instead of using a VMs as the computing unit we can host multiple Containers in a VM that each act as isolated pseudo VMs.
* Since the VM kernel is shared amongst containers they can run instantly and have a very small footprint needing only your application's files and the packages that it depends on.

Microsoft Azure has full blown support for Linux VM's and therefore is an excellent environment to host containers. We have built-in support for docker in our cross platform command line tools so setting up Docker hosts on Azure is very easy.

In this lab we will go through the proces of running a Docker client Linux VM in Azure (or you can use your local Linux environment if have have one already) and provisioning Docker host VM's in which we will run Docker containers. In one of these containers we will run the REST API service created in the 'Websites' lab.

Prerequisites
-------------
In order to successfully complete this lab you need to:

* Have successfully setup your Azure Subscription, your development environment and your Tessel according to instructions outlined in the [Setup Lab](../_setup).
* Optionally: Have your own Linux machine or Linux VM available if you prefer a local client.

Instructions
------------
### Setup a client VM
* Create Linux VM.
![Portal Screenshot](images/portal.png)
* Install Node, NPM and the xplat CLI
* Install Docker
* Confirm that Docker is installed buy running 'sudo docker version' (just run 'sudo docker' to see all the commands supported).


### Provision a container host in Azure
Now we have the client tools up and running we want to provision a VM that will act as our Container host. You could also run the Containers locally ofcourse, but in this lab we want to leverage the power of Azure to handle that task on potentialy huge numbers of VM's ranging from small (and very cheap) to mega ships of containers, that's where Docker got its name from.

* Check the installation of the x-plat CLI tools by typing 'azure' in the client console.
* Setup the certificate stuff
* List available Ubuntu images by running 'azure vm image list | grep 14_04'. We filter the list so we only see the latest 14.04 versions of the Ubuntu LTS release that are available in azure.
* Copy the image name of the latest daily build, we will this in our next command the base the container host VM on.
* Enter 'azure vm docker create -e 22 -l 'West Europe' vmhostname "vmimagename". The 'docker' option used in this command instruct Azure to prefit the VM with the Docker components and a docker daemon (background service). -e is the endpoint on port 22, -l is the location 'West Europe is the closest to our location here in Berlin.

After a couple of minutes, we have our host VM running,a storage account for the host VM VHD file, and the certificates for running the Daemon and have it listen to port 4243.


#### Building & running a container image 
After a couple of minutes, we have our host VM running,a storage account for the host VM VHD file, and the certificates for running the Daemon and have it listen to port 4243.

* Make sure the host VM is available by visiting portal.azure.com and klik the browse button to go to the list of running VM. Select the VM with the hostname we used in the 'azure vm docker create command (step B5). Also note both endpoints created from the command and the CLI tools and the Docker extension.
* To make our container available outside of the host we need to add another endpoint. Enter 'azure vm endpoint create -n "HTTP" "vmhostname" 80 80' to add the endpoint for HTTP traffic through TCP port 80. You can check the portal website to confirm the creating of the endpoint. the -n is just an endpoint name. The options '80 80' refer to the mapping of an internal port (the one openen in the host VM) and external port (the one we can talk to from our Tessel client).
* Check whether our Docker service is running by running 'sudo docker --tls -H tcp://vmhostname.cloudapp.net:4243 info'. --tls lets us run a command on the host VM from the client console/terminal and this works because we already have the necessary certificates setup.

We could use the tls command also to setup an image for our container on but a better approach would be to define a Dockerfile and let Docker manage the creation of the image. The Dockerfile instructs Docker what base image should be used and what command it must execute on top of the base image to create additional layers that ultimately make up the image that has all the parts our app needs to run. In our case this will be Node, NPM (the Node package manager) and our application script files.

#### Running the REST API and connecting up the Tessel client
* Create the Dockerfile
* Let Docker build the image
* Run the image
* Setup the Tessel to call the API running in the container.

	// Use comments in code only if code is otherwise confusing.
	// We want the code to be as good and clean written that it
	// is self-explanatory and doesn't need comments. Still don't
	// be afraid to use comments if needed.

	code.indent(tab); // Indent code with 4 spaces (or tab) to have it appear as code

#### Calling the service from a Tessel.

Summary
-------
During this lab we have seen that with Docker we can containerize our deployments and thus make more efficient use of our cloud resources in Azure. Containerization enables us to run our application components (API's, Sites, Databases etc.) in lightweight isolated runtimes that are very easy to deploy, interconnect and move to wherever whenever we wish. Containerization can be considered the next step in Platform as a Service and with Azure this scenario is fully supported with Linux and in the near future on Windows VM's as well.
