.. _getting-started:

Chapter 2. Getting Started
******************

This chapter explains how to set up an example virtual host from installation and system configuration.
All this proccess can be done with a simple text editor.

STON is developed to run on the standard Linux server.
Any hardware, OS or file system dependencies are excluded from developing stages.
It is very important to help customers to select appropriate equipments because configuring proper server based on the nature and scale of service is the beginning of service.

.. toctree::
   :maxdepth: 2

.. _getting-started-serverconf:

Server Components
====================================

When configuring a general server, CPU, memory and storage are major components to consider.
Especially for a service that requires a high performance like 10Gbps, each component should meet the requirement in order to reach the desired service performance.

-  **CPU**

   At least quad core processor is recommended. 
   Many-Core system increases the scalability of STON.
   Every additional processor core will boost processing power of the server.
   However, high processing power does not necessarily mean high traffic transmission rate.

   .. figure:: img/10g_cpu.jpg
      :align: center

      Multiple processing core will prove its effectiveness when more clients are accessing to the server.

   In terms of bandwidth use, transferring a 4KB file for 260 thousand times or one time transfer of one 1GB file takes identical bandwidth.
   The most critical criterion for selecting CPU is a processing power of simultaneous connetions.


-  **Memory**

   The STON recommends at least 4GB of system memory.
   Frequently accessed contents should be allocated on the memory in order to increase the performance.
   A server with lack of physical memory will inevitably accesses to the storage disk more frequently that increases burden of the disk.
   If the I/O load of the disk is significantly high regardless of the number of served contents, installing additional memory can reduce the I/O load.
   

-  **Disk**

   At least 3 disks including OS disk are recommended.
   Just like memory, more disk gives better performance because I/O load will be dispersed and more contents could be cached.
   
   .. figure:: img/02_disk.png
      :align: center
      
      OS and STON always have to be installed on separate disks.
   
   Usually the STON is installed on the OS disk.
   Log is also configured on the identical disk where OS is installed.
   Since log is recording real time service status, the disk is always suffered from write load.
   (OS와 STON을 별도의 디스크에 구성해야 하는 이유에 대한 부연설명인가요?)
   일반적으로 OS가 설치된 디스크에 STON을 설치한다. 
   로그 역시 같은 디스크에 구성하는 것이 일반적이다. 
   로그는 서비스 상황을 실시간으로 기록하기 때문에 항상 Write부하가 발생한다.
   
   STON utilizes disks as RAID 0.
   성능과 RAID의 상관여부(상관여부라는게 요구되는 성능에 따라 RAID를 구성할지 말지의 여부인가요??)는 고객 서비스 특성에 따라 달라진다.
   However, when file modification is not frequent and the size of content is much larger than that of physical memory, read speed can be effectively increased via RAID.


.. _getting-started-os:

OS Configuration
====================================

The standard installation is good enough.
The STON works great with standard 64bit Linux(Cent 6.2 or higher, Ubuntu 10.04 or higher), and it does not rely on any packages.


.. _getting-started-install:

Installation
====================================

1. First, download the latest version of STON. ::

      [root@localhost ~]# wget  http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      --2014-06-17 13:29:14--  http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      Resolving foobar.com... 192.168.0.14
      Connecting to foobar.com|192.168.0.14|:80... connected.
      HTTP request sent, awaiting response... 200 OK
      Length: 71340645 (68M) [application/x-gzip]
      Saving to: “ston.2.0.0.rhel.2.6.32.x64.tar.gz”
      
      100%[===============================================>] 71,340,645  42.9M/s   in 1.6s
      
      2014-06-17 13:29:15 (42.9 MB/s) - “ston.2.0.0.rhel.2.6.32.x64.tar.gz” saved [71340645/71340645]


2. Extract downloaded package. ::

		[root@localhost ~]# tar -zxf ston.2.0.0.rhel.2.6.32.x64.tar.gz

3. Run the installation script. ::

		[root@localhost ~]# ./ston.2.0.0.rhel.2.6.32.x64.sh

