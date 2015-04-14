.. _getting-started:

Chapter 2. Getting Started
******************

This chapter explains abot installation and configuration up to setting up a sample virtual host. A simple text editor is all you need.

STON Edge Server is proprietary software, running on 64-bit Linux servers. A suitable selection of hardware components is important.

.. toctree::
   :maxdepth: 2

.. _getting-started-serverconf:

Choosing the best hardware
====================================

Components such as CPU, memory and storage have to be chosen carefully. Especially for a service that requires a high performance such as 10Gbps throughput, each component must meet the requirements in order to reach desired performance.

-  **CPU**

   A quad core processor is recommended at least.
   Many-Core CPU increases the scalability of STON Edge Server.
   Every additional processor core may boost the processing power of the server.
   However, high processing power does not necessarily mean high traffic transmission.

   .. figure:: img/10g_cpu.jpg
      :align: center

      More client access needs more CPU cores.

   In terms of bandwidth, transferring a 4KB file for 262,144 times or a 1GB file for 1 time takes the same size of bandwidth.
   The number of simultaneous connections is the most critical criterion for selecting CPUs. 


-  **Memory**

   At least 4GB of system memory is recommended for memory indexing. ( :ref:`adv_topics_mem` )
   Memory should be allocated to frequently accessed contents in order to increase the performance.
   A server lacking physical memory will inevitably access to the storage disk more frequently, creating an increases burden on the disk.
   If the I/O load of the disk is significantly high, regardless of the number of served content, installing additional memory can reduce the I/O load.
   

-  **Disk**

   At least three disks, including the OS disk are recommended.
   Just like memory, more disks give better performance because the I/O load will be dispersed and more contents can be cached.
   
   .. figure:: img/02_disk.png
      :align: center
      
      OS and STON always have to be installed on a disk separate from content disks.
   
   STON is installed on the OS disk.
   The log is also configured on the identical disk where OS is installed.
   The log records real time service status, thus, it causes write load.
   
   STON utilizes disks in the RAID 0 setting.
   Depending on the required performance of the client's service, RAID may or may not be adopted.
   However, when file modification is not frequent and the size of content is much larger than that of physical memory, read speed can be effectively increased via RAID.


.. _getting-started-os:

OS Configuration
====================================

Default OS installation is enough.
STON works on the 64-bit Linux(CentOS 6.2 or higher, Ubuntu 10.04 or higher), and does not require any particular package.


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


2. Extract the downloaded package. ::

		[root@localhost ~]# tar -zxf ston.2.0.0.rhel.2.6.32.x64.tar.gz

3. Run the installation script. ::

		[root@localhost ~]# ./ston.2.0.0.rhel.2.6.32.x64.sh

4. All installation processes are recorded in the install.log file. The log file helps track any possible issues during the installation process. ::

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
Use the "stonu" command to update STON to the latest version. ::

	./stonu 2.0.1
	
:ref:`wm-update` option from :ref:`wm` command will also help easily update STON.

   .. figure:: img/conf_update1.png
      :align: center


.. _getting-started-run:

Run
====================================

The following path is the default directory to install STON. ::

    /usr/local/ston/

If any of the below files is missing in the default directory or has invalid XML syntax, STON would not be running.

- license.xml
- server.xml
- vhosts.xml

After the initial installation, xml files might be missing from the default directory.
In this case, copy the distributed license.xml file to the directory.
Then duplicate or rename server.xml.default and vhosts.xml.default files from the default installation directory.
*.default files will be distributed with the latest package.


.. _getting-started-samplevhost:

Hello World
====================================
Open vhosts.xml file and modify, as shown below. ::

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

   STON uses disk storage. A default disk has to be configured in order to run STON.
   Disk set-up details are in the next chapter.

3. Run STON.  ::

      [root@localhost ~]# service ston start

   In order to stop STON, use the stop command. ::

      [root@localhost ~]# service ston stop


.. _getting-started-runcheck:

Checking Virtual Host
-----------------------------------------------

(For Windows 7) Add www.example.com domain in the C:\\Windows\\System32\\drivers\\etc\\hosts file as shown below. ::

    192.168.0.100        www.example.com

If all settings are correctly configured, the following page will be displayed on the browser when www.example.com is accessed.

   .. figure:: img/helloworld3.png
      :align: center


.. _getting-started-rrderr:

