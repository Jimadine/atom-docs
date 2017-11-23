.. _installation-upgrading:

=========
Upgrading
=========

This guide explains how to upgrade from an earlier AtoM release (including
ICA-AtoM versions 1.1 to 1.3.1 or newer) to |version|.

.. IMPORTANT::

   If you are on an earlier release of ICA-AtoM (older than 1.1), please
   upgrade to the latest ICA-AtoM release before following these instructions.
   Please see `Upgrading from ICA-AtoM 1.0.x <https://wiki.accesstomemory.org/Community/Community_resources/Documentation/Upgrading_from_ICA-AtoM_1.0.x>`_ on the AtoM wiki.

While we have tried to make this document usable by readers with a broad range
of technical knowledge, it may be too complex if you have no previous
experience with installing web applications or using the Linux command line.

Additionally, consider disabling your web site during the upgrade. Redirect
your users to a "Down for maintenance" page temporary using your web server
redirection capabilities, or you can put your site in read-only mode while
performing the upgrade.

.. _upgrading-requirements:

Check minimum requirements
==========================

Please refer to the :ref:`Minimum requirements <installation-requirements>`
page to make sure that your system meets all the requirements. This is
especially important if you are going to be upgrading from an older version
of ICA-AtoM (1.1 - 1.3.1 or later), as there will be some new dependencies
that we will install as part of the upgrade process.

.. _upgrading-release-notes:

Read the release notes
======================

This is the opportunity to find out what has been changed in the new release,
and if there are new features, enhancements and bug fixes that may be of
interest to you and your organization.

As a major release with hundreds of issue tickets closed, made public during a
time when AtoM is in the process of rewriting its documentation and migrating
wiki content, the AtoM 2.0.0 release notes were made in a more general form in
our User forum,
`here <https://groups.google.com/d/msg/ica-atom-users/_zgOnNxM1mE/ODGTv_Bxox4J>`__.

In the future, we will update these instructions with a consistent place on
our new wiki (coming soon), where you can always find release notes for the
latest version of AtoM, as well as those from previous releases.

.. _upgrading-install-atom:

Install the latest version of AtoM
==================================

Follow the instructions available in our documentation on :ref:`installation`
- our most comprehensive installation notes are included in the
:ref:`Linux <installation-linux>` section, with additional information for
different operating systems and server configurations.

.. IMPORTANT::

   Remember to create a **new** database for this installation. When you run
   the web installer, it will erase your previous data if you are using the
   same database!

.. _upgrading-copy-data:

Copy your old data
==================

At this point, you should have an |version| functional installation using a
fresh database. Now we are going to copy the contents of the old uploads and 
downloads directories, as well as the database:

1. `rsync <https://rsync.samba.org/>`__ is a robust directory sync solution
   that we can use to copy the contents of your old uploads directory to the
   new one, even when both directories are in the same machine. Using the
   command-line, enter the following command:

.. code-block:: bash

   $ rsync -av /var/www/icaatom_old/uploads/ /usr/share/nginx/atom/uploads/

Where ``icaatom_old`` is the name of your old installation. The path included
for the new installation in this example (``/usr/share/nginx/atom``) is the path
we recommend in our installation documentation.

Alternatively, you can just use `cp <https://en.wikipedia.org/wiki/Cp_%28Unix%29>`__:

.. code-block:: bash

   $ cp -r /var/www/icaatom_old/uploads/ /usr/share/nginx/atom/uploads/

We're going to want to do the same with the downloads directory as well - this is 
where :ref:`reports <reports-printing>`, :ref:`cached xml <cache-xml-setting>`, 
and downloads created by the job scheduler (such as 
:ref:`clipboard exports <csv-export-clipboard>`) are kept. 

.. code-block:: bash

   $ rsync -av /var/www/icaatom_old/downloads/ /usr/share/nginx/atom/downloads/

