+++++++++++++
Configuration
+++++++++++++


Cubes workspace configuration is stored in a ``.ini`` file with sections:

* ``[workspace]`` – Cubes workspace configuration
* ``[server]`` - server related configuration, such as host, port
* ``[models]`` - list of models to be loaded 
* ``[store]`` – default datastore configuration
* ``[translations]`` - model translation files, option keys in this section
  are locale names and values are paths to model translation files. See
  :doc:`localization` for more information.
* ``[model]`` (depreciated) - model and cube configuration

.. note::

    The configuration has changed. Since Cubes supports multiple data stores,
    their type (backend) is specifien in the store configuration as
    ``type`` property, for example ``type=sql``.

Quick Start
===========

Simple configuration might look like this::

    [workspace]
    model: model.json

    [store]
    type: sql
    url: postgresql://localhost/database

Workspace
=========

* ``stores`` – path to a file containing store descriptions – every section is
  a store with same name as the section
* ``models_path`` – path to a directory containing models. If this is set to
  non-empty value, then all model paths specified in ``[models]`` are prefixed
  with this path
* ``log`` - path to a log file
* ``log_level`` - level of log details, from least to most: ``error``, 
    ``warn``, ``info``, ``debug``

* ``timezone`` - name of default time zone. Used in date and time operations,
  such as :ref:`named relative time <named_relative_time>`.
* ``first_weekday`` – name of first day of the week. It can also be a number
  where 0 is Monday, 6 is Sunday

* ``authorization`` – authorization method to be used

Namespaces
----------

If not specified otherwise, all cubes share the same default namespace. There
names within namespace should be unique. For simplicity and for backward
compatibility reasons there are two cube lookup methods: `recursive` and
`exact`. `recursive` method looks for cube name in the global namespace first
then traverses all namespaces and returns the first cube found. `exact`
requires exact cube name with namespace included as well. The option that
affects this behavior is: ``lookup_method`` which can be ``exact`` or
``recursive``.

Info
----

The info JSON file might contain:

* ``label`` – server's name or label
* ``description`` – description of the served data
* ``copyright`` – copyright of the data, if any
* ``license`` – data license
* ``maintainer`` – name of the data maintainer, might be in format ``Name
  Surname <namesurname@domain.org>``
* ``contributors`` - list of contributors
* ``keywords`` – list of keywords that describe the data
* ``related`` – list of related or "friendly" Slicer servers with other open
  data – a dictionary with keys ``label`` and ``url``.
* ``visualizers`` – list of links to prepared visualisations of the
  server's data – a dictionary with keys ``label`` and ``url``.


Models
======

Section ``[models]`` contains list of models. The property names are model
identifiers within the configuration (see ``[translations]`` for example) and
the values are paths to model files.

Example::

    [models]
    main: model.json
    mixpanel: mixpanel.json

If root ``models_path`` is specified in ``[workspace]`` then the relative
model paths are combined with the root. Example::

    [workspace]
    models_path: /dwh/cubes/models

    [models]
    main: model.json
    events: events.json

The models are loaded from ``/dwh/cubes/models/model.json`` and
``/dwh/cubes/models/events.json``.


Server
======

* ``json_record_limit`` - number of rows to limit when generating JSON 
    output with iterable objects, such as facts. Default is 1000. It is 
    recommended to use alternate response format, such as CSV, to get more 
    records.
* ``modules`` - space separated list of modules to be loaded (only used if 
    run by the ``slicer`` command)
* ``prettyprint`` - default value of ``prettyprint`` parameter. Set to 
    ``true`` for demonstration purposes.
* ``host`` - host where the server runs, defaults to ``localhost``
* ``port`` - port on which the server listens, defaults to ``5000``
* ``allow_cors_origin`` – Cross-origin resource sharing header. Other related
  headers are added as well, if this option is present.

* ``authentication`` – authentication method (see below for more information)

* ``pid_file`` – path to a file where PID of the running server will be
  written. If not provided, no PID file is created.

Model
=====

