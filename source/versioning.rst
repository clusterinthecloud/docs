Versioning
==========



Historical versioning
---------------------

In the past, when nodes booted they would run ``ansible-pull`` every time to set up their configuration.
The Ansible playbook backing them was accessed directly as the ``master`` branch on GitHub.
Of course, if the master branch had incompatible changes (either things which could not just be Ansible-updated to or didn't match the Terraform config) then it would not work.

To help resolve this, versioning was introduced where, on the initial creation of the cluster, a version number would be baked in.
The Ansible config would then have a long-lived branch matching that version which nodes would boot from.
The promise of the version number was then that any cluster which was created at version X would always be safe having its compute nodes boot from Ansible version X.

In early 2020, we went from configuring the nodes one boot to pre-creating node images.
This means that unless the admin actively does something (e.g. update and rerun the management Ansible or recreate the node images) things will stay in sync.

Then, in mid 2021 we switched away from using ``ansible-pull`` and instead run everything from a copy of the Ansible config stored on the management node.
Now, even re-creating the node image is safe as it will always read from the same Ansible configuration.
This means that the versioning was now much less important as the admin would have to explicitly update the repo for things to break.
