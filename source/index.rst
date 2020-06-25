Create a cluster in the cloud
=============================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   infrastructure
   oracle-infrastructure
   google-infrastructure
   finalise
   running

Welcome to the documentation for cluster in the cloud.
By the end of this you  will have a fully-operational, elastically-scaling, heterogeneous Slurm cluster running on cloud resources.

In the future, the intention is that this tutorial will cover installing on all major cloud providers
but for now Oracle Cloud Infrastructure and Google Cloud Platform are covered.

This tutorial and the Cluster in the Cloud software was created by `Matt Williams <https://github.com/milliams/>`_
at the `ACRC in Bristol <http://www.bristol.ac.uk/acrc/>`_.
Contributions to this tutorial document are welcome `at GitHub <https://github.com/ACRC/cluster-in-the-cloud>`_.

.. admonition:: If you need help

   If you have any questions or issues with the system as a whole,
   please raise them as a GitHub issue at `milliams/cluster-in-the-cloud <https://github.com/milliams/cluster-in-the-cloud/issues>`_.

Prerequisites
-------------

To complete this tutorial you will need:

* access to a command-line (i.e. Linux, MacOS Terminal or WSL)
* an `SSH key pair <https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/>`_
* an account with credit on Oracle or Google cloud

  * the account must have admin permissions to create infrastructure

* local software installed

  * Terraform 0.12
  * SSH
  * Git

Alternatively, for some cloud providers (only Google so far) we have a simpler "one-click" installer which has no up-front requirements at all.
That is covered in the Google chapter.

Start by :doc:`creating the infrastructure <infrastructure>`.
