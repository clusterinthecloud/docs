Set up the cloud infrastructure
===============================

Getting ready
-------------

The first step is to get a bunch of servers powered on in your cloud.
We do this using a tool called `Terraform <https://www.terraform.io/>`_.

Make sure that Terraform is installed by running the following in the command-line:

.. code-block:: shell-session

   $ terraform version

you should get output like:

.. code-block:: shell-session

   Terraform v0.11.8

We will be using the `Oracle Terraform plugin <https://github.com/oracle/terraform-provider-oci>`_ so follow the instructions there to install it.

We're now ready to start configuring our infrastructure.

Setting the config
------------------

Start by making a new directory which will hold all our configuration.
We will refer to this directory as the *base config directory*.
Change to that directory in your terminal.

Grab the Terraform config from Git using:

.. code-block:: shell-session

   $ git clone https://github.com/ACRC/oci-cluster-terraform.git

Now move into that directory and initialise the Terraform repo:

.. code-block:: shell-session

   $ terraform init

Now, when you check the Terraform version, you should see the OCI provider showing up:

.. code-block:: shell-session

   $ terraform version
   Terraform v0.11.8
   + provider.oci v2.1.13

Rename the example config file ``terraform.tfvars.example`` to ``terraform.tfvars`` and open it in a text editor:

.. code-block:: shell-session

   $ mv terraform.tfvars.example terraform.tfvars
   $ vim terraform.tfvars

Following the instructions at the `Oracle Terraform plugin docs <https://github.com/oracle/terraform-provider-oci#setup-credentials-for-using-oci>`_,
set the values of ``tenancy_ocid``, ``user_ocid``, ``private_key_path``, ``fingerprint`` and ``region``.

You will also need to set the compartment OCID of the compartment that you are using.
If you are using the default root compartment, this will be the same as your tenancy OCID.

The next thing to set is an SSH key that you will use to connect to the server once it is built.
See `GitHub's documentation <https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/>`_ on information on how to do this
and then paste the contents of the public key into the ``ssh_public_key`` config variable between the two ``EOF``\ s.

Finally, you need to decide what type of machines will make up your cluster.
This is dependent on what shapes you have access to so check your service limits in the OCI web console.
You will want a simple, lightweight VM for the management node and a set of more powerful VMs (or better, bare metal) machines for the compute nodes.
For this tutorial, we will use ``VM.Standard2.16`` for the management node and 4 ``VM.Standard2.24`` for the compute nodes but it will depend on what you have access to.

Set the ``ManagementShape`` config variable to the shape you want for the management node::

   ManagementShape = "VM.Standard2.16"

To set the compute nodes, there are two config variables we need to set.
The variable ``ComputeShapes`` contains a list of all the shapes for each node and ``InstanceADIndex`` contains a list of numbers referring to the availability domain each node should be in::

   InstanceADIndex = ["1", "1", "1", "1"]
   ComputeShapes = ["VM.Standard2.24", "VM.Standard2.24", "VM.Standard2.24", "VM.Standard2.24"]

You see that there are two lists, each with four elements.
The nth element in each list are related to each other.
Once the nodes are created, they will be named ``compute001``, ``compute002`` etc. in the order they are listed here.

If we instead wanted a ``BM.GPU2.2`` in AD 1, three ``BM.Standard1.36`` in AD 2 and one ``BM.DenseIO1.36`` in AD3 we would instead write::

   InstanceADIndex = ["1", "2", "2", "2", "3"]
   ComputeShapes = ["BM.GPU2.2", "BM.Standard1.36", "BM.Standard1.36", "BM.Standard1.36", "BM.DenseIO1.36"]

Finally, we need to tell Terraform about all of the ADs that we are putting this in to make sure that the networking is working correctly.
Set ``ADS`` to a list of all the availability domains that we have put infrastructure in::

   ADS = ["1"]

That has defined the types and location of all the nodes we are installing.
We need to tell OCI what OS to install onto each machine which we do by setting ``ComputeImageOCID`` and ``ManagementImageOCID``.
To decide what values to put in these, look at `OCI's list of images <https://docs.us-phoenix-1.oraclecloud.com/images/>`_.
We will install the latest version of Oracle Linux onto each::

   ComputeImageOCID = {
     VM.Standard2.24 = {
       eu-frankfurt-1 = "ocid1.image.oc1.eu-frankfurt-1.aaaaaaaa7qdjjqlvryzxx4i2zs5si53edgmwr2ldn22whv5wv34fc3sdsova"
     }
   }
   ManagementImageOCID = {
     eu-frankfurt-1 = "ocid1.image.oc1.eu-frankfurt-1.aaaaaaaa7qdjjqlvryzxx4i2zs5si53edgmwr2ldn22whv5wv34fc3sdsova"
   }

At this point, we are ready to provision our infrastructure.
Check that there's no immediate errors with

.. code-block:: shell-session

   $ terraform validate

It should return with no errors.
If there are any problems, fix them before continuing.

Next, check that Terraform is ready to run with

.. code-block:: shell-session

   $ terraform plan

which should have, near the end, something like ``Plan: 13 to add, 0 to change, 0 to destroy.``.

We're now ready to go. Run

.. code-block:: shell-session

   $ terraform apply

and, when prompted, tell it that "yes", you do want to apply.

It will take some time but should return without any errors with something green that looks like::

   Apply complete! Resources: 13 added, 0 changed, 0 destroyed.

   Outputs:

   ComputeHostnames = [
       compute001,
       compute002,
       compute003,
       compute004
   ]
   ComputePublicIPs = [
       130.61.39.169,
       130.61.83.195,
       130.61.37.134,
       130.61.44.148
   ]
   ManagementHostnames = [
       mgmt
   ]
   ManagementPublicIPs = [
       130.61.43.69
   ]

You are now ready to move on to :doc:`installing the software on the cluster <ansible>`.
