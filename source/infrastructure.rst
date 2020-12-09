Set up the cloud infrastructure
===============================

If you are using AWS or Google and want to use the 1-click installer, you do not need to install Terraform.
You can skip this chapter and go straight to their page.

Getting Terraform
-----------------

The first step is to get a bunch of servers powered on in your cloud.
We do this using a tool called `Terraform <https://www.terraform.io/>`_.

Make sure that Terraform is installed by running the following in the command-line:

.. code-block:: shell-session

   $ terraform version

you should get output like:

.. code-block:: shell-session

   Terraform v0.12.9

Getting the Terraform config
----------------------------

Terraform is a tool for creating infrastructure but we need to provide it with some configuration.

Start by making a new directory which will hold all our configuration.
Change to that directory in your terminal.

Grab the Terraform config from Git using:

.. code-block:: shell-session

   $ git clone https://github.com/clusterinthecloud/terraform.git
   $ cd terraform

We're now ready to start configuring our infrastructure on either:

- :doc:`AWS <aws-infrastructure>` or
- :doc:`Google Cloud Platform <google-infrastructure>` or
- :doc:`Oracle Cloud Infrastructure <oracle-infrastructure>`.
