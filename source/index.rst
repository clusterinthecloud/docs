Create a cluster in the cloud
=============================

Welcome to the documentation for Cluster in the Cloud.
By the end of this you  will have a fully-operational, elastically-scaling, heterogeneous Slurm cluster running on cloud resources.

In the future, the intention is that this tutorial will cover installing on all major cloud providers
but for now AWS, Google Cloud Platform and Oracle Cloud Infrastructure are covered.

This tutorial and the Cluster in the Cloud software was created by `Matt Williams <https://github.com/milliams/>`_
at the `ACRC in Bristol <http://www.bristol.ac.uk/acrc/>`_.
Contributions to this tutorial document are welcome `at GitHub <https://github.com/clusterinthecloud/docs>`_.

.. admonition:: If you need help
   :class: tip

   If you have any questions or issues with the system as a whole,
   please raise them as a GitHub issue at `clusterinthecloud/support <https://github.com/clusterinthecloud/support/issues>`_.

.. note::

   If you use Cluster in the Cloud for any published research, please mention us in your acknowledgements.

.. toctree::
   :maxdepth: 1
   :caption: Tutorial

   infrastructure
   aws-infrastructure
   google-infrastructure
   oracle-infrastructure
   finalise
   running

Prerequisites
-------------

To complete this tutorial you will need:

* access to a command-line (i.e. Linux, MacOS Terminal or WSL)
* an `SSH key pair <https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/>`_
* an account with credit on AWS, Google or Oracle cloud

  * the account must have admin permissions to create infrastructure

* local software installed

  * Terraform 1.0 or newer
  * SSH
  * Git

Alternatively, for some cloud providers (only AWS and Google so far) we have a simpler "one-click" installer which has fewer or no up-front requirements at all.
That is covered in the AWS and Google chapters.

Start by :doc:`creating the infrastructure <infrastructure>`.
