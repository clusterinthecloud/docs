Running the cluster
===================

Now that you have a cluster which is up and running,
it's worth knowing what you can do with it.

Slurm jobs
----------

A full Slurm tutorial is outside of the scope of this document but it's configured in a fairly standard way.
By default there's one single partition called ``compute`` which contains all the compute nodes.

A simple first Slurm script, ``test.slm``, could look like:

.. code-block:: bash

   #! /bin/bash

   srun -l hostname

which you could run with:

.. code-block:: shell-session

   [matt@mgmt ~]$ sbatch test.slm

To check that Slurm has started the node you need, you can run ``sinfo``:

.. code-block:: shell-session

   [matt@mgmt ~]$ sinfo
   PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
   compute*     up   infinite      1 alloc# vm-standard2-1-ad1-0001

where the ``state`` being ``alloc#`` means that the node has been allocated to a job and the ``#`` means that it is currently in the process of being turned on.

Eventually, once the node has started, the state will change to ``alloc``:

.. code-block:: shell-session

   [matt@mgmt ~]$ sinfo
   PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
   compute*     up   infinite      1  alloc vm-standard2-1-ad1-0001

and then once the job has finished the state will move to ``idle``:

.. code-block:: shell-session

   [matt@mgmt ~]$ sinfo
   PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
   compute*     up   infinite      1   idle vm-standard2-1-ad1-0001

If you want more control over the size of your job etc., then you can set those flags in the job script:

.. code-block:: bash

   #! /bin/bash

   #SBATCH --job-name=test
   #SBATCH --nodes=1
   #SBATCH --ntasks-per-node=2
   #SBATCH --cpus-per-task=1
   #SBATCH --time=10:00

   srun -l hostname

Slurm elastic scaling
---------------------

Slurm is configured to use its `elastic computing <https://slurm.schedmd.com/elastic_computing.html>`_ mode.
This allows Slurm to automatically terminate any nodes which are not currently being used for running jobs
and create any nodes which are needed for running jobs.
This is particularly useful in the cloud as a node which has been terminated will not be charged for.

Slurm does this by calling a script ``/usr/local/bin/startnode`` as the ``slurm`` user.
If necessary, you can call this yourself from the ``citc`` user like:

.. code-block:: shell-session

   [citc@mgmt ~]$ sudo scontrol update NodeName=vm-standard2-1-ad1-0001 State=POWER_UP

to turn on the node ``vm-standard2-1-ad1-0001``.

You should never have to do anything to explicitly shut down the cluster,
it will automatically terminate all nodes which are not in use after a timeout.
The management node will always stay running which is why it's worth only using a relatively cheap VM for it.

The rate at which Slurm shuts down is managed in ``/mnt/shared/etc/slurm/slurm.conf`` by the ``SuspendTime`` parameter.
See the `slurm.conf <https://slurm.schedmd.com/slurm.conf.html>`_ documentation for more details.

You can see the state of all the Slurm nodes (on or off) by running

.. code-block:: shell-session

   [citc@mgmt ~]$ list_nodes

Configuring node images
-----------------------

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

AWS GPU nodes
+++++++++++++

We need to adapt the default packer image as out of the box it does not
contain any of the nvidia software necessary to interact with the GPU.

The first step is to change the ``compute_image_extra.sh`` script to
install the nvidia driver and CUDA toolchain:

.. code-block:: shell-session

   [citc@mgmt ~]$ cat >> compute_image_extra.sh <<EOF
   sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo
   sudo dnf clean all
   sudo dnf -y install kernel-devel
   sudo dnf -y module install nvidia-driver:latest-dkms
   sudo dkms autoinstall
   EOF

We can now re-build the image used to provision compute nodes:

.. code-block:: shell-session

   [citc@mgmt ~]$ sudo /usr/local/bin/run-packer

Once this has succesfully completed you will be able to launch jobs on compute
nodes with nvidia GPUs.


Oracle GPU nodes
++++++++++++++++

Due to how GPU images are managed on Oracle, you will have to build these manually and separately.
Edit ``/etc/citc/packer/config.json`` and set the ``"shape_gpu"`` entry to match whichever type of GPU node you have available.
Then run packer in GPU mode with:

.. code-block:: shell-session

   [citc@mgmt ~]$ sudo /usr/local/bin/run-packer gpu

AWS ARM nodes
++++++++++++++++

AWS provides instance types with Graviton processors which are based on the aarch64 architecture.
This requires a special image to be built which is not created by default.
To build this image, run the following:

.. code-block:: shell-session

   [citc@mgmt ~]$ sudo /usr/local/bin/run-packer aarch64

which will build the aarch64 image.

Cluster shell
-------------

