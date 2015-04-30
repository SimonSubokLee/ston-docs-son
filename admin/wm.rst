.. _wm:

Chapter 18. WM (Web Management)
******************

This chapter introduces Web Management (WM), which is a Web management tool based on API. 
Using WM you can intuitively configure the service and organize clusters in order to manage a large number of STON.

WM is installed in the /usr/local/ston/wm directory when STON is installed. 
WM is implemented with Apache 2.2.24 + PHP 5.3.24. 
Since WM uses Apache, by editing the /usr/local/ston/wm/conf/httpd.conf file, you can change the configuration (e.g. HTTPS) as desired.
WM and STON are not strongly connected. 
WM configures the STON operation using STON configuration files and API, as shown in the figure below.

.. figure:: img/wm_compose.jpg
   :align: center
   
   WM uses STON configuration files and API.
   
Better management methods might be possible with a similar method.

.. toctree::
   :maxdepth: 2



Connection
====================================

WM uses the 8500 port as a default. If the IP of the installed STON application is set to 192.168.0.100, then the WM access address will be http://192.168.0.100:8500. 
As previously mentioned, customization for each customer is available by editing the httpd.conf file. 

.. figure:: img/wm_login.jpg
   :align: center
   
   WM Start Page
   

Account
====================================

The default account is set to [ID: **admin** , Password: **ston** ]. 
The dashboard main page that shows the general status of STON will be displayed after a successful login.
The figure below shows the WM Dashboard.

.. figure:: img/wm_main.jpg
   :align: center
   
   WM Dashboard
   

.. _wm-update:

Update to the Latest Version
====================================

When a new version is released, the following "Update is Available" message will be displayed.

.. figure:: img/wm_update_info.png
   :align: center
   
   New Update is Available.
   
If you click the message, you will be brought to the page where you can update to the latest version. 
Depending on the current service status, an update safety status will be displayed.

.. figure:: img/wm_update_page_alert.png
   :align: center
   
   WM Update could be hazardous.
   
When the update is completed, all services will automatically restart.



Menu Composition
====================================

Drop down menus can be expanded/shrunk with mouse clicks. 

.. figure:: img/wm_menu.jpg
   :align: center
   
   WM Menu
   
1.  **Global Setting**

    All functions except the virtual host default setting can be configured in global settings (server.xml).
    
#.  **Virtual Host Management**
    
    You can add/suspend/delete virtual hosts and subscribe to all virtual host statuses that are in service.
    
#.  **Cluster**

    You can compose/manage/destruct clusters. All services in a cluster can be viewed by servers and services.
    
#.  **Content Control**

    You can control the content in service(e.g. Purge).
    
#.  **Server Status**

    You can monitor global resources, such as the system status. All graphs use global resource graphs.
    
#.  **Service Status**
    
    You can monitor the service status of the virtual host. All graphs use virtual host graphs.
    
#.  **File System**

    STON can be mounted on Linux VFS.
    
    
Global Settings
====================================

All functions except the virtual host default setting can be configured in global settings (server.xml). 

.. figure:: img/wm_conf_global1.png
   :align: center
   
   WM Global Settings - General
   


Managing Virtual Hosts
====================================

This section explains how to configure the details of all virtual hosts in service and enables you to add new virtual hosts. 
Virtual hosts inherit default configurations (VHostDefault), unless they are modified/changed by the administrator. 
This concept is identical to that of the inheritance in the object-oriented method, in which the service virtual host can reconfigure (Overriding) almost all items.


New
---------------------

This creates a new virtual host for the service. 
If a cluster is configured, then multiple virtual hosts can be created in every server at the same time. 
All virtual hosts inherit the default virtual host (VHostDefault); 
therefore, setting the origin server address and the virtual host name will allow the virtual host to be deployed for the service. 
There are eight different sub options, and they can be expanded to show more detailed settings by clicking the **Unfold** button. 

.. figure:: img/wm_vhost_new1.png
   :align: center
   
   WM Virtual host management - New
   
   
List
---------------------

You can monitor all virtual host statuses in service. 
In addition, you can start/stop each virtual host. 
If a cluster is configured, then you can control all of the virtual hosts in the entire server at the same time. 
You can also select the default virtual host.

.. figure:: img/wm_vhost_list.png
   :align: center
   
   WM Virtual Host Management - List
   
   
Detail Configuration
---------------------

This section explains how to configure the default virtual host (VHostDefault) and the separate virtual host. 
In the combo box in the top left corner, you can select a virtual host. 
**"Default Virtual Host"** is the default configuration that all virtual hosts inherit. 
Therefore, any configurations that weren't overridden will be affected by changes to the "Default Virtual Host".

.. figure:: img/wm_vhost_conf1.png
   :align: center
   
   WM Virtual Host Configuration - Top Menu
   
A dozen sub menus, like the above figure, are provided.
The selected sub menu is emphasized with a red background color. 
When you click on each menu, a detailed configuration page will load, as shown in the figure below. 
All configurations are reflected after "Apply" or "Apply to All Clusters" is clicked.

.. figure:: img/wm_vhost_conf_sub1.png
   :align: center
   
   WM Virtual Host Configuration - Origin Server
   
Almost all items configured in this section can be overridden, so you should have a thorough understanding for the each configuration. 
For example, if the TTL value of the default virtual host is set to 60, all virtual hosts inherit this value. 
However, if the TTL value is specifically redefined, then the corresponding virtual host will use the overridden TTL value.

