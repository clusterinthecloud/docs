Creating the infrastructure on AWS
==================================

The first thing to do is to set up your local credentials so that you can communicate with the AWS APIs.

`Install the AWS command-line tools <https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html>`_ by following their instructions.

Once installed, `configure the tool <https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html>`_ by running:

.. code-block:: shell-session

   $ aws configure

All later steps will use these credentials.

There are two ways of creating Cluster in the Cloud on AWS.
The older way was to download the code to your computer and run it from there.
This requires installing Terraform, Git and setting up SSH locally.

There is now also a "1-click" installer available which is covered in the first section on this page.
The older method is also documented here for posterity.

1-click installer
-----------------

Download the installer (``install-citc.py``) and uninstaller (``destroy-citc.py``) scripts from `clusterinthecloud/installer <https://github.com/clusterinthecloud/installer>`__.
You can do this either by cloning the  Git repo (``git clone https://github.com/clusterinthecloud/installer.git``), by downloading the zip (`master.zip <https://github.com/clusterinthecloud/installer/archive/master.zip>`__) or by downloading the two files individually.

The only requirements for the script are Python >= 2.7, ``ssh``, ``ssh-keygen`` and ``scp``.

Run the install script as:

.. code-block:: shell-session

   $ ./install-citc.sh aws

If you need to specify the region for the cluster or an AWS credential profile, you can pass them in with ``--region`` or ``--profile``.

This should download all it needs automatically, start the cluster and provide you with a private key (in a subdirectory named after the cluster ID, and called ``citc-key``) and an IP address. Something like:


.. code-block:: text

   The file 'citc-terraform-useful-gnu/citc-key' will allow you to log into the new cluster
   Make sure you save this key as it is needed to destroy the cluster later.
   The IP address of the cluster is 130.61.43.69
   Connect with:
     ssh -i citc-terraform-useful-gnu/citc-key citc@130.61.43.69

You can now move on to the next page of the tutorial, :doc:`finalising the setup on the cluster <finalise>`.

Otherwise, if you wish to do the steps manually, then read on from here...

Setting the config
------------------

To initialise the local Terraform repo, start by running the following:

.. code-block:: shell-session

   $ terraform -chdir=aws init

Now, when you check the Terraform version, you should see the AWS provider showing up:

.. code-block:: shell-session

   $ terraform -chdir=aws version
   Terraform v1.0.3
   on linux_amd64
   + provider registry.terraform.io/hashicorp/aws v3.51.0
   + provider registry.terraform.io/hashicorp/external v2.1.0
   + provider registry.terraform.io/hashicorp/local v2.1.0
   + provider registry.terraform.io/hashicorp/null v3.1.0
   + provider registry.terraform.io/hashicorp/random v3.1.0
   + provider registry.terraform.io/hashicorp/template v2.2.0
   + provider registry.terraform.io/hashicorp/tls v3.1.0

Rename the example config file ``aws/terraform.tfvars.example`` to ``aws/terraform.tfvars`` and open it in a text editor:

.. code-block:: shell-session

   $ mv aws/terraform.tfvars.example aws/terraform.tfvars
   $ vim aws/terraform.tfvars

There's a few variables which can be set in here.
The only variable which you must set is ``admin_public_keys`` which must contain the public key you wish to use to log in to the admin account on the cluster.
You can set multiple public keys here if you wish.

To see what other possible configuration opions there are, look at ``aws/variables.tf``.

Running Terraform
-----------------

At this point, we are ready to provision our infrastructure.
Check that there's no immediate errors with

.. code-block:: shell-session

   $ terraform -chdir=aws validate

It should return with no errors.
If there are any problems, fix them before continuing.

Next, check that Terraform is ready to run with

.. code-block:: shell-session

   $ terraform -chdir=aws plan

which should have, near the end, something like ``Plan: 25 to add, 0 to change, 0 to destroy.``.

We're now ready to go. Run

.. code-block:: shell-session

   $ terraform -chdir=aws apply

and, when prompted, tell it that "yes", you do want to apply.

It will take some time but should return without any errors with something green that looks like::

   Apply complete! Resources: 25 added, 0 changed, 0 destroyed.

   Outputs:

   ManagementPublicIP = "130.61.43.69"
   cluster_id = "cheerful-macaw"

You are now ready to move on to :doc:`finalising the setup on the cluster <finalise>`.
