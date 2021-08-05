Configuring node images
=======================

.. admonition:: What is this page about?

    This page explains how you can configure the image that is used on the compute nodes.

The images that are used by the compute nodes are built using `Packer <https://packer.io>`_.
The initial image is build automatically when the cluster is first created with the bare essentials needed to run jobs.

If you want to change the image in any way, you can edit the script ``/home/citc/compute_image_extra.sh``.
This script is run automatically at the end of the Packer inside the new image so you can fill it with things like:

.. code-block:: bash

   #! /bin/bash
   sudo yum -y install opencl-headers clinfo

Note the use of ``sudo`` as this script does not run as ``root``.

Once the script has been edited to your liking, re-run Packer with:

.. code-block:: shell-session

   [citc@mgmt ~]$ sudo /usr/local/bin/run-packer

This will start a VM inside your cloud account, build the image and then shut down the VM.
From that point on, any newly-started nodes will use the new image.

GPU nodes
---------

The default image that is built by CitC does not include any GPU drivers.
If you want to use the GPUs on a given node, you will need to compile the drivers into the node image.

You can follow the `NVIDIA documentation for this <https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#redhat8-installation>`__, which at its simplest means adding:

.. code-block:: bash

   if [[ $(arch) == "x86_64" ]]; then
     sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo
     sudo dnf module install -y nvidia-driver:latest-dkms
   fi

to ``/home/citc/compute_image_extra.sh`` and re-running Packer as shown above.

This will just install the base drivers, but if you need CUDA too, then add ``sudo dnf install -y cuda`` to the commands in that file too.

AWS ARM nodes
-------------

AWS provides instance types with Graviton processors which are based on the aarch64 architecture.
This requires a special image to be built which is not created by default.
To build this image, run the following:

.. code-block:: shell-session

   [citc@mgmt ~]$ sudo /usr/local/bin/run-packer aarch64

which will build the aarch64 image.
