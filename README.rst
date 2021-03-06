=================================
Navigator Integration Application
=================================

Introduction
============

The Cask™ Data Application Platform (CDAP) is an integrated, open source application
development platform for the Hadoop ecosystem that provides developers with data and
application abstractions to simplify and accelerate application development.

Navigator Integration App is one such application built by the team at Cask for bridging CDAP Metadata
with Cloudera's data governance tool, Navigator. The Navigator Integration App is a CDAP-native application
that uses a real-time Flow to fetch the CDAP Metadata and write it to Navigator.

Resources
---------
- `Overview of CDAP Metadata
  <http://docs.cask.co/cdap/current/en/developers-manual/building-blocks/metadata-lineage.html#metadata>`__
- `Overview of CDAP Audit Logging
  <http://docs.cask.co/cdap/current/en/developers-manual/building-blocks/audit-logging.html#audit-logging>`__
- `Cloudera Navigator <http://www.cloudera.com/products/cloudera-navigator.html>`__
- `Real-time processing using Flows
  <http://docs.cask.co/cdap/current/en/developers-manual/building-blocks/flows-flowlets/index.html>`__
- `Navigator SDK <https://github.com/cloudera/navigator-sdk>`__


Getting Started
===============

Prerequisites
-------------
To use Navigator Integration App, you need CDAP version 4.0.x or higher and Navigator version of 2.4.0 or greater.

Metadata Publishing to Kafka
----------------------------
Navigator Integration App contains a flow that subscribes to the TMS (Transactional Messaging System, a CDAP component)
topic to which the CDAP Audit Logging system publishes metadata changes. Before using this application, you should 
`enable audit publishing in CDAP
<http://docs.cask.co/cdap/current/en/developers-manual/building-blocks/audit-logging.html#audit-logging-configuring-audit-publishing>`__.

Building Plugins
----------------
You begin by building directly from the latest source code::

  git clone https://github.com/caskdata/cdap-navigator.git
  cd cdap-navigator
  mvn clean package

After the build completes, you will have a JAR under the ``target/`` directory.

Deploying the Navigator Integration App
---------------------------------------

Step 1: Start by deploying the artifact JAR (either built from source or by downloading the released JAR from Maven).
Deploy the JAR using the CDAP CLI::

  > load artifact target/navigator-<version>-jar


Step 2: Create an application configuration file that contains:

- Audit Log Config (``auditLogConfig``): TMS consumer flowlet configuration information
  (information about where we can fetch audit messages)
- Navigator Config (``navigatorConfig``): Information required by the Navigator Client to publish data to Navigator

Example application configuration file::

  {
    "config": {
      "auditLogConfig": {
        "offsetDataset" :  "_auditOffset",
        "limit" : 200
      },
      "navigatorConfig": {
        "navigatorHostName": "navigatormetadataserver",
        "username": "abcd",
        "password": "1234"
      }
    }
  }

**Audit Log Config:**

This key contains a property map with these (optional) properties:

- ``namespace``: TMS topic namespace to which audit messages are published; default is ``system`` which corresponds
  to the namespace topic used in CDAP for audit messages
- ``topic``: TMS Topic to which CDAP audit messages are published; default is ``audit`` which
  corresponds to the default topic used in CDAP for audit messages
- ``offsetDataset``: Name of the dataset where TMS offsets are stored; default is ``auditOffset``
- ``limit``: Maximum number of audit messages to read in a single read in the consumer flowlet. By default it is set
  to 100.

**Navigator Config:**

This key contains a property map with these properties:

Required Properties:

- ``navigatorHostName``: Navigator Metadata Server hostname
- ``username``: Navigator Metadata Server username
- ``password``: Navigator Metadata Server password

Optional Properties:

- ``navigatorPort``: Navigator Metadata Server port; default is ``7187``
- ``autocommit``: Navigator SDK's autocommit property; default is ``true``
- ``namespace``: Navigator namespace; default is ``CDAP``
- ``applicationURL``: Navigator Application URL; default is ``http://navigatorHostName``
- ``fileFormat``: Navigator File Format; default is ``JSON``
- ``navigatorURL``: Navigator URL; default is ``http://navigatorHostName:navigatorPort/api/v8``
- ``metadataParentURI``: Navigator Metadata Parent URI; default is ``http://navigatorHostName:navigatorPort/api/v8/metadata/plugin``

Step 3: Create a CDAP application by providing the configuration file::

  > create app metaApp navigator 0.2.0-SNAPSHOT USER appconfig.txt

Starting the Navigator Integration App
--------------------------------------

To start the MetadataFlow::

  > start flow metaApp.MetadataFlow

You should now be able to view CDAP Metadata in the Navigator UI. Note that all CDAP Entities use ``SDK`` as
the SourceType and use ``CDAP`` as the namespace (this can be changed). Since Navigator SDK doesn't allow adding
new EntityTypes, we have used this mapping:

+-------------------+-----------------------+
| CDAP EntityType   | Navigator EntityType  |
+===================+=======================+
| Application       | File                  |
+-------------------+-----------------------+
| Artifact          | File                  |
+-------------------+-----------------------+
| Dataset           | Dataset               |
+-------------------+-----------------------+
| Program           | Operation             |
+-------------------+-----------------------+
| Stream            | Dataset               |
+-------------------+-----------------------+
| StreamView        | Table                 |
+-------------------+-----------------------+

Mailing Lists
-------------
CDAP User Group and Development Discussions:

- `cdap-user@googlegroups.com <https://groups.google.com/d/forum/cdap-user>`__

The *cdap-user* mailing list is primarily for users using the product to develop
applications or building plugins for appplications. You can expect questions from
users, release announcements, and any other discussions that we think will be helpful
to the users.

IRC Channel
-----------
CDAP IRC Channel: #cdap on irc.freenode.net


License and Trademarks
======================

Copyright © 2016 Cask Data, Inc.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except
in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the
License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
either express or implied. See the License for the specific language governing permissions
and limitations under the License.

Cask is a trademark of Cask Data, Inc. All rights reserved.

Apache, Apache HBase, and HBase are trademarks of The Apache Software Foundation. Used with
permission. No endorsement by The Apache Software Foundation is implied by the use of these marks.

Cloudera Navigator is a trademark of Cloudera.
