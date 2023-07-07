Running Ansible
===============

Both the management node and the compute nodes are configured using `Ansible <https://docs.ansible.com/ansible/>`_.
This is a configuration tool which does things like putting config files in the right place, installing software and creating user accounts.
The configuration for it is stored on GitHub at `github.com/clusterinthecloud/ansible <https://github.com/clusterinthecloud/ansible>`__

A local clone of the Ansible configuration is made early in the setup of CitC in the folder ``/root/citc-ansible``.
All further configuration of the system is made in reference to that folder.

Under normal circumstances, you should not have to deal with Ansible manually, but if something goes wrong or you are adding a new feature then you will need this.

Updating Ansible
----------------

If any bug fixes or new features are added to the version of Ansible on GitHub, they will not automatically become available on your cluster.
In order to retrieve the changes, you must run:

.. code-block:: shell-session

   $ sudo /root/update_ansible_repo

Configuring the management node
-------------------------------

The management node is configured automatically by a run of Ansible which is started when the node is first created.

To re-run this manually (for example to retry a failure, or if you've updated the config) you can run:

.. code-block:: shell-session

   $ sudo /root/run_ansible

This will run Ansible in the foreground and also send its output to the usual ``/root/ansible-pull.log`` file.

This will, by default, also re-build the compute node image which a, may not be what you want and b, will take longer.
You can disable the running of the image rebuild step with:

.. code-block:: shell-session

   $ sudo /root/run_ansible --skip-tags=packer

Compute nodes
-------------

The same Ansible configuration is used by Packer when creating compute node images.
The process described under :doc:`node-images` uses this same configuration folder.
