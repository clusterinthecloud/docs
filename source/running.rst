Running the cluster
===================

Now that you have a cluster which is up and running,
it's worth knowing what you can do with it.

Slurm elastic scaling
---------------------

Slurm is configured to use its `elastic computing <https://slurm.schedmd.com/elastic_computing.html>`_ mode.
This allows Slurm to automatically turn off any nodes which are not currently being used for running jobs
and turn on any nodes which are needed for running jobs.
This is particularly useful in the cloud as a node which has been shut down will not be charged for.

Slurm does this by calling a script ``/usr/local/bin/startnode`` as the ``slurm`` user.
If necessary, you can call this yourself from the ``opc`` user like:

.. code-block:: shell-session

   [opc@mgmt ~]$ sudo -u slurm /usr/local/bin/startnode compute001

to turn on the node ``compute001``.

You should never have to do anything to explicitly shut down the cluster,
it will automatically turn off all nodes which are not in use after a timeout.
The management node will always stay running which is why it's worth only using a relatively cheap VM for it.

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

You can combine the output from different nodes using the ``-b`` flag:

.. code-block:: shell-session

   [opc@mgmt ~]$ clush -w @compute -b uname -r
   ---------------
   compute[001-002] (2)
   ---------------
   3.10.0-862.2.3.el7.x86_64

Installing software on your cluster
-----------------------------------

In order to do any actual work you will likely need to install some software.
There are many ways to get this to work but I would recommend either using ``clush`` to install the software
or, preferably, create a local Ansible playbook which installs it for you across the cluster.

In the latter case, you can use ``/home/opc/hosts`` as an inventory file and point your playbook to use it.

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