4. All installation processes are recorded in the install.log file. The log file helps tracking any possible issues during installation process. ::

      #DownloadURL: http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      #DownloadTime: 13 sec
      #Target: STON 2.0.0
      #Date: 2014.03.03 16:48:35
      Prepare for STON 2.0.0 install process
          Stopping STON...
              STON stopped
      [Copying files]
          `./fuse.conf' -> `/etc/fuse.conf'
          `./libfuse.so.2' -> `/usr/local/ston/libfuse.so.2'
          `./libtbbmalloc_proxy.so' -> `/usr/local/ston/libtbbmalloc_proxy.so'
          `./start-stop-daemon' -> `/usr/sbin/start-stop-daemon'
          `./libtbbmalloc_proxy.so.2' -> `/usr/local/ston/libtbbmalloc_proxy.so.2'
          `./libtbbmalloc.so' -> `/usr/local/ston/libtbbmalloc.so'
          `./libtbbmalloc.so.2' -> `/usr/local/ston/libtbbmalloc.so.2'
          `./libtbb.so' -> `/usr/local/ston/libtbb.so'
          `./libtbb.so.2' -> `/usr/local/ston/libtbb.so.2'
          `./stond' -> `/usr/local/ston/stond'
          `./stonx' -> `/usr/local/ston/stonx'
          `./stonr' -> `/usr/local/ston/stonr'
          `./stonu' -> `/usr/local/ston/stonu'
          `./stonapi' -> `/usr/local/ston/stonapi'
          `./server.xml.default' -> `/usr/local/ston/server.xml.default'
          `./vhosts.xml.default' -> `/usr/local/ston/vhosts.xml.default'
          `./ston_format.sh' -> `/usr/local/ston/ston_format.sh'
          `./ston_diskinfo.sh' -> `/usr/local/ston/ston_diskinfo.sh'
          `./wm.sh' -> `/usr/local/ston/wm.sh'
      [Exporting config files]
          #Export so directory
          /usr/local/ston/ to ld.so.conf
          #Export sysctl to /etc/sysctl.conf
          vm.swappiness=0
          vm.min_free_kbytes=524288
          #Export sudoers for WM
          Defaults    !requiretty
          winesoft ALL=NOPASSWD: /etc/init.d/ston stop, /etc/init.d/ston start, /bin/ps -ef
      [Configuring STON daemon script]
          STON deamon activate in run-level 2345.
      [Installing sub-packages]
          curl installed.
          libjpeg installed.
          libgomp installed.
          rrdtool installed.
      [Installing WM]
          Stopping WM...
          WM stopped
          `./wm.server_default.xml' -> `/usr/local/ston/wm/tmp/conf/server_default.xml'
          `./wm.vhost_default.xml' -> `/usr/local/ston/wm/tmp/conf/vhost_default.xml'
          WM configuration found. Current WM port : 8500
          PHP module for Legacy(CentOS 5.5) installed
          `./libphp5.so.5.5' -> `/usr/local/ston/wm/modules/libphp5.so'
          WM installation almost complete. Changing WM privileges.
      Installation successfully complete


.. _getting-started-update:

Update
====================================
When the updated version is released, use stonu command to update STON to the latest version. ::

	./stonu 2.0.1
	
:ref:`wm-update` option from :ref:`wm` command will also help easy update of STON.

   .. figure:: img/conf_update1.png
      :align: center


.. _getting-started-run:

Run
====================================

The following is the default directory where STON is usually installed at. ::

    /usr/local/ston/

If any of the below files is missing in the default directory or has incorrect XML syntax, STON will not run.

- license.xml
- server.xml
- vhosts.xml

After the initial installation, xml files might be missing from the default directory.
In this case, copy the distributed license.xml file to the directory.
Then duplicate or rename server.xml.default and vhosts.xml.default files from the default installation directory.
*.default files will be distributed with the latest package all the time.


.. _getting-started-samplevhost:

Hello World
====================================
Open vhosts.xml file and modify as below. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>hello.winesoft.co.kr</Address>
            </Origin>
        </Vhost>
    </Vhosts>   


.. _getting-started-runston:

Run STON
-----------------------------------------------
1. Copy license.xml to the install directory.

2. Open server.xml file and configure <Storage>. ::

    <Server>
        <Cache>
            <Storage>
                <Disk>/cache1/</Disk>
                <Disk>/cache2/</Disk>
            </Storage>
        </Cache>
    </Server>
      
.. note::

   STON uses disk as a storage, default disk has to be configured in order to run STON.
   Setting up the disk will be reviewed in the next chapter.

3. Run STON.  ::

      [root@localhost ~]# service ston start

   In order to stop STON, use stop command.  ::

      [root@localhost ~]# service ston stop


.. _getting-started-runcheck:

Checking Virtual Host
-----------------------------------------------

(For Windows 7) Add www.example.com domain in the C:\\Windows\\System32\\drivers\\etc\\hosts file as below. ::

    192.168.0.100        www.example.com

