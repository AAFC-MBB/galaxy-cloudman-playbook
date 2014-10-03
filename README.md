A playbook for building the *Galaxy on the Cloud*
=================================================

This playbook is used to build the components required to run
[CloudMan][cloudman] and [Galaxy on the Cloud][goc]. The playbook is
used by th [Galaxy project][gp] itself and is intended for anyone
wanting to deploy their own instance, either on a public or private
cloud.

There are several roles contained in this playbook; the roles manage
the build process of different components. To get a complete built,
you will first need to build the machine image. Then launch an instance
of that machine image and build the rest of the componts off of it.
More insturctions about the build process can be found [here][building].

Machine Image
-------------
To build an image, make sure the default values provided in the
`roles/cloudman_image/defaults` directory suite you. Next, create a copy of
`inventory/cloud-builder.sample` as `inventory/cloud-builder`, launch a new
instance (this role has been developed and tested on Ubuntu 14.04) and set the
instance IP address for `cloudman-image` host in `cloud-builder`. Finally, run
the role with

    ansible-playbook -i inventory/cloud-builder cloud.yml --tags "cloudman" --extra-vars vnc_password=<choose a password> --extra-vars cm_cleanup=yes

On average, the build time takes about 30 minutes. *Note that after the playbook
has run to completion, you will no longer be able to ssh into the instance!* If
you still need to ssh, omit `--extra-vars cm_cleanup=yes` from the above command.
Before creating the image, however, you must rerun the playbook with that flag set.

### Customizing
A configuration file exposing adjustable options is available under
`roles/cloudman_image/defaults/cloudman.yml`. Besides allowing you to set some
of the image configuration options, this file allows you to easily control which
steps of the image building process run. This can be quite useful if a step fails
and you want to rerun only it or if you're just trying to run a certain steps.

Galaxy File System (galaxyFS)
-----------------------------
Launch an instance of the machine image built in the previous step and attach a
new volume to it. Create a (`XFS`) file system on that volume and mount it
(under `/mnt/galaxy`). Note that this can also be done from the CloudMan's
Admin page by adding a new-volume-based file system. Set the lauched instance
IP in `inventory/cloud-builder` for `cloudman-instance` and run the role with

    ansible-playbook -i inventory/cloud-builder cloud.yml --tags "galaxyFS" --extra-vars psql_galaxyftp_password=<choose a password>

After the run has completed (typically ~15 minutes), you can start the Galaxy
application by hand and install desired tools via the Tool Shed. To start Galaxy,
change into the `galaxy` user and from the `galaxy_server_dir` (e.g.,
*/mnt/galaxy/galaxy-app*) just run `sh run.sh`.

Once the tools have been installed, you need to create a snapshot of the file
system. Before doing so, stop any services that might still be using the file
system, unmount the file system and create a snapshot of it from the Cloud's console.

### Customizing
This role requires a number of configuration options for the Galaxy application,
CloudMan application, PostgreSQL the database, as well as the glue linking those.
The configuration options have been aggregated under
`group_vars/cloudman-instance.yml` and represent represent reasonable defaults.
Keep in mind that changing the options that influence how the system is deployed
and/or managed may also require changes in CloudMan.


[cloudman]: http://usecloudman.org/
[goc]: https://wiki.galaxyproject.org/Cloud
[gp]: http://galaxyproject.org/
[building]: https://wiki.galaxyproject.org/CloudMan/Building
