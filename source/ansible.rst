Finalising the setup
====================

Terraform will have automatically started the cluster software configuration step.
It will run in the background and will take some time to complete.
In the meantime, you can connect to the custer and follow its progress.

Final configuration step
------------------------

You can log into the management node at ``yourusername@mgmtipaddress``,
using the IP address that terraform printed at the endo of its run.
For example:

.. code-block:: shell-session

   $ ssh -A opc@130.61.43.69

Once logged in, you can run the ``finish`` script:

.. code-block:: shell-session

   [opc@mgmt ~]$ ./finish

It will most likely tell you that the nodes have not finished configuring.
If the ``finish`` script is not there, wait a minute or two and it should appear.

To follow the progress, you can look at the file ``ansible-pull.log`` in ``opc``'s home directory.

You can keep on running trying to run ``finish`` until all nodes have finished configuring.
Once they have, you need to tell the system about what user accounts you want to create.

Copy the ``users.yml.example`` file to ``users.yml``:

.. code-block:: shell-session

   [opc@mgmt ~]$ cp users.yml.example users.yml

and edit it to contain the users you want.
For the ``key`` attribute you can specify a URL of a file which contains a list of public keys (such as provided by GitHub)
or explicitly provide a public key inline.
For example, it might look like:

.. code-block:: yaml

   ---
   users:
     - name: matt
       key: https://github.com/milliams.keys
     - name: anotheruser
       key: ssh-rsa UmFuZG9tIGtleSBjb250ZW50cy4gUHV0IHlvdXIgb3duIGtleSBpbiBoZXJlIG9idmlvdXNseS4= user@computer

Run ``finish`` again and it should create those users across the system:

.. code-block:: shell-session

   [opc@mgmt ~]$ ./finish

Once it has succeeded, log out and try logging as one of those users.

Check Slurm is running
----------------------

.. code-block:: shell-session

   $ ssh -A matt@130.61.43.69

Once logged in, try running the ``sinfo`` command to check that Slurm is running:

.. code-block:: shell-session

   [matt@mgmt ~]$ sinfo
   PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
   compute*     up   infinite      4   idle compute[001-004]

Brilliant! Start submitting jobs.

Check out the information on :doc:`running the cluster <running>`.
