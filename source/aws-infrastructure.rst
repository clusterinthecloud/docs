Creating the infrastructure on AWS
==================================

The first thing to do is to set up your local credentials so that you can communicate with the AWS APIs.

`Install the AWS command-line tools <https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html>`_ by following their instructions.

Once installed, `configure the tool <https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html>`_ by running:

.. code-block:: shell-session

   $ aws configure

All later steps will use these credentials.

The final step of setup needed is to create a key which Terraform will use to communicate with the server to upload some configuration.
For now this must be created with no passphrase:

.. code-block:: shell-session

   $ ssh-keygen -t rsa -f ~/.ssh/aws-key -N ""

Setting the config
------------------

To initialise the local Terraform repo, start by running the following:

.. code-block:: shell-session

   $ terraform init aws

Now, when you check the Terraform version, you should see the AWS provider showing up:

.. code-block:: shell-session

   $ terraform version
   Terraform v0.12.21
   + provider.aws v2.16.0
   + provider.external v1.2.0
   + provider.local v1.3.0
   + provider.null v2.1.2
   + provider.random v2.2.1
   + provider.template v2.1.0

Rename the example config file ``aws/terraform.tfvars.example`` to ``terraform.tfvars`` and open it in a text editor:

.. code-block:: shell-session

   $ mv aws/terraform.tfvars.example terraform.tfvars
   $ vim terraform.tfvars

There's a few variables which can be set in here.
The only variable which you must set is ``admin_public_keys`` which must contain the public key you wish to use to log in to the admin account on the cluster.
You can set multiple public keys here if you wish.

To see what other possible configuration opions there are, look at ``aws/variables.tf``.

Running Terraform
-----------------

At this point, we are ready to provision our infrastructure.
Check that there's no immediate errors with

.. code-block:: shell-session

   $ terraform validate aws

It should return with no errors.
If there are any problems, fix them before continuing.

Next, check that Terraform is ready to run with

.. code-block:: shell-session

   $ terraform plan aws

which should have, near the end, something like ``Plan: 22 to add, 0 to change, 0 to destroy.``.

We're now ready to go. Run

.. code-block:: shell-session

   $ terraform apply aws

and, when prompted, tell it that "yes", you do want to apply.

It will take some time but should return without any errors with something green that looks like::

   Apply complete! Resources: 22 added, 0 changed, 0 destroyed.

   Outputs:

   ManagementPublicIP = 130.61.43.69

You are now ready to move on to :doc:`finalising the setup on the cluster <finalise>`.
