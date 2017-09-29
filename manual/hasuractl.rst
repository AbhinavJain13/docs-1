:orphan:

.. meta::
   :description: Reference documentation for using Hasura's command line tooling, hasuractl
   :keywords: hasura, docs, CLI, HasuraCTL, hasuractl

.. _hasuractl:

.. highlight:: bash

hasuractl
=========

``hasuractl`` is the command line tool for the Hasura platform.

Starting v0.15, it is the primary mode of managing Hasura projects and Hasura clusters.

.. _hasuractl-installation:

Installation
------------

.. note::

   ``git`` is a dependency for ``hasuractl``. If you don't have it installed, follow the instructions `here <https://git-scm.com/book/id/v2/Getting-Started-Installing-Git>`_.

.. _hasuractl-installation-linux:

Linux
~~~~~

Install the latest ``hasuractl`` using the following command:

.. code:: bash

    # curl 
    $ curl -L https://hasura.io/install.sh | bash 
    # OR
    # wget
    $ wget -qO- https://hasura.io/install.sh | bash


    # This command will download a bash script and execute it, which will in turn download the latest version of `hasuractl` and install it into `/usr/local/bin`. You will be prompted for the root password to complete installation.

.. _hasuractl-installation-windows:

Windows
~~~~~~~

Download `hasuractl.exe <https://storage.googleapis.com/hasuractl/latest/windows-amd64/hasuractl.exe>`_.
and place it in your ``PATH``. Refer to this `video <https://drive.google.com/file/d/0B_G1GgYOqazYUDJFcVhmNHE1UnM/view>`_
if you need help with the installation on Windows.

.. note::

    It is recommended to use `git-bash <https://git-scm.com/download/win>`_ on Windows to execute hasuractl commands.

.. _hasuractl-installation-macos:

Mac OS
~~~~~~

Run the following command to install ``hasuractl``:

.. code:: bash

    # curl 
    $ curl -L https://hasura.io/install.sh | bash 
    # OR
    # wget
    $ wget -qO- https://hasura.io/install.sh | bash


    # This command will download a bash script and execute it, which will in turn download the latest version of `hasuractl` and install it into `/usr/local/bin`. You will be prompted for the root password to complete installation.

.. _hasuractl-getting-started:

Getting started
---------------

.. _hasuractl-getting-started-create-project:

Create a project and cluster
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are new to ``hasuractl`` and have not created any projects or clusters before, continue. Otherwise jump to <another-location>.

1. Install ``hasuractl``
2. Login using your Hasura account:

.. code:: bash

   $ hasuractl login

3. Create a Hasura project and get a trial cluster, also change to the project directory:

.. code:: bash

   $ hasuractl create my-project --type=trial 
   $ cd my-project 

   # This command creates a directory called my-project, creates a trial cluster called `hasura` for you, adds it to the project and sets it as default.

4. Open the API console:

.. code:: bash

   $ hasuractl api-console 
          
Using the API console, you can try out Hasura APIs for Auth, Data, File and Notify. You can also create and manage tables for your database, see users in your cluster etc.

.. _hasuractl-getting-started-deploy-code:

Deploy custom code
~~~~~~~~~~~~~~~~~~

For hosting your own code or static HTML websites, Hasura provides ready-made quickstart templates for a variety of frameworks. Find all the quickstart templates `here <https://github.com/hasura/quickstart-docker-git>`_

You can add a template along with it's source code to your newly created Hasura project and deploy it to the cluster. These templates are deployed as micro-services on Hasura platform. Changes to the source code can be re-deployed using ``git push``.

1. Initialize a git repo inside your project

.. code:: bash

   $ git init 

2. Add the service and create it on the cluster:

.. code:: bash

   # for e.g., deploy a Python Flask based web server, name it api
   $ hasuractl service quickstart api --template python-flask

   # This command downloads the template, copies it into ``services`` directory in the project, creates this service on the cluster, adds a URL route for it, adds your SSH key to the cluster, creates a git remote for you to push and creates an initial commit for the code.

3. Deploy the code

.. code:: bash

   $ git push hasura master

   # Your service will be live at https://api.<cluster-name>.hasura-app.io

4. Deploy changes

Make changes to the source code in ``service/python-flask`` directory, commit them and push again:

.. code:: bash

   $ git add <files>
   $ git commit -m "<commit-message>"
   $ git push hasura master

.. note::

   You can find all the available quickstart templates here: `https://github.com/hasura/quickstart-docker-git <https://github.com/hasura/quickstart-docker-git>`_

