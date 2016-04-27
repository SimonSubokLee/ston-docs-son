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
      
   Devices 항목을 작성한다.


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
      

Device 연동에 성공하면 STON Template 에서 제공하는 18가지 항목의 그래프를 사용 할 수 있다.
The STON template provides 18 different 

.. figure:: img/cacti08.png
   :align: center
      
   "Create Graphs for this Host" 링크를 클릭한다.

.. figure:: img/cacti09.png
   :align: center
      
   18가지 그래프가 제공된다.
      
[Create] 버튼을 클릭하여 생성된 그래프를 확인한다.

.. figure:: img/cacti10.png
   :align: center
      
   그래프가 생성되었다.

      
.. _cacti_graph_tree:

Graph Tree 생성
====================================

Graph Tree를 생성한다.

.. figure:: img/cacti11.png
   :align: center
      
   [Graph Trees] 클릭한다.
      
.. figure:: img/cacti12.png
   :align: center
      
   우측 상단의 [Add]를 클릭한다.
      
.. figure:: img/cacti13.png
   :align: center
      
   Graph Tree 생성한다.
      

STON을 Graph Tree에 추가한다.

.. figure:: img/cacti14.png
   :align: center
      
   [Tree Items] 메뉴에서 [Add]를 클릭한다.

.. figure:: img/cacti15.png
   :align: center
      
   [Tree Items]항목을 작성 한다.


-  ①	“Host”를 선택한다.
-  ②	추가할 “Devices”로 선택한다.
-  ③	“Graph Template”로 선택 한다.   
  
   
.. _cacti_graph_confirm:

Graphs 확인
====================================

좌측 상단의 [graphs] 메뉴를 클릭하여 그래프가 정상적으로 나오는지 확인한다.

.. figure:: img/cacti16.png
   :align: center
   
   주기적으로 정상동작 여부를 확인한다.
