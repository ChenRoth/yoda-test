---
layout: bt_wiki
title: Quickstart
category: Manager Intro
draft: false
weight: 200

quickstart_link: getting-started.html
blueprint_file_link: https://raw.githubusercontent.com/cloudify-cosmo/cloudify-nodecellar-example/3.3/singlehost-blueprint.yaml
virtualbox_link: https://www.virtualbox.org/
vagrant_link: http://www.vagrantup.com
vagrant_file_link: http://gigaspaces-repository-eu.s3.amazonaws.com/org/cloudify3/3.3.0/ga-RELEASE/Vagrantfile
terminology_link: reference-terminology.html
workflows_link: workflows-built-in.html
blueprint_guide_link: understanding-blueprints.html
installation_general_link: installation.html
nodecellar_version: 3.3
---

hugo

hugo
You can take Cloudify's Manager for an instant test drive with an [online trial.](http://getcloudify.org/widget.html)
hugo


In this tutorial you will start a Cloudify Manager within a Vagrant box on your laptop, and install a sample blueprint on it.

The [blueprint](hugo) that you'll be deploying describes a nodejs application that connects to a MongoDB database and presents a wine catalog.


hugo
The [Terminology Reference Page](hugo) will help you understand some of the terms stated in this guide. For your convenience, links are supplied throughout the guide to point you to the right term definition in the terminology page.
hugo

# IMPORTANT: Before You Begin

You'll need to have the following setup in your environment:

* [Oracle VirtualBox](hugo) (this box has been tested with version 4.3 or higher, but earlier versions should work as well).
* [Vagrant](hugo) (Make sure that you are using version 1.5 or above!).
* At least 2GB of free RAM

hugo
Your Hypervisor must support nested virtualization in order to run Virtualbox inside a VM. Unless you know you can run a VM inside a VM, please run the box from either your laptop or on a bare metal server.
hugo

