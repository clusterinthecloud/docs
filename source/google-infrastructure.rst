Creating the infrastructure on Google
=====================================

Setting up the environment
--------------------------

Before we can install the cluster onto your cloud environment, we need to do some initial one-off setup.
Firstly, we need to install the command-line tool ``gcloud`` which allows you to configure Google Cloud.
Download and setup ``gcloud`` based on the `instructions from Google <https://cloud.google.com/sdk/docs/>`_.

Once you have ``gcloud`` installed, start by associating it with your Google account:

.. code-block:: shell-session

   $ gcloud config set account <googleaccount@example.com>

where you should replace ``<googleaccount@example.com>`` with your email address.

Now that it knows who you are, you should set a default project.
You can find the ID of your project with ``gcloud projects list``.

.. code-block:: shell-session

   $ gcloud config set project <citc-123456>

Once the project has been set, we can enable the required APIs to build Cluster in the Cloud.
This step will likely take a few minutes so be patient:

.. code-block:: shell-session

   $ gcloud services enable compute.googleapis.com \
                            iam.googleapis.com \
                            cloudresourcemanager.googleapis.com \
                            file.googleapis.com

That's all the structural setup for the account needed.
The last ``gcloud`` thing we need to do is create a service account which Terraform uses to communicate with GCP.
Make sure to replace every instance of ``<citc-123456>`` with your project ID:

.. code-block:: shell-session

   $ gcloud iam service-accounts create citc-terraform --display-name "CitC Terraform"
   $ gcloud projects add-iam-policy-binding <citc-123456> --member serviceAccount:citc-terraform@<citc-123456>.iam.gserviceaccount.com --role='roles/editor'
   $ gcloud projects add-iam-policy-binding <citc-123456> --member serviceAccount:citc-terraform@<citc-123456>.iam.gserviceaccount.com --role='roles/iam.securityAdmin'
   $ gcloud iam service-accounts keys create citc-terraform-credentials.json --iam-account=citc-terraform@<citc-123456>.iam.gserviceaccount.com

This will create a local JSON file which contains the credentials for this user.

The final step of setup needed is to create a key which Terraform will use to communicate with the server to upload some configuration.
For now this must be created with no passphrase:

.. code-block:: shell-session

   $ ssh-keygen -t rsa -f ~/.ssh/citc-google -C provisioner -N ""

Setting the config
------------------

To initialise the local Terraform repo, start by running the following:

.. code-block:: shell-session

   $ terraform init google-cloud-platform

Now, when you check the Terraform version, you should see the Google provider showing up:

.. code-block:: shell-session

   $ terraform version
   Terraform v0.12.9
   + provider.external v1.2.0
   + provider.google v2.10.0
   + provider.tls v1.3.0
   + provider.template v2.1.0

Rename the example config file ``google-cloud-platform/terraform.tfvars.example`` to ``terraform.tfvars`` and open it in a text editor:

.. code-block:: shell-session

   $ mv google-cloud-platform/terraform.tfvars.example terraform.tfvars
   $ vim terraform.tfvars

There's a few variables which we need to change in here.
First you must set the ``region`` and ``zone`` variables to the correct values for your account.
This will depend on what regions you have access to and where you want to build your cluster.

Then the ``project`` variable must be set to the project ID as we used above when running ``gcloud``.

Finally, if you wish you can change the node type used for the management node.
By default it's a lightweight single-core VM which should be sufficient for most uses but you can change it if you wish.

The rest of the variables should usually be left as they are.

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
