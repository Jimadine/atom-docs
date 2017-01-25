.. _maintenance-tuning:

========================
Tuning server parameters
========================

If you are looking for methods to improve the performance of AtoM and your
deployment, you are now in the right place!

First of all, most common performance issues could be solved by redesigning the
code, and that's something that we are working on constantly. But there's no
doubt that some configuration tweaks in your servers can give you better
results.

This page describes the most common performance tweaks used in AtoM.


PHP opcode cache
================

PHP is an interpreted language. Each time a client requests a PHP page, the
server will read the source code and compile it before it's executed.
`APC <http://php.net/manual/en/book.apc.php>`__ is one of the solutions
available that caches the compiled output of each PHP script in order to save
CPU cycles in subsequent requests.

Deploying APC is really easy:

.. code-block:: bash

   sudo apt-get install php-apc

We consider this component so important that it's listed as a mandatory
extension under :ref:`installation-requirements`. Really, don't ignore this:
we consider compiling the same source code again and again in a per-request
basis to be a huge waste of time and computing resources.

.. _maintenance-tuning-apc-stat:

In addition, there is one particular directive in APC that you should tweak to
get the best performance: ``apc.stat``, enabled by default, is a directive
that forces APC to check the PHP scripts on each request to determine if they
have been modified. Modifying PHP scripts may happen frequently in development
environments, but in production you don't really need this. When you are
upgrading AtoM, just restart the PHP pool and the APC cache will be flushed.

.. note::

   APC has not been ported to PHP 5.5. Instead, Zend has contributed their `Zend
   Optimizer+ opcode cache (OPcache) <http://php.net/manual/en/book.opcache.php>`__.
   However, user-data cache is not something that OPcache implements, so if you
   are planning to use PHP 5.5 please look at APCu as a valid replacement for
   user-land caching, which emulates its functionality while providing the same
   API.


Other components that can be tweaked
====================================

This document is work in progress. We are planning to describe more tweaking
options in at least the following components:

* MySQL
* Nginx
* PHP-FPM
* Linux
* Caching proxy servers (HTTP accelerators)
* etc...
