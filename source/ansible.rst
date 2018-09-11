Installing the software
=======================

Set up the connection
---------------------

Move to the base config directory and grab the Ansible configuration:

.. code-block:: shell-session

   $ git clone https://github.com/ACRC/slurm-ansible-playbook.git
   $ cd slurm-ansible-playbook

and start by editing the ``hosts`` file:

.. code-block:: shell-session

   $ vim hosts

Under ``[management]``, on the line that starts with ``mgmt``, set the IP address after ``ansible_host`` to the IP address from the Terraform output value ``ManagementPublicIPs``.

Under ``[compute]``, list all the hostnames from the Terraform ``ComputeHostnames`` output value.

Finally, under ``[compute:vars]``, set the IP address at the end of the ``ansible_ssh_extra_args`` line to the IP address from the Terraform ``ManagementPublicIPs`` output value.

At the end it should looks something like:

.. code-block:: ini

   [management]
   mgmt ansible_host=130.61.43.69

   [compute]
   compute001
   compute002
   compute003
   compute004

   [compute:vars]
   ansible_ssh_extra_args="-J opc@130.61.43.69"

Check whether it is working by running an Ansible ping:

.. code-block:: shell-session

   $ ansible -m ping -o all
   mgmt | SUCCESS => {"changed": false,"ping": "pong"}
   compute001 | SUCCESS => {"changed": false,"ping": "pong"}
   compute004 | SUCCESS => {"changed": false,"ping": "pong"}
   compute002 | SUCCESS => {"changed": false,"ping": "pong"}
   compute003 | SUCCESS => {"changed": false,"ping": "pong"}

Configuring the Ansible playbook
--------------------------------

.. code-block:: shell-session

   $ vim group_vars/all.yml

Set ``users`` to a list of users you want to have access on the cluster.
For each entry in the list, set ``name`` to the username you want for each user and set ``key`` to either the contents of the SSH public key file (as we did for the Terraform config) or a URL containing a list of public keys.

Set ``slurm_accounting_db_password`` to something randomly generated (such as the output of ``openssl rand -base64 32``). This is used for Slurm to communicate with its SQL database backend.

Set ``slurm_compute_nodelists`` to a list where each entry describes one hardware configuration.
For each list entry the following values are used:

nodelist
   The `list of the nodes <https://slurm.schedmd.com/slurm.conf.html#OPT_NodeName>`_ you are referring to.
   You can use nodelist syntax to refer to ranges.

memory
   The `real memory <https://slurm.schedmd.com/slurm.conf.html#OPT_RealMemory>`_ available on the node in megabytes.

sockets
   The number of `CPU sockets <https://slurm.schedmd.com/slurm.conf.html#OPT_Sockets>`_ available on the node.

cores_per_socket
   The number of `CPU cores <https://slurm.schedmd.com/slurm.conf.html#OPT_CoresPerSocket>`_ in each physical CPU.

threads_per_core
   The number of `threads <https://slurm.schedmd.com/slurm.conf.html#OPT_ThreadsPerCore>`_ available from each CPU core.

For our example, using four ``VM.Standard2.24`` VMs, we set the following:

.. code-block:: yaml

   slurm_compute_nodelists:
     - nodelist: compute[001-004]
       memory: 320000
       sockets: 1
       cores_per_socket: 24
       threads_per_core: 1

Then we set the partitions up.
For most simple cases, you only need one partition which you can set to contain all the compute nodes:

.. code-block:: yaml

   slurm_partitions:
     - name: compute
       nodelist: compute[001-004]
       default: YES

That is all we need to edit in ``group_vars/all.yml``.
Next, we edit ``management.yml``:

.. code-block:: shell-session

   $ vim group_vars/management.yml

Set the contents of the ``slurm_elastic`` variable in the same way as you did for the Terraform config in the last chapter but leave the ``config_directory`` variable set as it is.

Then set ``grafana_admin_password`` to something secure.
This will guard access to an internet-facing Grafana dashboard.

That should be all the configuration that is needed.

Run Ansible
-----------

We are now ready to run Ansible by running the following command.
It will take some time to complete but should print output along the way.

.. code-block:: shell-session

   $ ansible-playbook site.yml

You can now log into the management node at ``yourusername@mgmtipaddress``, for example

.. code-block:: shell-session

   $ ssh -A matt@130.61.43.69

Once logged in, try running the ``sinfo`` command to check that Slurm is running:

.. code-block:: shell-session

   $ [matt@mgmt ~]$ sinfo
   PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
   compute*     up   infinite      4   idle compute[001-004]

Brilliant! Start submitting jobs.
