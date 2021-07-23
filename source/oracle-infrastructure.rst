Creating the infrastructure on Oracle
=====================================

Setting the config
------------------

.. code-block:: shell-session

   $ terraform -chdir=oracle init

Now, when you check the Terraform version, you should see the OCI provider showing up:

.. code-block:: shell-session

   $ terraform -chdir=oracle version
   Terraform v1.0.3
   on linux_amd64
   + provider registry.terraform.io/hashicorp/oci v4.36.0
   + provider registry.terraform.io/hashicorp/random v3.1.0
   + provider registry.terraform.io/hashicorp/template v2.2.0
   + provider registry.terraform.io/hashicorp/tls v3.1.0

Rename the example config file ``oracle/terraform.tfvars.example`` to ``oracle/terraform.tfvars`` and open it in a text editor:

.. code-block:: shell-session

   $ mv oracle/terraform.tfvars.example oracle/terraform.tfvars
   $ vim oracle/terraform.tfvars

Following the instructions at the `Oracle Terraform plugin docs <https://www.terraform.io/docs/providers/oci/index.html#authentication>`_,
set the values of ``tenancy_ocid``, ``user_ocid``, ``private_key_path``, ``fingerprint`` and ``region``.
Make sure that the user account you use for ``user_ocid`` has admin access in your tenancy to create infrastructure.

You will also need to set the compartment OCID of the compartment that you are using.
If you are using the default root compartment, this will be the same as your tenancy OCID.

The next thing to set is an SSH key that you will use to connect to the server once it is built.
See `GitHub's documentation <https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/>`_ on information on how to do this
and then paste the contents of the public key into the ``ssh_public_key`` config variable between the two ``EOF``\ s.

You will want a simple, lightweight VM for the management node so
for this tutorial, we will use ``VM.Standard2.16`` for the management node.

Set the ``ManagementShape`` config variable to the shape you want for the management node::

   ManagementShape = "VM.Standard2.1"

The second thing we need to do for the management node is decide which AD it should reside in.
Set the variable ``ManagementAD`` to whichever AD you'd like to use::

   ManagementAD = "1"

Running Terraform
-----------------

At this point, we are ready to provision our infrastructure.
Check that there's no immediate errors with

.. code-block:: shell-session

   $ terraform -chdir=oracle validate

It should return with no errors.
If there are any problems, fix them before continuing.

Next, check that Terraform is ready to run with

.. code-block:: shell-session

   $ terraform -chdir=oracle plan

which should have, near the end, something like ``Plan: 11 to add, 0 to change, 0 to destroy.``.

We're now ready to go. Run

.. code-block:: shell-session

   $ terraform -chdir=oracle apply

and, when prompted, tell it that "yes", you do want to apply.

It will take some time but should return without any errors with something green that looks like::

   Apply complete! Resources: 9 added, 0 changed, 0 destroyed.

   Outputs:

   ManagementPublicIP = "130.61.43.69"
   cluster_id = "cheerful-macaw"

You are now ready to move on to :doc:`finalising the setup on the cluster <finalise>`.
