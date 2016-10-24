.. _cacti:

Appendix B: Cacti Monitoring
****************************

This appendix will explain how to set up monitoring of multiple instances of STON using `Cacti <http://www.cacti.net/>`__'s Graph Tree. The following two prerequisites are required.

-  A server with Cacti installed
-  SNMP activation (see :ref:`snmp`)


.. toctree::
   :maxdepth: 2


.. _cacti_template:

Adding Templates
====================================

Using the Host Template provided by STON, the monitoring environment can easily be set up (`download <http://webhard.winesoft.co.kr/ston/monitoring/cacti/ston_host_template.xml>`_).

.. figure:: img/cacti01.png
   :align: center
      
   Select [Import Templates].
      
.. figure:: img/cacti02.png
   :align: center
      
   Import cacti_host_template_ston.xml.
      

.. _cacti_device_add:

Device Registration
====================================

Register STON as a Cacti device.

.. figure:: img/cacti03.png
   :align: center
      
   Select [Devices].
      
.. figure:: img/cacti04.png
   :align: center
      
   Click the [Add] button in the [Devices] menu.
      
.. figure:: img/cacti05.png
   :align: center
      
   Fill in the device options.


1. Input the name used for STON.
#. Input STON's IP address.
#. Select "STON".
#. Select "Public".
#. Input the default port of 161.


Click the "Create" button to engage the device.

.. figure:: img/cacti06.png
   :align: center
      
   Engaged successfully.
      
.. figure:: img/cacti07.png
   :align: center
      
   Engagement error.
      
.. note::

   If the SNMP engagement fails:
   
   -  Check STON to make sure SNMP is enabled.
   -  Check to see if the SNMP port number matches STON's SNMP port number.
      

If the device engagement succeeds, you can use 18 different types of graph provided by the STON template.

.. figure:: img/cacti08.png
   :align: center
      
   Click "Create Graphs for this Host".

.. figure:: img/cacti09.png
   :align: center
      
   There are 18 types of graphs provided.
      
Click the [Create] button and check the graphs that were created.

.. figure:: img/cacti10.png
   :align: center
      
   The graphs have been created.

      
.. _cacti_graph_tree:

Graph Tree Creation
====================================

Create Graph Trees.

.. figure:: img/cacti11.png
   :align: center
      
   Select [Graph Trees].
      
.. figure:: img/cacti12.png
   :align: center
      
   Click the [Add] button on the right side.
      
.. figure:: img/cacti13.png
   :align: center
      
   Create Graph Trees.
      

You can add STON to the Graph Tree.

.. figure:: img/cacti14.png
   :align: center
      
   Click the [Add] button in the [Tree Items] menu.

.. figure:: img/cacti15.png
   :align: center
      
   Select the [Tree Items] options.


1. Select "Host".
#. Select the devices you will add.
#. Select "Graph Template".
  
   
.. _cacti_graph_confirm:

Graph Check
====================================

Select [Graphs] in the upper left side to check if the graphs are displaying correctly.

.. figure:: img/cacti16.png
   :align: center
   
   Check regularly that the graphs are showing properly.