Understanding a Hasura project
------------------------------

A *"project"* is a *"gittable"* directory in the file system, which captures all the information regarding clusters, services and migrations. It can also be used to keep source code for custom services that you write.

Creating a project
~~~~~~~~~~~~~~~~~~

.. code:: bash

   $ hasuractl create my-project 

   # creates a directory called `my-project` and initialize an empty Hasura project
   
.. note::

   You should initialize a git repo in the Hasura project directory (or add Hasura project directory to an existing git repo) so that the contents can be version controlled. You can then share this repo with others working on the same project.

Files and directories
~~~~~~~~~~~~~~~~~~~~~

The project (a.k.a. project directory) has a particular directory structure and it has to be maintained strictly, else hasuractl would not work as expected. A representative project is shown below:

.. code:: bash

   .
   ├── hasura.yaml
   ├── clusters
   │   ├── production/
   │   └── staging
   │       ├── .kubecontext
   │       ├── domains.yaml
   │       ├── gateway.yaml
   │       ├── nginx-directives.yaml
   │       ├── remotes.yaml
   │       ├── routes.yaml
   │       ├── auth.yaml
   │       ├── notify.yaml
   │       ├── filestore.yaml
   │       ├── authorized_keys 
   │       └── services
   │           ├── adminer
   │           │   ├── deployment.yaml
   │           │   └── service.yaml
   │           └── flask
   │               ├── deployment.yaml
   │               └── service.yaml
   ├── migrations
   │   ├── 1504788327_create_table_user.down.yaml
   │   ├── 1504788327_create_table_user.down.sql
   │   ├── 1504788327_create_table_user.up.yaml
   │   └── 1504788327_create_table_user.up.sql
   └── services
       ├── adminer/
       └── flask
           ├── app/
           ├── docker-config.yaml
           ├── Dockerfile
           └── README.md

* ``hasura.yaml``
  
  * Stores some metadata about the project, like name and default cluster
    
* ``clusters``
  
  * A *"cluster"* is a kubernetes cluster with Hasura platform installed on it
  * clusters directory holds information about all the clusters
  * Each sub-directory denotes a cluster added to the project
  * ``staging``
    
    * Directory that holds configuration for a cluster called staging, named after the cluster alias
    * ``.kubecontext``
      
      * Actual kubernetes context name is stored in this file
        
    * ``authorized_keys``
      
      * SSH keys allowed to access the cluster
      * One public key per line
        
    * ``*.yaml``
      
      * Configuration for the cluster, split into various yaml files
        
    * ``services``
      
      * Directory that holds kubernetes configurations for microservices added to this cluster
      * Each sub directory contains yaml spec files for a service
      * ``adminer``

        * Contains ``deployment.yaml`` and ``service.yaml`` for adminer service
 
* ``migrations``

  * Database migration files are kept in this directory
    
* ``services``

  * Default directory to store source code for custom microservices
  * Each sub-directory contains source code and *Dockerfile*
  
*hasuractl doesn't consider any other files or directories outside of those mentioned above*


Clusters
--------

A *"cluster"* is a Kubernetes system with Hasura platform installed on it. If you want to know more about how Hasura use Kubernetes, refer to our :ref:`architecture docs <platform-architecture>`.

Create a cluster
~~~~~~~~~~~~~~~~

You can create a free trial cluster using ``hasuractl`` for evaluation and development purposes. Hasura will create a virtual machine on it's cloud infrastructure, install kubernetes and Hasura on it and allot it to your Hasura account. Assuming you have already logged in and created a project called *my-project*,

.. code:: bash

   $ cd my-project
   $ hasuractl cluster create --type=trial


Add cluster to project
~~~~~~~~~~~~~~~~~~~~~~

All your clusters are visible on the `Hasura Dashboard <https://dashboard.hasura.io>`_. You can add a cluster listed on your dashboard to your project by executing the following command from inside the project directory:

.. code:: bash

   $ hasuractl cluster add [cluster-name] -c [cluster-alias]

   # creates a sub-directory named `[cluster-alias]` inside the `clusters` directory, adds the cluster configuration files into it

``[cluster-name]`` is the name shown on the Hasura Dashboard and ``[cluster-alias]`` is the name you want to attach to the cluster for easier access. For example, a cluster named ``caddy89`` can be added to the project with an alias ``dev`` and you will be referring to this cluster as ``-c dev`` in all the other ``hasuractl`` commands.
