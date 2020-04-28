.. _tutorials_addstation:

*****************
Add a new station
*****************

You will ...

* Add a new station to your SeisComP system for archiving
* Configure it for processing too

Pre-requisites for this tutorial:

* :ref:`tutorials_postinstall`
* :ref:`tutorials_geofon_waveforms`
* :ref:`tutorials_archiving`
* An understanding of :ref:`concepts_inventory`

Afterwards/Results/Outcomes:

* For the new station data are acquired and archived in real time
* The new station is used for automatic real-time data processing

Time range estimate:

* 40 minutes

----------

Before you start
================

Try to answer the questions:

* where will you get data?
* where will you get inventory?
* which data will you share?
* how long will you archive, and what streams?

For this example, we'll add station GF99 from the 6C network in Myanmar.

Common inventory sources
========================

Meta data can be obtained from many different sources or created from scratch.

Fetch inventories
-----------------

* Other SeisComP systems. Use :ref:`scxmldump` to fetch inventories.
* `Eida nodes`_. Use web interfaces such web browsers or wget to fetch an inventory.
* Data centers providing `FDSNWS`_. Use web interfaces such web browsers or wget to fetch an inventory.


Create and share inventories
----------------------------

* gempa's `SMP`_ for creating inventory from scratch and community sharing.
  Create inventories for new or old networks and stations from permanent or temporary
  deployments.
  SMP provides inventories in :term:`SCML` format in multiple versions which can be used without modification.

Configuring for acquisition and archiving
=========================================

Create top-level key file

- source, channels to be acquired, location code used, if any

- consider how long to archive

If you don't want to process a station, you can leave out the "global"
and "scautopick" lines in the top level key file, by editing the file,
or removing the bindings for these modules using :program:`scconfig`.

.. note ::

   **(Advanced)**
   You can restrict who has access to a station's data (from your server)
   in your Seedlink bindings.
   This sets the :confval:`access` variable in your :file:`seedlink.ini` file.
   This can be done for all stations, or per station.
   The documentation for :ref:`seedlink` gives details.

Configuring for processing
==========================

You will need inventory for the new station.
How to obtain this will vary, but for this example, suppose that
we have it in a single file, :file:`inventory_GF99.xml`.

Place this in :file:`~/seiscomp/etc/inventory`.

OR import, scinv, whatever. See the inventory tutorial.


Then:

.. code-block:: sh

   $ seiscomp update-config
   $ seiscomp restart


Checking the station is there and functioning
=============================================

* If :program:`seedlink` is configured correctly, the station's streams
  appears in output from :program:`slinktool`::

    $ slinktool -Q : | grep GF99
    6C GF99     HHE D 2019/12/06 04:15:08.6800  -  2019/12/06 09:30:17.7600
    6C GF99     HHN D 2019/12/06 04:15:10.9200  -  2019/12/06 09:30:17.3700
    6C GF99     HHZ D 2019/12/06 04:15:13.1000  -  2019/12/06 09:30:16.8800

  This shows three streams being acquired from station 'GF99'.
  The second time shown is the time of the most recent data for each stream.

* If :program:`slarchive` is configured correctly, waveform data for the
  station appears in :program:`slarchive`'s SDS archive directory:

   .. code-block:: sh

      $ ls -l seiscomp/var/lib/archive/2019/6C/GF99/
      total 12
      drwxr-xr-x 2 user user 4096 Dec  6 06:30 HHE.D
      drwxr-xr-x 2 user user 4096 Dec  6 06:30 HHN.D
      drwxr-xr-x 2 user user 4096 Dec  6 06:30 HHZ.D

      $ ls -l seiscomp/var/lib/archive/2019/6C/GF99/HHZ.D/
      total 12728
      -rw-r--r-- 1 user user 5492224 Dec  6 06:34 6C.GF99..HHZ.D.2019.339
      -rw-r--r-- 1 user user 7531008 Dec  6 16:01 6C.GF99..HHZ.D.2019.340

If you have configured the station for processing, then:

* On restarting :program:`scautopick`, the station appears in the
  :file:`scautopick.log` log
  file in :file:`~/.seiscomp/log`::

    2019/12/05 19:01:00 [info/Autopick] Adding detection channel 6C.GF99..HHZ

  After some time, a nearby event will occur and the station should then be picked.
  This should appear in the latest :file:`autoloc-picklog` file in
  :file:`~/.seiscomp/log`:

  .. code-block:: sh

     $ grep "GF99" .seiscomp/log/autoloc-picklog.2019-12-06
     2019-12-06 07:47:21.9 6C GF99   HHZ __  366.3 511450.094  1.1 A 20191206.074721.97-6C.GF99..HHZ

* The station should now appear in the GUIs.
  After restarting them,

  - The station should now show up in :program:`scmv`
    (as a new triangle at the expected location on the map,
    which is not black if the station is active).

  - In :program:`scrttv` a trace should be visible.

    [Problem: detecStream ??].

  - In :program:`scolv`, the new station is either already included
    in automatic locations, or can be added manually.


References
==========

.. target-notes::

.. _`EIDA nodes` : http://orfeus-eu.org/stationbook/nodes/
.. _`FDSNWS` : https://www.fdsn.org/webservices/datacenters/
.. _`SMP` : https://smp.gempa.de
