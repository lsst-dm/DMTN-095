..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

   Kubernetes and Container Guidelines

   This document is the preliminary version.  If you know of any additional information that needs to be in this document, or if you see something that needs to be corrected, please make a note of it so that it can be included in this document.  When all those comments are collected,  the document will be turned into a DMTN.



Kubernetes and Container Guidelines
===================================

This document is a set of guidelines for building and using containers on the
Kubernetes clusters. It contains background on the intended use of each
cluster,  guidelines for building containers, and guidelines issued by the
security group and the Kubernetes admin staff.

Clusters
--------

We currently have two Kubernetes clusters.  One is used for development, and
the Kubernetes admin staff uses another much smaller cluster for internal
testing.  We intend to deploy an additional pre-production Kubernetes cluster.

These are all independent clusters, rather than segmented portions of one 
cluster.  We need to be able to have independent Kubernetes versions for each
of the clusters. The production Kubernetes cluster may be running version
running version X.2, the development cluster may be running version X.4 and 
there may be a deployment of version Z.0 on the testing cluster.  If this were
one cluster, we would need to have each of those to have the same
version of the Kubernetes system software.


The pre-production cluster will be used through a VPN for demos and short-term use by users outside of LSST development staff. This cluster will always available for outside users, running the current stable release of the software. The intent of having this cluster is to provide a stable environment for guest users and demos without impacting other ongoing development. There will be limited developer access to this cluster to help perform software upgrades, but we expect this task to transition to other staff when this reaches production.

The development cluster will be very much the same as it is now.  We intend to implement the best practices guidelines gathered from the development team and listed below.   Some of these guidelines are “this is generally accepted practice” and others have put into place by security.

The third internal testing cluster will be a small cluster intended to verify update procedures of the Kubernetes and its supporting system software. This is for use by Kubernetes admin staff.  

The time cycle of upgrading the production cluster’s LSST software will be done independently of the Kubernetes system software updates.  The LSST software may require updates to that the Kubernetes system software in order to use new features and/or fix bugs. Updates are intended to be done on the development cluster in consultation with the development staff and will include regression tests before proceeding to upgrade the production cluster.  


Guidelines
----------

Container contents
^^^^^^^^^^^^^^^^^^

Each container should have one purpose, so trying to fit an entire application suite into one container should be avoided.  This permits individual parts to be upgraded more easily.  So in other words, if the application uses Services A, B and C which all work with each other (database, messaging system, Redis), put those in separate containers.

At the same time, don’t install extra packages that will be unused (i.e., don’t install a compiler when all you need is a running service).

Images and Libraries
^^^^^^^^^^^^^^^^^^^^

It is essential that all images and libraries used to build LSST docker containers are up to date and come from trusted sources.  Both Docker Store and Docker Hub have official repositories for CentOS base operating system images, and they each have been scanned for vulnerabilities.  It appears that Docker Hub images go through scans more frequently.  Nearly all of these images have tagged vulnerabilities, so we need to have a policy in place about whether those packages need to be patched (if possible) or removed if they are unused.  It is recommended to only use images from Docker Hub repositories tagged "official."


The DM stack has local versions of third party software that we depend on as part of the build of the stack. This is to ensure we always have a copy of the package that’s compatible with our build.  We’ll need to do something similar for images used to build containers so that we know we can always build containers with specific images.  Otherwise if the image was ever deleted from or replaced on a remote repository, we might not be able to build the same container. We should download and preserve images locally, and store containers in a local docker registry.  

We will deploy local Docker registries for internal operations in production. Local registries will give us faster download times, better security and better control of the service itself. If we primarily relied on an outside registry, service (or even business) failures would prevent us from operating through no fault of our own. Security staff should vet all containers in these registries.

We will continue distribute LSST stack containers via Docker Hub, but the having the containers local gives us more control in our day to day usage.  

Network Ports
^^^^^^^^^^^^^

Requests for exposing network ports exposed to the public internet should be requested through the K8s sysadmin.


Roles and Security
^^^^^^^^^^^^^^^^^^

Role-based access permissions will be introduced soon.  These permissions are both for security, and to prevent services (and users) in one namespace accidentally affecting other services and users in another namespace. K8s admin staff is working on the guidelines for this.

No developer process should run as root in an executing container, and users will have roles restricted to their needs for development (as opposed to having full admin access to all parts of the K8s cluster).  This is to prevent accidental damage that might be caused by a command executed in the wrong place. It will also help restrict access to more sensitive parts of the system (system logs, secrets, etc.), to which users should not have access.


Access to File Sets
^^^^^^^^^^^^^^^^^^^

/project
/project/shared
/datasets
/scratch
/jhome
/Software

/project (read-write): All users are put into one or more groups(s), and have directory access below the “project” file set to each group to which they belong. This access is not unrestricted to all of “project.”

/project/shared/: All users have read access to all directories.

/datasets (read-only): Individuals/groups have different types of access, depending on their standing in the project. Some datasets are restricted for some period to LSST  collaborators before they become available to other parts of the project.

/scratch (read-write): Currently, LSST developers have directory access below this file set. Science Users have no access to this file set.  Scratch space is temporary and is purged on a regular schedule.

/jhome (read-write): LSST Developers and Science Users have access to the jhome file set. Currently, LSST developers have this as a separate mount point named jhome which is accessible from their accounts on lsst-dev. When they log in, their home directory is in /home/{user}. Users of lsst-dev also have access to jhome. LSST Science Users can only access the “jhome” file set through the accounts they access on the K8s commons and have no visibility to /home. In production, this will be the case for all users. An LSST Science User has write access to write to /project and /scratch, and 100GB of disk space.

/software (read-only): All developers have read-only access to this file set. This access is currently not available via Jupyter Notebook. This access may be added in the future to access the batch system commands.

Namespaces
^^^^^^^^^^

Kubernetes namespaces allow partitioning of applications into their areas, with unique resource names within that namespace. For example, JupyterLab is deployed in the jupyter-lsst namespace. The development groups for the PDAC are already implementing namespaces for their applications.

As of this writing, no access control enforcement is available for namespaces in Kubernetes. Anyone (or any pod) with privileges on the cluster can access any namespace and its resources. Currently, we afford some measure of restricted user access by employing the use of Kubernetes namespace contexts. When working within a namespace, only resources in that namespace can be seen and accessed. Users can still override this or move into new contexts, so this is not meant to be a substitute for real ACL. We expect to implement ACL for namespaces when Kubernetes deploys that feature in a future release.




.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :encoding: latex+latin
..    :style: lsst_aa