If all settings are correctly configured, the following page will be displayed on the browser when www.example.com is accessed.

   .. figure:: img/helloworld3.png
      :align: center


.. _getting-started-rrderr:

Trouble Shooting for Slow WM or Graph Displaying Error
-----------------------------------------------

RRD graph is dynamically downloaded and installed during installation procedure.
Therefore, RRD may not be installed properly under the restricted network.
In addition, :ref:`wm` may run very slowly or :ref:`api-graph` may not work at all.
In order to fix these issues, follow the belw steps.


**1. Checking the rrdtool Installation Status**

   Below procedure is to check the rrdtool installation status. ::
   
      [root@localhost ston]# yum install rrdtool
      Loaded plugins: fastestmirror, security
      Loading mirror speeds from cached hostfile
      * base: centos.mirror.cdnetworks.com
      * elrepo: ftp.ne.jp
      * epel: mirror.premi.st
      * extras: centos.mirror.cdnetworks.com
      * updates: centos.mirror.cdnetworks.com
      Setting up Install Process
      Package rrdtool-1.3.8-6.el6.x86_64 already installed and latest version
      Nothing to do
      
   Below procedure is the rrdtool installation check for Ubuntu. ::

      root@ubuntu:~# apt-get install rrdtool
      Reading package lists... Done
      Building dependency tree
      Reading state information... Done
      rrdtool is already the newest version.
      The following packages were automatically installed and are no longer required:
        libgraphicsmagick3 libgraphicsmagick++3 libgraphicsmagick1-dev libgraphics-magick-perl libgraphicsmagick++1-dev
      Use 'apt-get autoremove' to remove them.
      0 upgraded, 0 newly installed, 0 to remove and 102 not upgraded.
      
      
**2. RRD Manual Installation**

   If yum fails to install rrdtool, administrator should `downlaod <http://pkgs.repoforge.org/rrdtool/>`_ 
   proper package for installed OS version and manually proceed installation.
   
======================================== =================== ======= ============================
Name                                     Last Modified       Size    Description
======================================== =================== ======= ============================
tcl-rrdtool-1.4.7-1.el5.rf.i386.rpm      06-Apr-2012 16:57   36K     RHEL5 and CentOS-5 x86 32bit
tcl-rrdtool-1.4.7-1.el5.rf.x86_64.rpm	 06-Apr-2012 16:57   37K     RHEL5 and CentOS-5 x86 64bit
tcl-rrdtool-1.4.7-1.el6.rfx.i686.rpm     06-Apr-2012 16:57   35K     RHEL6 and CentOS-6 x86 32bit
tcl-rrdtool-1.4.7-1.el6.rfx.x86_64.rpm	 06-Apr-2012 16:57   35K     RHEL6 and CentOS-6 x86 64bit
======================================== =================== ======= ============================


.. _env-vhost-activeorigin:

Origin Server
============================================

Virtual host is intended to serve contents to users on behalf of the origin server. 
Depends on the service platform, various origin servers can be accessed by serveral ways.

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>1.1.1.1</Address>
                <Address>1.1.1.2</Address>
            </Origin>
        </Vhost>
    </Vhosts>

-  ``<Address>``
   The address of origin server where the virtual host duplicates contents.
   Unlimited number of addresses can be added in the list. (무엇에 대한 개수제한이 없는지??)개수제한은 없다.
   When there are more than 2 addresses, Active/Active method(Round-Robin) is adopted to choose an address.
   If the origin server port is 80, it can be omitted.

For example, if the origin server is using a different port such as 8080, then port number has to be specified as 1.1.1.1:8080.
There are 8 different types of address based on {IP|Domain}{Port}{Path} format.

============================== ==========================
Address                        Host Header
============================== ==========================
1.1.1.1	                       Virtual host name
1.1.1.1:8080	               Virtual host name:8080       
1.1.1.1/account/dir	           Virtual host name            
1.1.1.1:8080/account/dir       Virtual host name:8080       
example.com	                   example.com             
example.com:8080	           example.com:8080        
example.com/account/dir	       example.com             
example.com:8080/account/dir   example.com:8080
============================== ==========================

As long as the host header is not specified in the :ref:`origin-httprequest`, the host header from above table will be trasmitted. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>origin.com:8888/account/dir</Address>            
            </Origin>
        </Vhost>
    </Vhosts>

For example, the above setting will request below host header as an origin server. ::

   GET / HTTP/1.1
   Host: origin.com:8888