.. note::

    This section is depreciated. Use `model` in ``[workspace]`` for single
    model file or ``[models]`` for multiple models.

* ``path`` - path to model .json file
* ``locales`` - comma separated list of locales the model is provided in. 
    Currently this variable is optional and it is used only by experimental 
    sphinx search backend.

Data stores
===========

There might be one or more store configured. The section ``[store]``
of the ``cubes.ini`` file describes the default store. Multiple stores are
configured in a separate ``stores.ini`` file. The path to the stores
configuration file might be specified in a variable ``stores`` of the
``[workspace]`` section

Properties of the datastore:

* ``type`` (required) – data store type, such as ``sql``
* ``model`` – model related to the datastore
* ``namespace`` – namespace where the store's cubes will be registered
* ``model_provider`` – model provider type for the datastore

Example SQL store::

    [store]
    type: sql
    url: postgresql://localhost/data
    schema: cubes

For more information and configuration options see :doc:`backends/sql`.

Example mixpanel store::

    [datastore]
    type: mixpanel
    model: mixpanel.json
    api_key: 123456abcd
    api_secret: 12345abcd

Multiple Slicer stores::

    [datastore_slicer1]
    type: slicer
    url: http://some.host:5000

    [datastore_slicer2]
    type: slicer
    url: http://other.host:5000

The cubes will be named `slicer1.*` and `slicer2.*`. To use specific
namespace, different from the store name::

    [datastore_slicer3]
    type: slicer
    namespace: external
    url: http://some.host:5000

Cubes will be named `external.*`

To specify default namespace::

    [datastore_slicer4]
    type: slicer
    namespace: default.
    url: http://some.host:5000

Cubes will be named without namespace prefix.

Example
=======

Example configuration file::

    [workspace]
    model: ~/models/contracts_model.json

    [server]
    reload: yes
    log: /var/log/cubes.log
    log_level: info

    [store]
    type: sql
    url: postgresql://localhost/data
    schema: cubes

Authentication and Authorization
================================

Cubes provides mechanisms for authentication at the server side and
authorization at the workspace side.

Configure authorization:

.. code-block:: ini

    [workspace]
    authorization: simple

    [authorization]
    rights_file: /path/to/access_rights.json

Built-in authorization methods:

* ``none`` – no authorization
* ``simple`` – uses a JSON file with per-user access rights

The simple authorization has following options:

* ``rights_file`` – path to the file with access rights
* ``roles_file`` – path to the file with roles
* ``identity_dimension`` – name of a flat dimension that will be used for cell
  restriction. Key of that dimension should match the identity.
* ``order`` – ``allow_deny`` or ``deny_allow`` (default)
* ``guest`` – name of a guest role. If specified, then this role will be used
  for all unknown (not specified in the file) roles.

Configure authentication:

.. code-block:: ini

    [server]
    authentication: parameter

    [authentication]
    # additional authentication parameters

Built-in server authentication methods:

* ``none`` – no authentication
* ``http_basic_proxy`` – HTTP basic authentication. Will pass the `username`
  to the authorizer
* ``pass_parameter`` – authentication withot verification, just a way of
  passing an URL parameter to the authorizer. Default parameter name is
  ``api_key``

.. note::

    When you have authorization method specified and is based on an users's
    indentity, then you have to specify the authentication method in the
    server. Otherwise the authorizer will not receive any identity and might
    refuse any access.


Server Query Logging
====================

Logging handlers for server requests have sections with name prefix
`query_log`. All sections with this prefix (including section named as the
prefix) are collected and chained into a list of logging handlers. Required
option is `type`. You might have multiple handlers of the same time.

Logging types:

* `default` – log using Cubes logger
* `csv_file` – log into a CSV file
* `sql` – log into a SQL table

CSV request logger options:

* `path` – path to a CSV file that will be appended (and created if necessary)

SQL request logger options:

* `url` – database URL
* `table` – database table
* `dimensions_table` – table with dimension use (optional)

Tables are created automatically.