A common task is to want to run commands across all nodes in a cluster.
By default you have access to `clustershell <http://clustershell.readthedocs.io/>`_.
Read the documentation there to get details of how to use the tool.

The gist is that you give it a hostname or a group and a command to run.
You can see a list of the available groups with ``cluset``:

.. code-block:: shell-session

   [citc@mgmt ~]$ cluset --list-all
   @compute
   @state:idle
   @role:mgmt

You can then run a command with ``clush``:

.. code-block:: shell-session

   [citc@mgmt ~]$ clush -w @compute uname -r
   vm-standard2-1-ad1-0001: 4.14.35-1844.2.5.el7uek.x86_64
   vm-standard2-1-ad3-0001: 4.14.35-1844.2.5.el7uek.x86_64
   vm-standard2-2-ad3-0001: 4.14.35-1844.2.5.el7uek.x86_64
   vm-standard2-2-ad2-0001: 4.14.35-1844.2.5.el7uek.x86_64
   vm-standard2-1-ad2-0001: 4.14.35-1844.2.5.el7uek.x86_64

You can combine the output from different nodes using the ``-b`` flag:

.. code-block:: shell-session

   [citc@mgmt ~]$ clush -w @compute -b uname -r
   ---------------
   vm-standard2-[1-2]-ad3-0001,vm-standard2-1-ad[1-2]-0001,vm-standard2-2-ad2-0001 (5)
   ---------------
   4.14.35-1844.2.5.el7uek.x86_64

Bear in mind that since the nodes are created afresh each time they are started,
any changes you make to a running node will not be persisted.
It will also not be able to run on any nodes that are not currently running.

Installing software on your cluster
-----------------------------------

To make software available across your cluster, the best way is to install it onto the shared filesystem at ``/mnt/shared``.
Make sure that all the dependencies for it are available either on the shared filesystem or in the base image you're using.
i.e. don't use ``yum install`` to provide dependencies.

Consider using a tool like `EasyBuild <https://easybuild.readthedocs.io>`_ or `Spack <https://spack.io/>`_ to manage your software stack.

Monitoring
----------

The cluster automatically collects data from all the nodes and makes them available in a web dashboard.

It is available at the IP address of your management node on port 3000.
Point your browser at ``http://your.mgmt.ip.address:3000`` and log in with the username *admin*.
The password for the dashboard can be found by running:

.. code-block:: shell-session

   [citc@mgmt ~]$ sudo get_secrets

Once you are logged in, you can find a dashboard showing the state of nodes by clicking on "Home" in the top-left and selecting "Slurm".

Reporting issues
----------------

Please report questions and problems to `Cluster in the Cloud GitHub Issues
<https://github.com/clusterinthecloud/support/issues>`_.

You can gather diagnostic information from your cluster which might help
pinpoint problems by running

.. code-block:: shell-session

   [citc@mgmt ~]$ sudo sosreport --only-plugins citc && sudo chown $USER /var/tmp/sosreport*

For ``case id`` put your GitHub username (if you have one). This will generate
a ``tar.xz`` file that can be downloaded and then attached to, for example, a
GitHub issue.

Destroying the whole cluster
----------------------------

.. warning::

   Please bear in mind that this will also destroy your file system which contains your user's home area
   and any data stored on the cluster.

When you've completely finished with the cluster, you can destroy it.

First you must make sure that you do not have any running jobs or compute nodes.
You can kill all running nodes by running the following on the management node:

.. code-block:: shell-session

   [citc@mgmt ~]$ /usr/local/bin/kill_all_nodes

Then, you can log out and locally run:

.. code-block:: shell-session

   $ terraform destroy aws

or

.. code-block:: shell-session

   $ terraform destroy google

or

.. code-block:: shell-session

   $ terraform destroy oracle

This command *will* ask for confirmation before destroying anything but be sure to read the list of things it's going to terminate to check that it's doing the right thing.
It will also attempt to terminate any running compute nodes you still have but make sure to check the web interface afterwards.

AWS 1-click uninstall
+++++++++++++++++++++

If you installed the cluster with the 1-click installer on AWS then you can uninstall it in a similar way by running following:

.. code-block:: shell-session

   $ ./destroy-citc.py aws 130.61.43.69 citc-terraform-useful-gnu/citc-key

but passing in the IP address of the management node that you we given when the cluster was created, as well as the SSH key to connect with.


Google 1-click uninstall
++++++++++++++++++++++++

If you installed the cluster with the 1-click installer on Google Cloud then you can uninstall it in a similar way by running the following in a Google Cloud Shell:

.. code-block:: shell-session

   $ docker run -it -e CLOUDSDK_CONFIG=/config/gcloud \
                    -v $CLOUDSDK_CONFIG:/config/gcloud \
                    clusterinthecloud/google-destroy
