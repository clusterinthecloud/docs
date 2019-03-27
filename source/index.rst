Create a cluster in the cloud
=============================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   infrastructure
   finalise
   running

Welcome to the documentation for cluster in the cloud.
By the end of this you  will have a fully-operational, elastically-scaling Slurm cluster running on cloud resources.

In the future, the intention is that this tutorial will cover installing on all major cloud providers but for now only Oracle Public Cloud is covered.

This tutorial was created by Matt Williams at the `ACRC in Bristol <http://www.bristol.ac.uk/acrc/>`_.
Contributions to this document are welcome `at GitHub <https://github.com/ACRC/cluster-in-the-cloud>`_.

Prerequisites
-------------

To complete this tutorial you will need:

* access to a command-line (i.e. Linux, MacOS Terminal or WSL)
* an `SSH key pair <https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/>`_
* an account with credit on Oracle cloud

  * the account must have admin permissions to create infrastructure

* local software installed

  * Terraform 0.11
  * SSH

Start by :doc:`creating the infrastructure <infrastructure>`.
