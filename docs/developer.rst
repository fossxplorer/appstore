App Developer Guide
===================

Most of today's developers publish their source code on GitHub, BitBucket or on their own GitLab instance. These tools typically also provide a way to release new versions based on Git tags or by uploading custom archives.

Advanced users and developers typically prefer to download the app directly from these services whereas administrators or novice users look for app releases on the app store. This means that you have to take care of publishing two releases on two different platforms.

We want to avoid duplication and make it harder to ship broken releases by mistake, therefore we went for the following solution:

* Your app's source code is hosted on GitHub or a similar service

* You should use Git tags to create new releases on these services

* Archives are typically created automatically for you. If you require compilation or other transformations like minification, you should upload a pre-built archive to the appropriate releases page

This keeps your repository up to date and satisfies the needs of developers and advanced users.

App Release Workflow
--------------------

To publish an app release on the app store you simply send us a download link for the release archive using either :doc:`ncdev <ncdev>` or any tool that can access the :doc:`restapi` (even with curl). We then do the following:

* Your archive is downloaded from the given location. This ensures that your users don't hit dead links. If your archive is too big, we will abort the download.

* The archive is then extracted and the package structure is validated:

 * The archive most only contain one top level folder consisting of lower case ASCII characters and underscores
 * The archive must contain an **info.xml** file inside the **appinfo** directory which in turn is located in the top folder

* The app's metadata is then extracted from the **info.xml** file

* The info.xml is reformatted using XSLT to bring everything into the correct order (required for XSD 1.0) and unknown elements are dropped

* The cleaned up info.xml is then validated using an XML Schema (see :ref:`info-schema`)

* The release is then either created or updated. The downloaded archive will be deleted

.. _app-metadata:

App Metadata
------------

App metadata is currently only being read from the **appinfo/info.xml** file. Future releases might include further files like CHANGELOG.md and AUTHORS.md files.

info.xml
~~~~~~~~
A minimum valid **info.xml** would look like this:

.. code-block:: xml

    <?xml version="1.0"?>
    <info xmlns:xsi= "http://www.w3.org/2001/XMLSchema-instance"
          xsi:noNamespaceSchemaLocation="https://apps.nextcloud.com/schema/apps/info.xsd">
        <id>news</id>
        <name>News</name>
        <description>An RSS/Atom feed reader</description>
        <author>Bernhard Posselt</author>
        <category>multimedia</category>
        <version>8.8.2</version>
        <licence>agpl</licence>
        <dependencies>
            <owncloud min-version="9.0"/>
        </dependencies>
    </info>

A full blown example would look like this (needs to be utf-8 encoded):

.. code-block:: xml

    <?xml version="1.0"?>
    <info xmlns:xsi= "http://www.w3.org/2001/XMLSchema-instance"
          xsi:noNamespaceSchemaLocation="https://apps.nextcloud.com/schema/apps/info.xsd">
        <id>news</id>

        <!-- translation can be done via the lang attribute, defaults to English -->
        <name lang="de">Nachrichten</name>
        <name>News</name>

        <!-- if missing, the description is used -->
        <summary lang="en">An RSS/Atom feed reader</summary>

        <!-- description tag allows Markdown -->
        <description lang="en"># Description\nAn RSS/Atom feed reader</description>
        <description lang="de"><![CDATA[# Beschreibung\nEine Nachrichten App, welche mit [RSS/Atom](https://en.wikipedia.org/wiki/RSS) umgehen kann]]></description>

        <!-- semantic version, three digits separated by a dot -->
        <version>8.8.2</version>

        <!-- only agpl is an acceptable license -->
        <licence>agpl</licence>

        <author mail="mail@provider.com" homepage="http://example.com">Bernhard Posselt</author>
        <author>Alessandro Cosentino</author>
        <author>Jan-Christoph Borchardt</author>

        <!-- documentation -->
        <documentation>
            <user>https://github.com/owncloud/news/wiki#user-documentation</user>
            <admin>https://github.com/owncloud/news#readme</admin>
            <developer>https://github.com/owncloud/news/wiki#developer-documentation</developer>
        </documentation>

        <!-- multiple categories are also possible -->
        <!-- possible values: multimedia, tools, games, pim -->
        <category>multimedia</category>
        <category>tools</category>


        <website>https://github.com/owncloud/news</website>

        <!-- issue tracker -->
        <bugs>https://github.com/owncloud/news/issues</bugs>

        <!-- screenshots, can be multiple and will be displayed in order -->
        <!-- need to be served with https -->
        <screenshot>https://example.com/1.png</screenshot>
        <screenshot>https://example.com/2.jpg</screenshot>

        <!-- dependencies, all version attributes except for the ownCloud min-version are optional -->
        <dependencies>
            <php min-version="5.6" min-int-size="64"/>
            <!-- php extensions, uses the same names as composer -->
            <!-- supported databases and versions -->
            <database min-version="9.4">pgsql</database>
            <database>sqlite</database>
            <database min-version="5.5">mysql</database>

            <!-- command line tools -->
            <command>grep</command>
            <command>ls</command>

            <lib min-version="2.7.8">libxml</lib>
            <lib>curl</lib>
            <lib>SimpleXML</lib>
            <lib>iconv</lib>

            <!-- version numbers will be padded to three digits with 0 (min-version) and 2^64 (max-version) -->
            <owncloud min-version="9.0" max-version="9.1"/>
        </dependencies>

        <!-- further elements to test if parser ignores non defined fields -->
    </info>

The following character maximum lengths are enforced:

* All description Strings are (almost) of unlimited size
* All Url Strings have a maximum of 256 characters
* All other Strings have a maximum of 128 characters



.. _info-schema:

Schema Integration
------------------
We provide an XML schema for the info.xml file which is available under `https://apps.nextcloud.com/schema/apps/info.xsd <https://apps.nextcloud.com/schema/apps/info.xsd>`_ and can be used to validate your info.xml or provide autocompletion in your IDE.

You can validate your info.xml using `various online tools <http://www.utilities-online.info/xsdvalidation/>`_

Various IDEs automatically validate and auto complete XML elements and attributes if you add the schema in your info.xml like this:

.. code-block:: xml

    <?xml version="1.0"?>
    <info xmlns:xsi= "http://www.w3.org/2001/XMLSchema-instance"
          xsi:noNamespaceSchemaLocation="https://apps.nextcloud.com/schema/apps/info.xsd">

          <!-- content here -->

    </info>

Verification
------------
Since we don't host the package ourselves this implies that the download location must be trusted. The following mechanisms are in place to guarantee that the downloaded version has not been tampered with:

* You can submit a sha256sum hash in addition to the download link. The hash is validated on the user's server when he installs it. If you omit the hash, we generate it from the downloaded archive

* You can sign your code `using a certificate <https://docs.nextcloud.org/server/9/developer_manual/app/code_signing.html>`_

* You must supply an HTTPS download url for the archive