.. NOTE:: 
   
   You may choose to delete the contents of the ``jobs`` subdirectory after 
   copying it over - this subdirectory in the downloads folder generally contains 
   zip files of previous exports. As such, it is temporary data and does not need 
   to be kept. We recommend leaving the ``jobs`` subdirectory itself in place, 
   for future exports. 

   If you want to delete the contents of this directory, you can use the 
   following command: 

   .. code-block:: bash

      rm -f /usr/share/nginx/atom/downloads/jobs/*

2. Dump the contents of your old database to a temporary file:

.. code-block:: bash

   $ mysqldump -u username -p old_database > /tmp/database.sql

3. Drop and re-create the new AtoM database to remove any unnecessary tables and
   columns.

.. code-block:: bash

   $ mysql -u username -p -e 'drop database new_database; create database
   new_database character set utf8 collate utf8_unicode_ci;'

4. Now, load the contents into the new database:

.. code-block:: bash

   $ mysql -u username -p new_database < /tmp/database.sql

.. _upgrading-run-upgrade-task:

Run the upgrade task
====================

This is perhaps the most critical step in the upgrade process. If you
encounter any errors, please consult our 
`User Forum <https://groups.google.com/forum/#!forum/ica-atom-users>`__, or if 
you don't find a solution, feel free to post a question there yourself. We will 
also be trying to add to our `FAQ <https://wiki.accesstomemory.org/AtoM-FAQ>`__ 
as we receive feedback, to help users troubleshoot any upgrading issues 
encountered.

First, change the current directory:

.. code-block:: bash

   $ cd /usr/share/nginx/atom

Now, run the upgrade-sql task:

.. code-block:: bash

   $ php symfony tools:upgrade-sql

.. _upgrading-migrate-translations:

Migrate translations
====================

.. WARNING::

   At this time, we are troubleshooting challenges in translation migration
   process from older releases to |version|. Please see issue
   `#5505 <https://projects.artefactual.com/issues/5505>`__ for progress - we
   will update this documentation with instructions when the tranlsation
   migration process has been optimized and tested. Thank you in advance for
   your patience.

.. _upgrading-regen-digital-objects:

Regenerate the digital object reference and thumbnail images (optional)
=======================================================================

If you are upgrading from version 1.3.1 or earlier, you may want to regenerate
the :term:`digital object` :term:`reference <reference display copy>` and
:term:`thumbnail` images. The thumbnail size was smaller in 1.x, so those
images will often appear fuzzy in the redesigned digital object browse. A
directory naming convention has also been added to make the location of the
:term:`master digital object` more secure.

First, make sure you have not changed the directory (``/usr/share/nginx/atom``).

Now, run the regen-derivatives task:

.. code-block:: bash

   php symfony digitalobject:regen-derivatives

For more information on this task and its available options, see: 
:ref:`cli-regenerate-derivatives`.

.. _upgrading-rebuild-index-cc:

Rebuild search index and clear cache
====================================

To make all these changes take effect, you will need to re-index the files
you've imported into your database, and clear the application cache.

First, rebuild the search index:

.. code-block:: bash

   php symfony search:populate

For more information and options on this task, see: 
:ref:`maintenance-populate-search-index`.

Then, clear your `cache <http://symfony.com/legacy/doc/book/1_0/en/12-Caching>`__
to remove any out-of-date data from the application:

.. code-block:: bash

   $ php symfony cc

See :ref:`maintenance-clear-cache` for more detailed instructions.

.. _upgrading-use-software:

Set site base URL
=================

One final step is to set your site's base URL. This URL is used in XML exports
to formulate absolute URLs referring to resources.

To set the site base URL:

.. |gears| image:: ../../images/gears.png
   :height: 18
   :width: 18

1. Click on the |gears| :ref:`Admin <main-menu-admin>` menu in the :term:`main
   menu` located in the :term:`header bar` and select Settings.

2. Click on or scroll down to Site information. Enter your site's base URL
   into the site base URL field. If your domain is "townarchives.org", for
   example, your base URL would normally be "http://townarchives.org".

.. SEEALSO::

   * :ref:`Site information <site-information>`

.. _upgrading-custom-themes:

Upgrading with a custom theme plugin
====================================

If you have developed a custom theme plugin for your application (for more
information, see :ref:`customization-custom-theme`), you may need to perform
an additional step following an upgrade to ensure that all pages are styled
correctly.

Specifically, :ref:`job-details` may not appear properly styled in a custom
theme without an additional step. To ensure your Jobs pages properly inherit
the base Dominion theming, you will need to add a call to import the
``jobs.less`` CSS file to your theme plugin's ``main.less`` file. If you have
followed our recommendations for creating a theme plugin, then you should find
the ``main.less`` file for your plugin in
``plugins/yourThemePluginName/css/main.less``. Here is an example of where you
need to add a line in the ArchivesCanada theme plugin:

* https://github.com/artefactual/atom/blob/HEAD/plugins/arArchivesCanadaPlugin/css/main.less#L78

The line you will need to add is to import the base Jobs CSS, like so: 

.. code-block:: bash

   @import "../../arDominionPlugin/css/less/jobs.less" 

After adding the line, you should rebuild the CSS for the plugin, using the 
``make`` command. Here is an example of rebuilding the CSS for the ArchivesCanada 
theme - you can swap in the name of your plugin: 

.. code-block:: bash

   make -C plugins/arArchivesCanadaPlugin

You will also want to clear the application cache, and restart PHP-FPM. 

To clear the application cache: 

.. code-block:: bash

   php symfony cc

For more information, see: :ref:`maintenance-clear-cache`. 

To restart PHP-FPM on Ubuntu 14.04: 

.. code-block:: bash

   sudo service php5-fpm restart

To restart PHP-FPM on Ubuntu 16.04: 

.. code-block:: bash

   sudo systemctl restart php7.0-fpm

.. TIP::

   If you are still not seeing your changes take effect, remember to clear your
   web browser's cache as well! 

Start using the software!
=========================

Congratulations! If you are reading this, it means that you have upgraded your
data successfully. Now please check that everything is working fine.

.. IMPORTANT::

   Before you put your site in production again, please take a look at your
   data and check that everything looks good and the data has imported
   correctly. We will continue to refine this documentation over time to make
   the upgrade process as smooth as possible, but we still think it is always
   important to double-check your work. Let us know if you encounter any
   problems!


:ref:`Back to top <installation-upgrading>`
