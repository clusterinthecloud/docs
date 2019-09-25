Creating the infrastructure on Google
=====================================

Setting up the environment
--------------------------

- Install gcloud
- Run ``gcloud iam service-accounts list`` to find email of "Compute Engine default service account"
- ``gcloud config set account googleaccount@googlecloud.example``
- ``gcloud config set project citc-123456``

- ``gcloud iam service-accounts create citc-terraform --display-name "CitC Terraform"``
- ``gcloud projects add-iam-policy-binding citc-123456 --member serviceAccount:citc-terraform@citc-123456.iam.gserviceaccount.com --role='roles/editor'``
- ``gcloud projects add-iam-policy-binding citc-123456 --member serviceAccount:citc-terraform@citc-123456.iam.gserviceaccount.com --role='roles/iam.securityAdmin'``
- ``gcloud iam service-accounts keys create citc-terraform-credentials.json --iam-account=citc-terraform@citc-123456.iam.gserviceaccount.com``

- ``gcloud services enable compute.googleapis.com iam.googleapis.com cloudresourcemanager.googleapis.com file.googleapis.com``

- Create ssh key pair ``ssh-keygen -t rsa -f ~/.ssh/citc-google -C provisioner``

Setting the config
------------------

.. code-block:: shell-session

   $ terraform init google-cloud-platform

Now, when you check the Terraform version, you should see the OCI provider showing up:

.. code-block:: shell-session

   $ terraform version
   Terraform v0.12.9
   + provider.oci v3.44.0
   + provider.template v2.1.0
   + provider.tls v2.1.0

Rename the example config file ``google-cloud-platform/terraform.tfvars.example`` to ``terraform.tfvars`` and open it in a text editor:

.. code-block:: shell-session

   $ mv google-cloud-platform/terraform.tfvars.example terraform.tfvars
   $ vim terraform.tfvars

.. note::
   setup

Running Terraform
-----------------

At this point, we are ready to provision our infrastructure.
Check that there's no immediate errors with

.. code-block:: shell-session

   $ terraform validate google-cloud-platform

It should return with no errors.
If there are any problems, fix them before continuing.

Next, check that Terraform is ready to run with

.. code-block:: shell-session

   $ terraform plan google-cloud-platform

which should have, near the end, something like ``Plan: 11 to add, 0 to change, 0 to destroy.``.

We're now ready to go. Run

.. code-block:: shell-session

   $ terraform apply google-cloud-platform

and, when prompted, tell it that "yes", you do want to apply.

It will take some time but should return without any errors with something green that looks like::

   Apply complete! Resources: 11 added, 0 changed, 0 destroyed.

   Outputs:

   ManagementPublicIP = 130.61.43.69

You are now ready to move on to :doc:`finalising the setup on the cluster <finalise>`.
