.. _wm:

Chapter 18. WM (Web Management)
******************

This chapter introduces Web Management(WM).
WM is a Web management tool based on API. 
You can intuitively configure the service with WM as well as organizing clusters in order to manage large number of STON.

WM is installed in the /usr/local/ston/wm directory when STON is installed. 
WM is implemented with Apache 2.2.24 + PHP 5.3.24. 
Since WM is using Apache, by editing /usr/local/ston/wm/conf/httpd.conf file, you can change the configuration(eg. HTTPS) as desired.
WM and STON is not strongly connected. 
WM configures STON operation only with STON configuration file and API as below figure.

.. figure:: img/wm_compose.jpg
   :align: center
   
   WM uses STON configuration file and API.
   
There could be a better management method that exceeds WM.

.. toctree::
   :maxdepth: 2



Connection
====================================

WM uses 8500 port as a default. If the IP of installed STON is 192.168.0.100, then WM access address will be http://192.168.0.100:8500. 
Customization for each customer is available as aforementioned by editing httpd.conf file.

.. figure:: img/wm_login.jpg
   :align: center
   
   WM Start Page
   

Account
====================================

The default account is set to [ID: **admin** , password: **ston** ]. 
Dashboard main page that shows general status of STON will be displayed after the successful login.

.. figure:: img/wm_main.jpg
   :align: center
   
   WM Dashboard
   

.. _wm-update:

Update to the Latest Version
====================================

If the latest version is released, the following " Update is Available" message will be displayed.

.. figure:: img/wm_update_info.png
   :align: center
   
   New Update is Available.
   
If you click the message, you will be brought to the page where you can update to the latest version. 
Depends on the current service status, update safety status will be displayed.

.. figure:: img/wm_update_page_alert.png
   :align: center
   
   WM Update could be hazardous.
   
When update is completed, all services are automatically restarted.



Menu Composition
====================================

Drop down menus can be expanded/shrunk with mouse clicks. 

.. figure:: img/wm_menu.jpg
   :align: center
   
   WM Menu
   
1.  **Global Setting**

    All functions except the virtual host default setting can be configured in the global setting(server.xml).
    
#.  **Virtual Host Management**
    
    You can add/suspend/delete virtual hosts, and subscribe all virtual host statuses in service.
    
#.  **Cluster**

    You can compose/manage/destruct clusters, and all services in the cluster can be viewed by servers, and services.
    
#.  **Contents Control**

    You can have a control over the contents in service(eg. Purge).
    
#.  **Server Status**

    Monitors global resources such as system status. All graphs use global resource graphs.
    
#.  **Service Status**
    
    Monitors service status of virtual host. All graphs use virtual host graphs.
    
#.  **File System**

    STON can be mounted on Linux VFS.
    
    
Global Setting
====================================

All functions except the virtual host default setting can be configured in the global setting(server.xml). 

.. figure:: img/wm_conf_global1.png
   :align: center
   
   WM Global Setting - General
   


Managing Virtual Host
====================================

This configures details of all virtual hosts in service as well as adding a new virtual host. 
As long as all virtual hosts do not change configurations explicitly, they use default virtual host configurations(VHostDefault). 
This is an identical concept of the inheritance in object-oriented method. 
Service virtual host can reconfigure(Overriding) almost all items.


New
---------------------

This creates a new virtual host for service. 
If cluster is configured, then multiple virtual hosts can be created in every server at the same time. 
All virtual hosts inherits default virtual host(VHostDefault), 
therefore setting the origin server address and the virtual host name will allow the virtual host to be deployed for the service. 
There are total 8 different sub options, and they can be expanded to detail settings by clicking the **Unfold** button. 

.. figure:: img/wm_vhost_new1.png
   :align: center
   
   WM Virtual host management - New
   
   
List
---------------------

You can monitor all virtual host statuses in service. 
In addition, you can start/stop each virtual host. 
If cluster is configured, then you can control all virtual hosts in the entire server at the same time. 
You can also select the default virtual host.

.. figure:: img/wm_vhost_list.png
   :align: center
   
   WM Virtual host management - List
   
   
Detail Configuration
---------------------

This configures default virtual host(VHostDefault) and separate virtual host. 
With the combo box on the top left corner, you can select a virtual host. 
**"Default Virtual Host"** is the default configuration that all virtual hosts inherits. 
Therefore, an configuration that wasn't overrided will be affected by the change of the "Default Virtual Host".

