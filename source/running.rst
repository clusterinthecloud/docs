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

   #SBATCH --job-name=test
   #SBATCH --partition=compute
   #SBATCH --nodes=1
   #SBATCH --ntasks-per-node=2
   #SBATCH --cpus-per-task=1
   #SBATCH --time=10:00
   #SBATCH --exclusive

   echo start
   srun -l hostname
   echo end

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

Slurm elastic scaling
---------------------

Slurm is configured to use its `elastic computing <https://slurm.schedmd.com/elastic_computing.html>`_ mode.
This allows Slurm to automatically terminate any nodes which are not currently being used for running jobs
and create any nodes which are needed for running jobs.
This is particularly useful in the cloud as a node which has been terminated will not be charged for.

Slurm does this by calling a script ``/usr/local/bin/startnode`` as the ``slurm`` user.
If necessary, you can call this yourself from the ``opc`` user like:

.. code-block:: shell-session

   [opc@mgmt ~]$ sudo scontrol update NodeName=vm-standard2-1-ad1-0001 State=POWER_ON

to turn on the node ``vm-standard2-1-ad1-0001``.

You should never have to do anything to explicitly shut down the cluster,
it will automatically terminate all nodes which are not in use after a timeout.
The management node will always stay running which is why it's worth only using a relatively cheap VM for it.

The rate at which Slurm shuts down is managed in ``/mnt/shared/etc/slurm/slurm.conf`` by the ``SuspendTime`` parameter.
See the `slurm.conf <https://slurm.schedmd.com/slurm.conf.html>`_ documentation for more details.

Cluster shell
-------------

A common task is to want to run commands across all nodes in a cluster.
By default you have access to `clustershell <http://clustershell.readthedocs.io/>`_.
Read the documentation there to get details of how to use the tool.

The gist is that you give it a hostname or a group and a command to run.
You can see a list of the available groups with ``cluset``:

.. code-block:: shell-session

   [opc@mgmt ~]$ cluset --list-all
   @compute
   @state:idle
   @role:mgmt

You can then run a command with ``clush``:

.. code-block:: shell-session

   [opc@mgmt ~]$ clush -w @compute uname -r
   compute001: 3.10.0-862.2.3.el7.x86_64
   compute002: 3.10.0-862.2.3.el7.x86_64
   compute003: 3.10.0-862.2.3.el7.x86_64
   compute004: 3.10.0-862.2.3.el7.x86_64

You can combine the output from different nodes using the ``-b`` flag:

.. code-block:: shell-session

   [opc@mgmt ~]$ clush -w @compute -b uname -r
   ---------------
   compute[001-004] (4)
   ---------------
   3.10.0-862.2.3.el7.x86_64

Installing software on your cluster
-----------------------------------

.. todo::

    put it in /mnt/shared

    we should make an ansible-pull hook

Performance metrics
-------------------

The cluster automatically collects data from all the nodes and makes them available in a web dashboard.

It is available at the IP address of you management node on port 3000.
Point your browser at http://your.mgmt.ip.address:3000 and log in with the username *admin* and the password *admin*.
You will be prompted to create a new password before you continue.

Destroying the whole cluster
----------------------------

.. warning::

   Please bear in mind that this will also destroy your file system which contains your user's home area
   and any data stored on the cluster.

When you've completely finished with the cluster,
you can destroy it using Terraform.

.. code-block:: shell-session

   $ terraform destroy