Trouble Shooting for Slow WM or Graph Displaying Error
-----------------------------------------------

RRD graph is dynamically downloaded and installed during the installation procedure.
Therefore, RRD may not be installed properly under the restricted network.
In addition, :ref:`wm` may run very slowly or :ref:`api-graph` may not work at all.
In order to fix these issues, follow the steps below.


**1. Check the RRDtool Installation Status**

   The procedure displayed below features how to check the RRDtool installation status. ::
   
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
      
   (for Ubuntu) ::

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

   If YUM fails to install the RRDtool, the administrator should `download <http://pkgs.repoforge.org/rrdtool/>`_ 
   the proper package for installing the OS version and manually proceed with installation.
   
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

A virtual host is intended to serve content to users on behalf of the origin server. 
Depending on the service platform, any origin server is accessible by IP or domain address.

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>1.1.1.1</Address>
                <Address>1.1.1.2</Address>
            </Origin>
        </Vhost>
    </Vhosts>

-  ``<Address>``
   The address of the origin server where the virtual host duplicates content.
   An unlimited number of origin addresses can be added to the list. 
   When there are more than 2 addresses, Active/Active method(Round-Robin) is adopted to choose an address.
   If the origin server port is 80, it can be omitted.

For example, if the origin server is using a different port such as 8080, then the port number has to be specified as 1.1.1.1:8080.
There are eight different types of addresses based on the {IP|Domain}{Port}{Path} format.

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

As long as the host header is not specified in the :ref:`origin-httprequest`, the host header from the table above will be transmitted. ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>origin.com:8888/account/dir</Address>            
            </Origin>
        </Vhost>
    </Vhosts>

For example, the setting above will request the host header below as an origin server. ::

   GET / HTTP/1.1
   Host: origin.com:8888

.. note:

   If additional path is attached at the origin server like example.com/account/dir,
   requested URL will also be attached at the end of origin server address.
   For example, if a client requests /img.jpg, then the final address will be example.com/account/dir/img.jpg.

.. _env-vhost-standbyorigin:

Standby Origin Server Address
------------------------------------------------

Configure the standby origin server, as shown below. ::

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

   If all ``<Address>`` are working without problems, ``<Address2>`` will not run for the service.
   However, if any failure is detected in the active servers from ''<Address>'', the substitute ones from ''<Address2>'' are activated.
   If servers from ''<Address>'' are recovered, the substitue servers return to standby status.
   If there is any error in the substitue servers, the substitute server will never become active nor standby for service until all errors are recovered. 


.. _api-etc-help:

API Call
====================================

The STON provides HTTP based API.
:ref:`env-host` authorizes an API call, and if the API call is unauthorized, the connection will be terminated immediately.

Check STON Version. ::

   http://127.0.0.1:10040/version
    
Execute the identical API version by checking command on the Linux Shell. ::

   ./stonapi version

.. note:

   HTTP API recognizes "&" as a QueryString identifier, but Linux console takes it in a different meaning.
   In the Linux console when calling a command with "&", administrator should use either "\&" or wrap the URL with parenthesis.


Hardware Information Inquiry
====================================

The following API looks over hardware information. :: 

   http://127.0.0.1:10040/monitoring/hwinfo
   
The result will be returned into JSON format as shown below. ::

   {
      "version": "1.1.9",
      "method": "hwinfo",
      "status": "OK",
      "result":
      {
         "OS" : "Linux version 3.3.0 ...(skip)...", 
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


Restart/Quit
====================================

The following commands restart or quit STON. 
In order to avoid an unintended result, STON provides a confirmation for restart/quit commands on the web page. ::

   http://127.0.0.1:10040/command/restart
   http://127.0.0.1:10040/command/restart?key=JUSTDOIT       // Immediately execute the command
   http://127.0.0.1:10040/command/terminate
   http://127.0.0.1:10040/command/terminate?key=JUSTDOIT       // Immediately execute the command
   

.. _getting-started-reset:

Caching Reset
====================================

The following commands stop service and discard all cached contents.
The command will format all disks and the service will resume, when the disk format is completed. ::

   http://127.0.0.1:10040/command/cacheclear
   http://127.0.0.1:10040/command/cacheclear?key=JUSTDOIT       // Immediately execute the command
   
In the console window, the following command will reset one or the entire virtual host. ::

   ./stonapi reset
   ./stonapi reset/ston.winesoft.co.kr
