.. Hasura Platform documentation master file, created by
   sphinx-quickstart on Thu Jun 30 19:38:30 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. meta::
   :description: Reference documentation for Hasura's ``File`` microservice. The service is used to upload & download files and has built-in access control.
   :keywords: hasura, docs, fileStore, file, file upload, file download

====
File
====

The ``File`` service on Hasura let's you upload and download files. This can
be used to store user data files.

The File service, just like the data service, allows configuring access
control on files based on roles.

The ``File`` API provides the following features:

1. CRUD APIs to download, upload and delete files.
2. Role based access control per file.


.. _filestore-authz-webhooks:

Authorization webhooks
----------------------

File needs to authorize if the current user is allowed to call a ``File``
API (say to upload or download a file). This authorization is done via a
webhook.

By default, only the ``admin`` role has access to the ``File`` APIs, for other
roles the webhook has to be configured.

The File service contacts the webhook with the following parameters:

1. The File ID: the unique id of the file.
2. The operation on the file: one of ``create``, ``read``, ``delete``.

The user's information is passed in the request headers as
``X-Hasura-User-Id``, ``X-Hasura-User-Role`` and ``X-Hasura-Allowed-Roles``.

.. note::

    The headers ``X-Hasura-User-Id``, ``X-Hasura-User-Role`` and
    ``X-Hasura-Allowed-Roles`` are a Hasura platform feature to enable access
    control. To know more about it read here.


Webhook response
----------------

If the webhook returns a ``200`` response then the authorization is granted,
and if it returns ``403`` response, then the file operation is denied.

.. list-table::
   :widths: 10 10 30
   :header-rows: 1

   * - Status code
     - Authorization action
     - Description

   * - ``200``
     - Access granted
     - Success

   * - ``403``
     - Access denied
     - The message returned by the webhook is sent to the client

   * - Anything else
     - Error
     - Internal error is thrown


Setting up the webhook
----------------------

As a user, you are supposed to provide the ``File`` service with a webhook, which is a
HTTP endpoint running inside the Hasura project. By default filestore provides you with three
webhooks. They are mentioned in the manual.

If you want to create custom permissions for you files, you need to write a custom endpoint which receives the above
parameters from the ``File`` service, perform required authorization checks and return
a response. You can deploy this custom endpoint as a custom service from the
Hasura project console.

Let's say the webhook that you have deployed is available at
``http://filecheck.<your-hasura-project>.hasura-app.io/check``. Internally,
this endpoint will be available at ``filecheck.default/check``. Then the
``File`` service will call your API to authorize before doing the actual file
operation, like so:

.. code-block:: http

    GET filecheck.default/check?file_id=<file-id>&file_op=<operation> HTTP/1.1


Where ``operation`` is one of ``create``, ``read``, ``delete``.

Once you get this information about the file and the file operation, and
retrieve the user and roles information from the Hasura headers, you should
perform the required authorization checks. If you want to grant the access to
the user then the webhook should return a ``200`` response. If you want to deny
access to the user then your webhook should return a ``403`` response.


API
---

To perform the actual file operations, the following APIs are provided.

.. toctree::
  :maxdepth: 2

  api