hugo
* Do not run the command prompt as Administrator (privilege escalation).
* Hyper-V & Virtualbox [do not play nice together](https://docs.vagrantup.com/v2/hyperv/index.html). Disabling Hyper-V is
possible by running the `bcdedit /set hypervisorlaunchtype off` command (reboot is needed).
hugo

# Step by Step Walkthrough

## Step 1: Download Vagrantfile and run VM

The first thing you'll need to do is download the Vagrantfile that Vagrant will use to create a virtual machine with the Cloudify manager and CLI pre installed.

Download this [Vagrantfile](hugo) to your local directory. Then, run this command:

hugo
vagrant up
hugo

Once the Cloudify Vagrant box is up, you can access the manager web console through your local browser by pointing the browser to [http://10.10.1.10/](http://10.10.1.10/).

## Step 2: SSH to the Vagrant Box and Connect to the Running Manager

To connect to the newly Up'd box, type:

hugo
vagrant ssh
hugo

...after which Cloudify's CLI will be at your disposal.

## Step 3: Download the blueprint

Cloudify uses [blueprints](hugo#blueprint) to describe the overall application orchestration, including the application nodes, workflows, and relationships.

You'll have to clone a sample blueprint from our Github repository from the Vagrant box.

hugo
cd blueprints
git clone https://github.com/cloudify-cosmo/cloudify-nodecellar-example
cd cloudify-nodecellar-example/
git checkout tags/hugo
hugo

## Step 4: Upload the Blueprint and Create a Deployment

Now, we upload a sample [blueprint](hugo#deployment) based on it.

In the `cloudify-nodecellar-example` directory that you just cloned, you can see a blueprint file (named `singlehost-blueprint.yaml`) alongside other resources related to this blueprint.

To upload the blueprint run:

hugo
cfy blueprints upload -b nodecellar -p singlehost-blueprint.yaml
hugo

hugo
The DNS address used by cloudify in the getting-started box is set to 8.8.8.8.
hugo

The `-b` flag specifies the unique name we've assigned to this blueprint on the Cloudify manager.
Before creating a deployment, let's see what this blueprint looks like.

Point your browser at the manager's URL again and refresh the screen. You will see the nodecellar blueprint listed there.

![Blueprints table](hugo)

Click the blueprint. You can see its topology. A [topology](hugo#node).

In our case, we have the following nodes:

* One VM
* A nodejs server
* A MongoDB database
* A nodejs application called nodecellar (which is a nice sample nodejs application backed by mongodb).

![Nodecellar Blueprint](hugo)

This blueprint defines some input parameters:

![Nodecellar Inputs](hugo)

The inputs values are located at ~/cloudify/blueprints/inputs/nodecellar-singlehost.yaml.

These are the values relevant for our example:

hugo
agent_private_key_path: /home/vagrant/.ssh/id_rsa
agent_user: vagrant
host_ip: 10.10.1.10
hugo

hugo
Because the Vagrant box is a self-contained example, these values cannot be changed, and are presented here only for the sake of clarity.
hugo

Now, we need to create a deployment.

In Cloudify, a [deployment](hugo#deployment) represents a virtual environment on your Cloudify manager with all of the software components needed to execute the application lifecycle described in a blueprint, based on the inputs provided in the `cfy deployments create` command.

To create a deployment, type the following command:

hugo
cfy deployments create -b nodecellar -d nodecellar --inputs ../inputs/nodecellar-singlehost.yaml
hugo

We've now created a deployment named `nodecellar` based on a blueprint with the same name.

This deployment is not yet materialized, since we haven't issued an installation command. If you click the "Deployments" icon in the left sidebar in the web UI, you will see that all nodes are labeled with 0/1, which means they're pending creation.

![Nodecellar Deployment](hugo)

## Step 5: Install the Deployment

In Cloudify, installing a certain `deployment` is done by executing the [install](hugo#workflow).

Type the following command in your terminal:

hugo
cfy executions start -w install -d nodecellar
hugo

This will take a couple of minutes, during which the resources will be created and configured.

You can track the installation progress in the web console or in your terminal application. In your terminal, you will see that each [event](hugo#event) is labeled with its time, the deployment name, and the node in our topology that it relates to, e.g.

hugo
2014-12-02T09:46:05 CFY <nodecellar> [nodejs_d36c8] Creating node
hugo

In the Web UI, you can checkout the Logs/Events page for an overview of all logs and events in your manager.

![Events](hugo)

<br>

Alternatively, click on a specific deployment in the deployment tab. A list containing events and logs for the deployment will be shown.

You can also have a look at the Monitoring tab and see some default metrics:

![Metrics](hugo)

hugo
The blueprint we installed actually defines a custom collector for the Mongo database. To add mongo related graphs to the dashboard, have a look at [Adding Custom Graphs](hugo).
hugo

## Step 6: Test Drive the Application

To test the application, you will need to access it using its public IP address. Go to [http://10.10.1.10:8080](http://10.10.1.10:8080) to access it from your web browser. The marvelous nodecellar application should be up on your screen. Click the "Browse wines" button to verify that the application was installed successfully and can access the mongodb database to read the list of wines.

![Nodecellar](hugo)

## Step 7: Uninstall the Deployment

Uninstalling the deployment is just a matter of running another workflow. In our nodecellar example, this will teardown all the resources provisioned by the `install` workflow.

To run the [uninstall](hugo#uninstall) workflow, type the following command:

hugo
cfy executions start -w uninstall -d nodecellar
hugo

Like with the `install` workflow, you can track the progress of the uninstall process in the CLI or the web UI using the events that are displayed in both.

Once the workflow is completed, you can verify that the resources were indeed destroyed.

In a real cloud deployment, each and every resource provisioned by the deployment will be destroyed. Since this is a single host example, there aren't any external resources, only application related ones.

## Step 8: Delete the Deployment

The next step is deleting the deployment. Assuming the uninstallation went fine, all of the application resources will have been removed.

The deployment itself still has record on the manager. All of its static and runtime properties are still stored in the manager's database. To clean up the deployment's information on the manager, delete the deployment by running this command:

hugo
cfy deployments delete -d nodecellar
hugo

## Step 9: Tear down the Manager

If you have no use for it, you can tear down the manager. This can be done by issuing the following command:

hugo
cfy teardown -f
hugo

In a real cloud deployment, this will terminate the Manager VM and delete the resources associated with it.

In our case, since the manager is installed on the same machine the CLI is installed on, it will not teardown the machine.

Once you're done, you can exit the ssh session.

If you want to destroy the machine, run:

hugo
vagrant destroy -f
hugo

If you want to start the same machine again, just "Up" it. If you want to completely remove the box from your machine, run:

hugo
vagrant box remove cloudify-box
hugo

# What's Next

* Understand the requirements for bootstrapping your very own Cloudify Manager by reading [this](hugo).

* Try to install the same application on a real cloud provider by following the [Cloudify Getting Started Guide](hugo).

* Learn more about blueprints by following the [Blueprints Tutorial](hugo).
