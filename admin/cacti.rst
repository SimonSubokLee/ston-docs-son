.. _cacti:

Appendix B: Cacti Monitoring
*************************

This appendix is about monitoring and graphing STON with Graph Tree from `Cacti <http://www.cacti.net/>`_ .
The following prerequisites are required.

-  A server with Cacti installed
-  SNMP activation (Please refer to :ref:`snmp`)


.. toctree::
   :maxdepth: 2


.. _cacti_template:

Adding templates
====================================

The Host Template provided along with STON may help configuring monitoring. 
Please `download at <http://webhard.winesoft.co.kr/ston/monitoring/cacti/ston_host_template.xml>`_ )

.. figure:: img/cacti01.png
   :align: center
      
   Select Import Templates menu.
      
.. figure:: img/cacti02.png
   :align: center
      
   Import cacti_host_template_ston.xml
      

.. _cacti_device_add:

Device Registration
====================================

Register STON as a Cacti device.

.. figure:: img/cacti03.png
   :align: center
      
   Select [Devices] menu.
      
.. figure:: img/cacti04.png
   :align: center
      
   Click [Add] button from [Devices] menu.
      
.. figure:: img/cacti05.png
   :align: center
      
   Fill in the device options.


-  ① Input the STON's name.
-  ② Input the STON's IP.
-  ③ Select ”STON”.
-  ④ Select “Public”.
-  ⑤ Input the default port 161.


Click the "Create" button and engage the deivce.

.. figure:: img/cacti06.png
   :align: center
      
   Engaged successfully.
      
.. figure:: img/cacti07.png
   :align: center
      
   Enegement error.
      
.. note::

   If the SNMP engagement fails:
   
   -  Please check if SNMP from STON is enabled.
   -  Please make sure if the SNMP port matches that of STON.
      

The STON template provides 18 different graph types.

.. figure:: img/cacti08.png
   :align: center
      
   Click the "Create Graphs for this Host" link.

.. figure:: img/cacti09.png
   :align: center
      

Click the [Create] button.

.. figure:: img/cacti10.png
   :align: center
      
   Graphs created now.

      
.. _cacti_graph_tree:

Graph Tree Creation
====================================

Create Graph Trees.

.. figure:: img/cacti11.png
   :align: center
      
   Click the [Graph Trees] tab.
      
.. figure:: img/cacti12.png
   :align: center
      
   Click the [Add] button.
      
.. figure:: img/cacti13.png
   :align: center
      
   Create Graph Trees.
      

Add STON to the Graph Tree.

.. figure:: img/cacti14.png
   :align: center
      
   Click the [Add] button from [Tree Items] menu.

.. figure:: img/cacti15.png
   :align: center
      
   Select [Tree Items] options.


-  ①	Select “Host”.
-  ②	Select “Devices” to add.
-  ③	Select “Graph Template”.   
  
   
.. _cacti_graph_confirm:

Graphs Generated
====================================

Click the [graphs] menu and make sure the graph displayed.

.. figure:: img/cacti16.png
   :align: center
   
   Check regularly for operation.