.. figure:: img/wm_vhost_conf1.png
   :align: center
   
   WM Virtual Host Configuration - Top Menu
   
A dozen of sub menus are provided like the above figure, and currently selected sub menu is emphasized with the red background color. 
By clicking each menu, detail configuration page is loaded as below figure. 
All configurations are reflected after clicking "Apply" or "Apply to All Clusters".

.. figure:: img/wm_vhost_conf_sub1.png
   :align: center
   
   WM Virtual Host Configuration - Origin Server
   
Almost all items configured in this section can be overrided so you should have a thorough understanding for the each configuration. 
For example, if the TTL value of default virtual host is set to 60, all virtual hosts are inherited this value. 
However, if the TTL value is specifically redefined, then the corresponding virtual host will use overrided TTL value.

.. figure:: img/wm_vhost_conf_sub_ttl.png
   :align: center
   
There could be 3 different possibilities as belows.

-  **Override with another value**

   From the above illustration, when the default TTL is 60, the user A will be served with 180 if the TTL value is the overrided with 180. 
   The default virtual host is not affected with the overrided configuration.
   
-  **Override with the same value** 

   If you override the TTL with the default value, it is considered as an override and the user B will be served with 60 TTL. 
   If the TTL of default virtual host is changed to 30, overrided value will not be affected and the user B will be keep servicing with 60 TTL.
   
-  **Do not override**

   The user C will inherit the TTL of default virtual host, and if the default value is updated to 30, the user C will be served with changed default TTL of 30.

In the WM, colors are used to identify override states. 
White background identiifies inheritance of the default virtual host configuration. 
Apricot background identifies overrided configuration. 
All override configurations have an X button on the right. 
If you click on this button, all overrided configurations will be removed.
   
   

Cluster
====================================

Multiple STONs can be merged into one cluster for integrated management/opreation. 
All STONs are configured to have an equal authority, which STON you may log in within the cluster, you can manage the entire cluster.


Composition
---------------------

You can create a cluster or add another server in the existing cluster. 
In order to create a cluster, authentication procedure of the WM account is required. 
If a WM is configured with the identical account(ID and password), the authentication procedure will be skipped.

.. figure:: img/wm_cluseter1.png
   :align: center
   
   Creating a new cluster
   
   
.. figure:: img/wm_cluseter2.png
   :align: center
   
   Cluster list
   
When a cluster is composed, you can perform a batch configuration with "Apply to All Clusters" when managing virtual hosts. 
In addition, you can duplicate and apply configuration from one server to another server in the cluster. 
If a specific server needs to participate in another cluster, the corresponding server have to leave the cluster and recompose.


Server Status
---------------------

Statuses and service reports of cluster affiliated STON servers can be checked. 
If you click each item that consists server list, detailed information can be acquired.

.. figure:: img/wm_cluseter3_2.png
   :align: center
   
   Each server status
   
   
Virtual Host Status
---------------------

MRTG of all virtual hosts in the cluster are put together in one screen. 
You can also start/stop all virtual hosts in the cluster at the same time. 
If you click each item that consists server list, detailed information can be acquired.

.. figure:: img/wm_cluseter4.png
   :align: center
   
   Status of each virtual host
   
   
Contents Control
====================================

You can browse/control currently servicing contents or perform cleanup. 
If the cluster is configured, you can browse or control contents in all STON server at the same time.

.. figure:: img/wm_ctrl2.png
   :align: center
   
   Caching status check
   
   
.. figure:: img/wm_ctrl3.png
   :align: center
   
   API call
   
      
Server Status(위에도 "서버 상태"가 있습니다??)서버 상태
====================================

(이 내용은 Contents Control과 중복됩니다??)
서비스 중인 컨텐츠를 열람/제어하거나 클린업을 수행할 수 있다. 
클러스터 구성이 되어있다면 모든 STON의 컨텐츠를 동시에 열람하거나 제어할 수 있다.

.. figure:: img/wm_gstat1.png
   :align: center
   
   Hardware information
   
   
Service Status
====================================

Monitors service status of each virtual host.

.. figure:: img/wm_vstat3.png
   :align: center
   
   Service status of virtual host
   