.. figure:: img/wm_vhost_conf_sub_ttl.png
   :align: center
   
Three different possibilities are explained below:

-  **Override with another value**

   In the above illustration, when the default TTL is 60, User A will be served with 180 if the TTL value is overridden with 180. 
   The default virtual host is not affected by the overridden configuration.
   
-  **Override with the same value** 

   If you override the TTL with the default value, it is considered as an override and User B will be served with 60 TTL. 
   If the TTL of the default virtual host is changed to 30, the overridden value will not be affected and the User B will be keep servicing with 60 TTL.
   
-  **Do not override**

   User C will inherit the TTL of the default virtual host, and if the default value is updated to 30, the User C will be served with the changed default TTL of 30.

In WM, colors are used to identify override states. 
A white background identiifies inheritance of the default virtual host configuration. 
An apricot background identifies an overridden configuration. 
All override configurations have an X button on the right. 
If you click on this button, all overridden configurations will be removed.
   
   

Cluster
====================================

Multiple STONs can be merged into one cluster for integrated management/operation. 
All STONs are configured to have equal authority, so no matter which STON you log into within the cluster, you can manage the entire cluster.


Composition
---------------------

You can create a cluster or add another server to an existing cluster. 
In order to create a cluster, an authentication procedure for the WM account is required. 
If WM is configured with identical account information (ID and password), the authentication procedure will be skipped.

.. figure:: img/wm_cluseter1.png
   :align: center
   
   Creating a new cluster
   
   
.. figure:: img/wm_cluseter2.png
   :align: center
   
   Cluster list
   
When a cluster is composed, you can manage virtual hosts by performing a batch of configurations with "Apply to All Clusters". 
In addition, you can duplicate and apply configurations from one server to another server in the cluster. 
If a specific server needs to participate in another cluster, the corresponding server has to leave the cluster and recompose.


전용 포트분리
---------------------

최초 설치시 WM과 클러스터 포트가 같은 포트를 사용한다. 
이 방식은 WM계정만으로 클러스터링 구성이 가능하다는 장점이 있지만 접근 IP를 제한하는 환경에서는 문제가 될 수 있다.

* 보안상의 이유로 WM을 일부 IP에서만 접근되도록 제한한다.
* 클러스터링을 위해서는 모든 서버들이 서로 다른 서버들의 IP를 허가해주어야 한다.
* (CDN 처럼) 서버 수가 매우 많거나 서버들의 IP가 유동적인 경우 IP목록을 작성할 수 없다.

클러스터 포트를 분리하여 이 문제를 해결할 수 있다. 
서버들끼리 신원은 WM 계정이 아닌 라이선스를 통해 확인된다. 
같은 라이선스를 가진 서버들끼리만 클러스터가 가능해져 보안성이 높아진다.


**1. [Apache 서버] httpd.conf 멀티 Port설정**

(기본 설치라면) /usr/local/ston/wm/conf/httpd.conf 파일을 열어 다음과 같이 포트를 추가한다.

.. figure:: img/wm_cluster_multiport.png
   :align: center

저장 후 반영을 위해 Apache서버를 재시작한다.


**2. [WM] 클러스터 구성**

정상적으로 멀티포트 구성이 되었다면 다음과 같이 "클러스터 포트 분리" 버튼이 생성된다.

.. figure:: img/wm_cluster_multiport1.png
   :align: center

버튼을 클릭한다.


**3. [WM] 클러스터 포트선택**

분리할 수 있는 포트 목록을 보여준다. 포트를 선택해 구성한다.

.. figure:: img/wm_cluster_multiport2.png
   :align: center

클러스터링에 포함되는 모든 서버는 반드시 같은 포트를 사용해야 한다.


Server Status
---------------------

Statuses and service reports of cluster affiliated STON servers can be checked. 
If you click on the each item that composes a server list, detailed information can be viewed.

.. figure:: img/wm_cluseter3_2.png
   :align: center
   
   Each server status
   
   
Virtual Host Status
---------------------

MRTG of all virtual hosts in the cluster are put together in one screen. 
You can also start/stop all virtual hosts in the cluster at the same time. 
If you click on the each item that composes a server list, detailed information can be viewed.

.. figure:: img/wm_cluseter4.png
   :align: center
   
   Status of each virtual host
   
   
Content Control
====================================

You can browse/control content that is currently being serviced or perform cleanup. 
If the cluster is configured, you can browse or control content in all STON servers at the same time.

.. figure:: img/wm_ctrl2.png
   :align: center
   
   Caching status check
   
   
.. figure:: img/wm_ctrl3.png
   :align: center
   
   API call
   
      
Server Status(Hardware)
====================================

(이 내용은 바로 위의 Contents Control과 완전히 동일합니까??)
서비스 중인 컨텐츠를 열람/제어하거나 클린업을 수행할 수 있다. 
클러스터 구성이 되어있다면 모든 STON의 컨텐츠를 동시에 열람하거나 제어할 수 있다.

.. figure:: img/wm_gstat1.png
   :align: center
   
   Hardware information
   
   
Service Status
====================================

Monitors the service status of each virtual host.

.. figure:: img/wm_vstat3.png
   :align: center
   
   Service status of virtual host
   
