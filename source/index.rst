Create a cluster in the cloud
=============================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   infrastructure
   ansible

Welcome to the documentation for cluster in the cloud.
By the end of this you  will have a fully-operational, elastically-scaling Slurm cluster running on cloud resources.

In the future, the intention is that this tutorial will cover installing on all major cloud providers but for now only Oracle Public Cloud is covered.

This tutorial was created by Matt Williams at the `ACRC in Bristol <http://www.bristol.ac.uk/acrc/>`_.
Contributions to this document are welcome `at GitHub <https://github.com/ACRC/cluster-in-the-cloud>`_.

Prerequisites
-------------

To complete this tutorial you will need:

* access to a command-line (i.e. Linux, MacOS Terminal or WSL)
* an account with credit on Oracle cloud
* local software installed

  * Terraform 0.11
  * Ansible 2.5 or greater
  * SSH 7.3 or greater

Start by :doc:`creating the infrastructure <infrastructure>`.