.. note:

   원본서버에 example.com/account/dir처럼 경로가 붙어있다면 요청된 URL은 원본서버 주소 경로 뒤에 붙는다. 
   클라이언트가 /img.jpg를 요청하면 최종 주소는 example.com/account/dir/img.jpg가 된다.


.. _env-vhost-standbyorigin:

보조 원본주소
------------------------------------------------

보조 원본서버를 설정한다.::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>1.1.1.1</Address>
                <Address>1.1.1.2</Address>
                <Address2>1.1.1.3</Address2>
                <Address2>1.1.1.4</Address2>
            </Origin>
        </Vhost>
    </Vhosts>
    
-  ``<Address2>``

   모든 ``<Address>`` 가 정상동작하고 있다면 ``<Address2>`` 는 서비스에 투입되지 않는다. 
   Active서버에 장애가 감지되면 해당 서버를 대체하기 위해 투입되며 
   Active서버가 복구되면 다시 Standby상태로 돌아간다. 
   만약 Standby서버에 장애가 감지되면 해당 Standby서버가 복구되기 전까지 서비스에 투입되지 않는다.


.. _api-etc-help:

API 호출
====================================

HTTP기반의 API를 제공한다.
API 호출권한은 :ref:`env-host` 의 영향을 받는다. 
허가되지 않았다면 곧바로 연결을 종료한다.

STON버전을 확인한다. ::

   http://127.0.0.1:10040/version
    
같은 API를 Linux Shell에서 명령어로 수행한다. ::

   ./stonapi version

.. note:

   HTTP API는 &를 QueryString의 구분자로 인식하지만 Linux 콘솔에서는 다른 의미를 가진다. 
   &가 들어가는 명령어를 호출하는 경우 \&로 입려하거나 반드시 괄호(" /...&... ")로 호출하는 URL을 묶어야 한다.


하드웨어 정보조회
====================================

하드웨어 정보를 조회한다. :: 

   http://127.0.0.1:10040/monitoring/hwinfo
   
결과는 JSON형식으로 제공된다. ::

   {
      "version": "1.1.9",
      "method": "hwinfo",
      "status": "OK",
      "result":
      {
         "OS" : "Linux version 3.3.0 ...(생략)...", 
         "STON" : "1.1.9", 
         "CPU" : 
         { 
            "ProcCount": "4", 
            "Model": "Intel(R) Xeon(R) CPU           E5606  @ 2.13GHz", 
            "MHz": "1200.000", 
            "Cache": "8192 KB"
         }, 
         "Memory" : "8 GB", 
         "NIC" : 
         [ 
            { 
               "Dev" : "eth1", 
               "Model" : "Intel Corporation 82574L Gigabit Network Connection", 
               "IP" : "192.168.0.13", 
               "MAC" : "00:25:90:36:f4:cb" 
            } 
         ], 
         "Disk" : 
         [ 
            { 
               "Dev" : "sda", 
               "Model" : "HP DG0146FAMWL (scsi)", 
               "Total" : "238787584", 
               "Usage" : "40181760" 
            },
            {
               "Dev" : "sdb", 
               "Model" : "HITACHI HUC103014CSS600 (scsi)", 
               "Total" : "144706478080", 
               "Usage" : "2101075968" 
            },
            {
               "Dev" : "sdc", 
               "Model" : "HITACHI HUC103014CSS600 (scsi)", 
               "Total" : "144706478080", 
               "Usage" : "2012160000" 
            }
         ]
      } 
   }


재시작/종료
====================================

명령어를 통해 STON을 재시작/종료할 수 있다. 
의도하지 않은 결과를 피하기 위해 웹 페이지를 통한 확인작업이 반드시 필요하도록 개발되었다. ::

   http://127.0.0.1:10040/command/restart
   http://127.0.0.1:10040/command/restart?key=JUSTDOIT       // 즉시 실행
   http://127.0.0.1:10040/command/terminate
   http://127.0.0.1:10040/command/terminate?key=JUSTDOIT       // 즉시 실행
   

.. _getting-started-reset:

Caching 초기화
====================================

서비스를 중단하며 캐싱된 모든 컨텐츠를 삭제한다. 
설정된 모든 디스크를 포맷하며 작업이 완료되면 다시 서비스를 재개한다. ::

   http://127.0.0.1:10040/command/cacheclear
   http://127.0.0.1:10040/command/cacheclear?key=JUSTDOIT       // 즉시 실행
   
콘솔에서는 다음 명령어를 통해 전체 또는 하나의 가상호스트를 초기화한다. ::

   ./stonapi reset
   ./stonapi reset/ston.winesoft.co.kr
