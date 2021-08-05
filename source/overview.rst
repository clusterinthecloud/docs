Overview
========

.. admonition:: What is this page about?

    This document is intended for people who are interested in contributing to Cluster in the Cloud
    or who are facing a bug or issue and want to understand how things work beneath the surface.

Cluster in the Cloud can be though of as working in three main steps:

.. contents::
    :local:
    :depth: 1

The first two are relatively static and only happens on cluster creation (and are destroyed on cluster destruction)
while the third will be constantly happening while submitting jobs.

Creating the infrastructure
---------------------------

The first step is to create the base infrastructure for the cluster.
Exactly what gets created here varies from one cloud provider to another but in general it is:

- Virtual network
- Management/login node
- Shared storage

The resources are created using `Terraform <https://www.terraform.io/>`_ from the configuration stored in `clusterinthecloud/terraform <https://github.com/clusterinthecloud/terraform>`_.

Terraform only manages these resources, and not the compute nodes themselves.
This means that when shutting down the cluster, the compute nodes must be destroyed by some other channel.

Configuring the software
------------------------

Once the management node has been created it will kick off a configuration step using `Ansible <https://github.com/ansible/ansible>`_.

The Ansible configuration for the management node is all contained at `clusterinthecloud/ansible <https://github.com/clusterinthecloud/ansible>`_
and it's in this repo that the majority of the logic of the system is codified.

The Ansible playbook for the management node does a lot of steps, but importantly includes:

- Setting up the Slurm controller daemon
- Setting up the LDAP server for managing users
- Configuring the monitoring system
- Kicking off the creation of the compute node image

This last step uses `Packer <https://www.packer.io>`_ to start a new VM instance and run a related Ansible playbook on it
and then save the resulting machine as the template node image for all compute nodes.
This image creation step :doc:`can be run manually at any time <node-images>` by the admin to update the image.

Dynamically managing nodes
--------------------------

The final step is how the system provisions and destroys compute nodes.

The core logic is based on Slurm's `power management <https://slurm.schedmd.com/power_save.html>`_ and `elastic computing <https://slurm.schedmd.com/elastic_computing.html>`_ support.
When a job is submitted, Slurm assigns a node from the list of potential nodes.
It then runs a Python script, provided by CitC which talks to the cloud provider's API to create the instance with the correct settings (including choosing the node image).
Once Slurm sees that a node has been idle for a while, it will run another CitC Python script which again talks to the API to terminate the instance.
