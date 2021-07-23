Creating the infrastructure on Google
=====================================

There are two ways of creating Cluster in the Cloud on Google Cloud.
The older way was to download the code to your computer and run it from there.
This requires installing Terraform, Git and setting up SSH locally.

There is now also a "1-click" installer available which is covered in the first section on this page.
The older method is also documented here for posterity.

1-click installer
-----------------

Go to the Google Cloud Console and `open a Cloud Shell <https://console.cloud.google.com/home/dashboard?cloudshell=true>`_.
In the cloud shell type:

.. code-block:: shell-session

   $ docker run -it -e CLOUDSDK_CONFIG=/config/gcloud \
                    -v $CLOUDSDK_CONFIG:/config/gcloud \
                    clusterinthecloud/google-install

It will download and start running the setup program and will ask some questions.
If a question has a default it will show you it in ``[square brackets]``, to accept a default, just press enter.

When it asks for your SSH keys, the simplest approch is to point it to a URL containing your public keys, such as on GitHub as a URL like ``https://github.com/milliams.keys``.

It may ask if you want to continue with the authorisation method used, press ``Y`` and enter..

It will then authenticate with Google and give you a long URL to click on. Click it, sign-in to Google and allow access.
Take the authentication token it gives you and paste it into the Cloud Shell (ctrl-v and press enter).

It will then go through the process of setting up Cluster in the Cloud and will give you the IP address of the login node to SSH into:

.. code-block:: text

   Your Cluster-in-the-Cloud has now been created :-)
   Proceed to the next stage. Connect to the cluster
   by running 'ssh citc@130.61.43.69'

   {"status":"0", "cluster_ip":"130.61.43.69"}

You can now move on to the next page of the tutorial, :doc:`finalising the setup on the cluster <finalise>`.

Otherwise, if you wish to do the steps manually, then read on from here...

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

Setting the config
------------------

To initialise the local Terraform repo, start by running the following:

.. code-block:: shell-session

   $ terraform -chdir=google init

Now, when you check the Terraform version, you should see the Google provider showing up:

.. code-block:: shell-session

   $ terraform -chdir=google version
   Terraform v1.0.3
   on linux_amd64
   + provider registry.terraform.io/hashicorp/external v2.1.0
   + provider registry.terraform.io/hashicorp/google v3.76.0
   + provider registry.terraform.io/hashicorp/local v2.1.0
   + provider registry.terraform.io/hashicorp/random v3.1.0
   + provider registry.terraform.io/hashicorp/template v2.2.0
   + provider registry.terraform.io/hashicorp/tls v3.1.0

Rename the example config file ``google/terraform.tfvars.example`` to ``google/terraform.tfvars`` and open it in a text editor:

.. code-block:: shell-session

   $ mv google/terraform.tfvars.example google/terraform.tfvars
   $ vim google/terraform.tfvars

There's a few variables which we need to change in here.
First you must set the ``region`` and ``zone`` variables to the correct values for your account.
This will depend on what regions you have access to and where you want to build your cluster.

Then the ``project`` variable must be set to the project ID as we used above when running ``gcloud``.

You must set ``admin_public_keys`` to contain the public key you wish to use to log in to the admin account on the cluster.
You can set multiple public keys here if you wish.

Finally, if you wish you can change the node type used for the management node.
By default it's a lightweight single-core VM which should be sufficient for most uses but you can change it if you wish.

The rest of the variables should usually be left as they are.

Running Terraform
-----------------

At this point, we are ready to provision our infrastructure.
Check that there's no immediate errors with

.. code-block:: shell-session

   $ terraform -chdir=google validate

It should return with no errors.
If there are any problems, fix them before continuing.

Next, check that Terraform is ready to run with

.. code-block:: shell-session

   $ terraform -chdir=google plan

which should have, near the end, something like ``Plan: 11 to add, 0 to change, 0 to destroy.``.

We're now ready to go. Run

.. code-block:: shell-session

   $ terraform -chdir=google apply

and, when prompted, tell it that "yes", you do want to apply.

It will take some time but should return without any errors with something green that looks like::

   Apply complete! Resources: 11 added, 0 changed, 0 destroyed.

   Outputs:

   ManagementPublicIP = "130.61.43.69"
   cluster_id = "cheerful-macaw"

You are now ready to move on to :doc:`finalising the setup on the cluster <finalise>`.
