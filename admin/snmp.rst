.. _snmp:

Chapter 16. SNMP
******************

This chapter explains SNMP(Simple Network Monitoring Protocol).
All values of :ref:`monitoring_stats` are also provided as SNMP.
Besides, SNMP provides more segmented time units and more detailed system status information.
SNMP provides real time stats and average stats(up to 60 minutes) for each virtual host. 

- No additional package is required.
- Does not run SNMP separately.
- Supports SNMP v1 and v2c.

.. toctree::
   :maxdepth: 2


.. _snmp-var:

Variables
====================================

A value that can be changed by a configuration or administrator's intention is specified as [variable name]. 
For example, if there are multiple disks, a designated reference number for each disk is required.
These values, which are specified as the ``[diskIndex]``, are assigned in sequential order beginning at 1. 

-  ``[diskIndex]``

   This index stands for all configured disks in storage. ::
   
      # server.xml - <Server><Cache>
   
      <Storage>
         <Disk>/cache1</Disk>
         <Disk>/cache2</Disk>
         <Disk>/cache3</Disk>
      </Storage>
      
   In the above configuration with 3 disks, the ``[diskIndex]`` of /cache1 becomes 1, and the ``[diskIndex]`` of /cache3 becomes 3. 
   For example, the OID, which refers to the entire volume of /cache1, is system.diskInfo.diskInfoTotalSize.1(1.3.6.1.4.1.40001.1.2.18.1.3.1).
   The last .1 refers to the first disk.
   
-  ``[vhostIndex]`` 

   This index is automatically assigned when the virtual host is loaded. ::
   
      # vhosts.xml
   
      <Vhosts>
         <Vhost Status="Active" Name="kim.com"> ... </Vhost>
         <Vhost Status="Active" Name="lee.com"> ... </Vhost>
         <Vhost Status="Active" Name="park.com" StaticIndex="10300"> ... </Vhost>
      </Vhosts>
   
   If the first three virtual hosts are loaded like the above configuration, a sequential number from 1 is assigned to the ``[vhostIndex]``. 
   Then the virtual host remembers the permanent ``[vhostIndex]`` even when the virtual host is discarded. 
   If an addition and a deletion of a virtual host occurs concurrently, deletion comes first and the newly added virtual host will be given an empty ``[vhostIndex]``.
   
   .. figure:: img/snmp_vhostindex.png
      :align: center
      
      Operation method of ``[vhostIndex]``

-  ``[diskMin]`` , ``[vhostMin]`` 

   These stand for time in minutes. 
   A value of 5 refers to the average of the last 5 minutes, and 60 refers to the average of the last 60 minutes. 
   Up to 60 minutes of data can be requested, and 0 refers to the real time data of every second.
   
SNMP uses Table architecture for the items with dynamically changing values. 
For example, "Total disk size" can differ based on the number of disks so it has to adopt Table architecture to express data. 
STON provides stats every minute for all virtual hosts. 
Therefore, you can use a complicated expression like ``[vhostMin]`` . ``[vhostIndex]``. 

This expression requests the stats of each virtual host for every minute, 
which is hard to express with Table architecture because it contains two variables. 
In order to resolve this problem, you can set the default value of ``[vhostMin]`` to enable SNMPWalk.


.. _snmp-conf:

Activation
====================================

You can set the SNMP operation method and ACL in the global setting(server.xml). ::

   # server.xml - <Server><Host>

   <SNMP Port="161" Status="Inactive">
      <Allow>192.168.5.1</Allow>
      <Allow>192.168.6.0/24</Allow>    
   </SNMP>   

-  The ``<SNMP>`` attribute configures the operation method of SNMPdml.

   - ``Port (default: 161)`` SNMP service port
   
   - ``Status (default: Inactive)`` Set this value to ``Active`` in order to activate SNMP.
   
-  ``<Allow>`` Configures an IP address to allow SNMP access. 
    Designated IP, designated IP range, bitmask and subnet are supported. 
    If the connected socket is not an approved IP, no response is returned.
    
    

Virtual Host/View Variables
====================================

This section explains how to configure the number of virtual host/View and the default time(in minutes) provided through SNMP. ::

   # server.xml - <Server><Host>
   
   <SNMP VHostCount=0, VHostMin=5 ViewCount=0, ViewMin=5 />

-  ``VHostCount (default: 0)`` If this is set to 0, only existing virtual hosts respond. 
   If this value is greater than 0, all configured virtual hosts will also respond whether they actually exist or not. 
   
-  ``ViewCount (default: 0)`` Applied to the View. ( identical to ``VHostCount`` )
   
-  ``VHostMin (default: 5 minutes, maximum: 60 minutes)``  Configures the ``[vhostMin]`` value. 
   Values from 0 to 60 are valid. 
   Setting this to 0 will return the real time data, whereas 1~60 returns the average data of the corresponding number of minutes.
   
-  ``ViewMin (default: 0)`` Applied to the View. ( identical to ``VHostMin`` ).

For example, the operation method of SNMPWalk in a three virtual host environment differs based on the VHostCount value.

- When VHostCount=0 ::

    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.1 = STRING: "web.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.2 = STRING: "img.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.3 = STRING: "vod.winesoft.co.kr"
    
- When VHostCount=5 ::

    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.1 = STRING: "web.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.2 = STRING: "img.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.3 = STRING: "vod.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.4 = ""
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.5 = ""



Other Variables
---------------------

Configures other variables. ::

   # server.xml - <Server><Host>
   
   <SNMP GlobalMin="5" DiskMin="5" ConfCount="10" />
    
-  ``GlobalMin (default: 5 minutes, maximum: 60 minutes)`` Configures the ``[globalMin]`` value.

-  ``DiskMin (default: 5 minutes, maximum: 60 minutes)`` Configures the ``[diskMin]`` value.

-  ``ConfCount (default: 10)`` Browses previous configuration lists. 
   Values from 1 to 100 are valid. 
   1 will browse the current configuration only, and 2 will browse the current and the previous configurations.
   Thus, a value of 100 will allow you to browse 99 previous configurations.
   


Community
====================================

This section explains how to set the community to allow/deny access to the specific OID. ::

   # server.xml - <Server><Host>

   <SNMP UnregisteredCommunity="Allow">
      <Community Name="example1" OID="Allow">
         <OID>1.3.6.1.4.1.40001.1.4.1</OID>
         <OID>1.3.6.1.4.1.40001.1.4.2</OID>
         <OID>1.3.6.1.4.1.40001.1.4.4</OID>
      </Community>
      <Community Name="example2" OID="Deny">
         <OID>1.3.6.1.4.1.40001.1.4.3.1.11.11.10.1-61</OID>
      </Community>
   </SNMP>
    
If you set the ``UnregisterdCommunity`` of ``<SNMP>`` as "Deny", unregistered Community requests are blocked.

-  ``<Community>`` Configures Community.

   - ``Name`` The Community name.
   
   - ``OID (default: Allow)`` Configures the lower ``<OID>`` tag value.
     If this value is set to ``Allow``, only the lower ``<OID>`` list can be accessed. 
     On the other hand, if the value is set to ``Deny``, the lower <OID> list can't be accessed.

Specific OID(1.3.6.1.4.1.40001.1.4.4) and ranged OID(1.3.6.1.4.1.40001.1.4.3.1.11.11.10.1-61) expressions are available. 
If you allow/deny OID, all subordinate OIDs follow the same configuration.



.. _snmp-meta:

meta
====================================

::

   OID = 1.3.6.1.4.1.40001.1.1

Provides meta information. 

===== ============= ========= ===========================================
OID   Name          Type      Description
===== ============= ========= ===========================================
.1    manufacture   String    "WineSOFT Inc."
.2    software      String    "STON"
.3    version       String    Version
.4    hostname      String    Host name
.5    state         String    "Healthy" or "Inactive" or "Emergency"
.6    uptime        Integer   Running time (in seconds)
.7    admin         String    <Admin> ... </Admin>
.10   Conf          OID       Conf expansion
===== ============= ========= ===========================================



.. _snmp-meta-conf:

meta.conf
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.1.10

The ``[confIndex]`` is configured in the ``ConfCount`` property in ``<SNMP>``.
The ``[confIndex]`` value of 1 refers to the current configuration values, whereas 2 refers to the previous configuration values. 
Thus, if the ``[confIndex]`` is 10, 9th prior configuration values will be returned.

==================== ======= ======= =============================================================================================
OID                  Name    Type    Description
==================== ======= ======= =============================================================================================
.1. ``[confIndex]``  ID      Integer Configuration ID
.2. ``[confIndex]``  Time    Integer Configuration time (Unix time)
.3. ``[confIndex]``  Type    Integer Configuration type (0 = Unknown, 1 = STON start, 2 = /conf/reload, 3 = /conf/upload, 4 = /conf/restore)
.4. ``[confIndex]``  Size    Integer Configuration file size
.5. ``[confIndex]``  Hash    String  Configuration file Hash string
.6. ``[confIndex]``  Path    String  Saved path of the configuration file
.7. ``[confIndex]``  Ver     String  STON version of the configuration
==================== ======= ======= =============================================================================================



.. _snmp-meta-system:

System
====================================

::

   OID = 1.3.6.1.4.1.40001.1.2

Provides the system information that runs STON.
The ``[sysMin]`` variable can be set from 0 to 60(in minutes) and provides the average value for the desired time period. 
The ``[sysMin]`` in SNMPWalk is set to 0 and provides current information.

=================== ========================================= ======= ===============================================
OID                 Name                                      Type    Description
=================== ========================================= ======= ===============================================
.1. ``[sysMin]``    cpuTotal                                  Integer Total CPU usage (100%)
.2. ``[sysMin]``                                                      Total CPU usage (10,000%)
.3. ``[sysMin]``    cpuKernel                                 Integer	CPU(Kernel) usage (100%)
.4. ``[sysMin]``                                                      CPU(Kernel) usage (10,000%)
.5. ``[sysMin]``    cpuUser                                   Integer CPU(User) usage (100%)
.6. ``[sysMin]``                                                      CPU(User) usage (10,000%)
.7. ``[sysMin]``    cpuIdle                                   Integer CPU(Idle) usage (100%)
.8. ``[sysMin]``                                                      CPU(Idle) usage (10,000%)
.9                  memTotal                                  Integer Total system memory (KB)
.10. ``[sysMin]``   memUse                                    Integer The amount of memory already in use (KB)
.11. ``[sysMin]``   memFree                                   Integer Free memory (KB)
.12. ``[sysMin]``   memSTON                                   Integer Memory used for STON (KB)
.13. ``[sysMin]``   memUseRatio                               Integer System memory usage (100%)
.14. ``[sysMin]``                                                     System memory usage (10,000%)
.15. ``[sysMin]``   memSTONRatio                              Integer STON memory usage (100%)
.16. ``[sysMin]``                                                     STON memory usage (10,000%)
.17                 diskCount                                 Integer Number of disks
.18.1               diskInfo                                  OID     diskInfo expansion
.19.1               diskPerf                                  OID     diskPerf expansion
.20. ``[sysMin]``   cpuProcKernel                             Integer CPU(Kernel) usage of STON (100%)
.21. ``[sysMin]``                                                     CPU(Kernel) usage of STON (10,000%)
.22. ``[sysMin]``   cpuProcUser                               Integer CPU(User) usage of STON (100%)
.23. ``[sysMin]``                                                     CPU(User) usage of STON (10000%)
.24. ``[sysMin]``   sysLoadAverage                            Integer Load Average for 1 minute (0.01)
.25. ``[sysMin]``                                                     Load Average for 5 minutes (0.01)
.26. ``[sysMin]``                                                     Load Average for 15 minutes (0.01)
.27. ``[sysMin]``   cpuNice                                   Integer CPU(Nice) (100%)
.28. ``[sysMin]``                                                     CPU(Nice) (10,000%)
.29. ``[sysMin]``   cpuIOWait                                 Integer CPU(IOWait) (100%)
.30. ``[sysMin]``                                                     CPU(IOWait) (10,000%)
.31. ``[sysMin]``   cpuIRQ                                    Integer CPU(IRQ) (100%)
.32. ``[sysMin]``                                                     CPU(IRQ) (10,000%)
.33. ``[sysMin]``   cpuSoftIRQ                                Integer CPU(SoftIRQ) (100%)
.34. ``[sysMin]``                                                     CPU(SoftIRQ) (10,000%)
.35. ``[sysMin]``   cpuSteal                                  Integer CPU(Steal) (100%)
.36. ``[sysMin]``   CPU(Steal)                                Integer (10,000%)
.40. ``[sysMin]``   TCPSocket.Established. ``[globalMin]``    Integer Number of established TCP connections
.41. ``[sysMin]``   TCPSocket.Timewait. ``[globalMin]``       Integer Number of TIME_WAIT status TCP connections
.42. ``[sysMin]``   TCPSocket.Orphan. ``[globalMin]``         Integer Number of TCP connections that have not been attached to the file handle
.43. ``[sysMin]``   TCPSocket.Alloc. ``[globalMin]``          Integer Allocated TCP connections
.44. ``[sysMin]``   TCPSocket.Mem. ``[globalMin]``            Integer undocumented
=================== ========================================= ======= ===============================================



.. _snmp-meta-system-diskinfo:
                                    
system.diskInfo
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.2.18.1

Provides disk information.

======================= ================== =========== =========================================
OID                     Name               Type        Description
======================= ================== =========== =========================================
.2. ``[diskIndex]``     diskInfoPath       String      Disk path                                 
.3. ``[diskIndex]``     diskInfoTotalSize  Integer     Total disk size (MB)                    
.4. ``[diskIndex]``     diskInfoUseSize    Integer     Total disk usage (MB)                          
.5. ``[diskIndex]``     diskInfoFreeSize   Integer     Total available disk size (MB)                 
.6. ``[diskIndex]``     diskInfoUseRatio   Integer     Disk usage ratio (100%)                    
.7. ``[diskIndex]``                                    Disk usage ratio (10,000%)                                              
.8. ``[diskIndex]``     diskInfoStatus     String      "Normal" or "Invalid" or "Unmounted"
======================= ================== =========== =========================================



.. _snmp-meta-system-diskperf:
                                    
system.diskPerf
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.2.19.1

Provides disk performance status.

======================================== =========================== ========== ===============================
OID                                      Name                        Type       Description
======================================== =========================== ========== ===============================
.2. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadCount           Integer    Successful Read count
.3. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadMergedCount     Integer    Accumulated Read count
.4. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadSectorsCount    Integer    Total count of read sectors
.5. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadTime            Integer    Elapsed Read time (ms)
.6. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteCount          Integer    Successful Write count
.7. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteMergedCount    Integer    Accumulated Write count
.8. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteSectorsCount   Integer    Total count of written sectors
.9. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteTime           Integer    Elapsed Write time (ms)
.10. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOProgressCount     Integer    The number of IO in progress
.11. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOTime              Integer    IO elapsed time (ms)
.12. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOTimeWeighted      Integer    IO elapsed time (ms, weighted value applied)
======================================== =========================== ========== ===============================



.. _snmp-global:

global
====================================

::

   OID = 1.3.6.1.4.1.40001.1.3

Global provides resource information(sockets, events, etc.) that is commonly used for all STON modules. 

-  **ServerSocket**
   
   A section of that connects STON to clients. This socket is used by STON to process client requests.
   
-  **ClientSocket**

   A section of that connects STON to the origin server. This socket is used by STON to send requests to the origin server.

===== =========================================== ========== ==================================================
OID   Name                                        Type       Description
===== =========================================== ========== ==================================================
.5    EQ. ``[globalMin]``                         Integer    The number of unprocessed Events in the STON Framework
.6    RQ. ``[globalMin]``                         Integer    The number of Events that are saved in the recently serviced content reference que
.7    waitingFiles2Write. ``[globalMin]``         Integer    The number of write pending files
.10   ServerSocket.Total. ``[globalMin]``         Integer    Total number of server sockets
.11   ServerSocket.Established. ``[globalMin]``   Integer    Total number of connected server sockets
.12   ServerSocket.Accepted. ``[globalMin]``      Integer    Total number of newly connected server sockets
.13   ServerSocket.Closed. ``[globalMin]``        Integer    The number of closed server sockets
.20   ClientSocket.Total. ``[globalMin]``         Integer    Total number of client sockets
.21   ClientSocket.Established. ``[globalMin]``   Integer    Total number of connected client sockets
.22   ClientSocket.Accepted. ``[globalMin]``      Integer    Total number of newly connected client sockets
.23   ClientSocket.Closed. ``[globalMin]``        Integer    The number of closed client sockets
.30   ServiceAccess.Allow. ``[globalMin]``        Integer    The number of allowed(Allow) sockets by ServiceAccess
.31   ServiceAccess.Deny. ``[globalMin]``         Integer    The number of denied(Deny) sockets by ServiceAccess
===== =========================================== ========== ==================================================



.. _snmp-cache:

cache
====================================

::
   
    OID = 1.3.6.1.4.1.40001.1.4

Cache service stats are elaborately collected/provided for each virtual host.

====== ============== ========= ============================================================
OID    Name           Type      Description
====== ============== ========= ============================================================
.1     host           OID       Host (expansion)
.2     vhostCount     Integer   The number of virtual hosts
.3.1   vhost          OID       Stats for each virtual host
.4     vhostIndexMax  Integer   Max value of ``[vhostIndex]``. SNMPWalk works based on this value.
.10    viewCount      Integer   View count
.11.1  view           OID       Stats per View
.12    viewIndexMax   Integer   Max value of [viewIndex]. SNMPWalk works based on this value.
====== ============== ========= ============================================================



.. _snmp-cache-host:

cache.host
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1

Provides host(all virtual hosts) information.

===== ========= =========== =========================
OID   Name      Type        Description
===== ========= =========== =========================
.2    name      String      Host name
.3    status    String      "Healthy" or "Inactive"
.4    uptime    Integer     STON running time (in seconds)
.10   contents  OID         Content information (expansion)
.11   traffic   OID         Stats (expansion)
===== ========= =========== =========================



.. _snmp-cache-host-contents:

cache.host.contents
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.10

Provides content stats that are serviced by hosts (all virtual hosts).

====== ================ ========== ============================
OID    Name             Type       Description
====== ================ ========== ============================
.1     memory           Integer    Memory caching size(KB)
.2     filesTotalCount  Integer    The number of files in service
.3     filesTotalSize   Integer    Total size of files in service(MB)
.10    filesCountU1KB   Integer    The number of files smaller than 1KB
.11    filesCountU2KB   Integer    The number of files smaller than 2KB
.12    filesCountU4KB   Integer    The number of files smaller than 4KB
.13    filesCountU8KB   Integer    The number of files smaller than 8KB
.14    filesCountU16KB  Integer    The number of files smaller than 16KB
.15    filesCountU32KB  Integer    The number of files smaller than 32KB
.16    filesCountU64KB  Integer    The number of files smaller than 64KB
.17    filesCountU128KB Integer    The number of files smaller than 128KB
.18    filesCountU256KB Integer    The number of files smaller than 256KB
.19    filesCountU512KB Integer    The number of files smaller than 512KB
.20    filesCountU1MB   Integer    The number of files smaller than 1MB
.21    filesCountU2MB   Integer    The number of files smaller than 2MB
.22    filesCountU4MB   Integer    The number of files smaller than 4MB
.23    filesCountU8MB   Integer    The number of files smaller than 8MB
.24    filesCountU16MB  Integer    The number of files smaller than 16MB
.25    filesCountU32MB  Integer    The number of files smaller than 32MB
.26    filesCountU64MB  Integer    The number of files smaller than 64MB
.27    filesCountU128MB Integer    The number of files smaller than 128MB
.28    filesCountU256MB Integer    The number of files smaller than 256MB
.29    filesCountU512MB Integer    The number of files smaller than 512MB
.30    filesCountU1GB   Integer    The number of files smaller than 1GB
.31    filesCountU2GB   Integer    The number of files smaller than 2GB
.32    filesCountU4GB   Integer    The number of files smaller than 4GB
.33    filesCountU8GB   Integer    The number of files smaller than 8GB
.34    filesCountU16GB  Integer    The number of files smaller than 16GB
.35    filesCountO16GB  Integer    The number of files bigger than 16GB
====== ================ ========== ============================



.. _snmp-cache-host-traffic:

cache.host.traffic
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11

Provides the stats of cache service and host traffic (of all virtual hosts). 
All traffic stats are averaged and provided for a span of up to 60 minutes. 
The "min" stands for "minute" and is available up to 60. 
If the "min" is omitted or set to 0, real time stats are provided.

===================== =============== ======= ==============================
OID                   Name            Type    Description
===================== =============== ======= ==============================
.1. ``[vhostMin]``    requestHitRatio Integer Request Hit Ratio (100%)
.2. ``[vhostMin]``                            Request Hit Ratio (10,000%)
.3. ``[vhostMin]``    bytesHitRatio   Integer Bytes Hit Ratio (100%)
.4. ``[vhostMin]``                            Bytes Hit Ratio (10,000%)
.10                   origin          OID     Origin traffic information (expansion)
.11                   client          OID     Client traffic information (expansion)
===================== =============== ======= ==============================



.. _snmp-cache-host-traffic-origin:

cache.host.traffic.origin
---------------------

::
   
    OID = 1.3.6.1.4.1.40001.1.4.1.11.10

Provides origin server traffic statistics.. 
Origin server traffic is divided into HTTP traffic and Port bypass traffic.

========================== =================================== ========== ===================================================================
OID                        Name                                Type       Description
========================== =================================== ========== ===================================================================
.1. ``[vhostMin]``         inbound                             Integer    The average traffic received from the origin server (Bytes)
.2. ``[vhostMin]``         outbound                            Integer    The average traffic sent to the origin server (Bytes)
.3. ``[vhostMin]``         sessionAverage                      Integer    Overall average sessions from all origin servers
.4. ``[vhostMin]``         activesessionAverage                Integer    The average number of transmitting sessions among all origin server sessions
.10                        http                                OID        HTTP traffic information of the origin server
.10.1. ``[vhostMin]``      http.inbound                        Integer    The average HTTP traffic received from the origin server (Bytes)
.10.2. ``[vhostMin]``      http.outbound                       Integer    The average HTTP traffic sent to the origin server (Bytes)
.10.3. ``[vhostMin]``      http.sessionAverage                 Integer    The average number of HTTP sessions in the origin server
.10.4. ``[vhostMin]``      http.reqHeaderSize                  Integer    The average HTTP Header traffic sent to the origin server (Bytes)
.10.5. ``[vhostMin]``      http.reqBodySize                    Integer    The average HTTP Body traffic sent to the origin server (Bytes)
.10.6. ``[vhostMin]``      http.resHeaderSize                  Integer    The average HTTP Header traffic received from the origin server (Bytes)
.10.7. ``[vhostMin]``      http.resBodySize                    Integer    The average HTTP Body traffic received from the origin server (Bytes)
.10.8. ``[vhostMin]``      http.reqAverage                     Integer    The average number of HTTP requests sent to the origin server
.10.9. ``[vhostMin]``      http.reqCount                       Integer    The number of HTTP requests sent to the origin server
.10.10. ``[vhostMin]``     http.resTotalAverage                Integer    Total number of average HTTP requests sent by the origin server
.10.11. ``[vhostMin]``     http.resTotalCompleteAverage        Integer    The average number of successful HTTP transactions from the origin server
.10.12. ``[vhostMin]``     http.resTotalTimeRes                Integer    The average elapsed time until the response header from the origin server is received (0.01ms)
.10.13. ``[vhostMin]``     http.resTotalTimeComplete           Integer    The average completion time of HTTP transactions from the origin server (0.01ms)
.10.14. ``[vhostMin]``     http.resTotalCount                  Integer    Total number of HTTP responses sent by the origin server
.10.15. ``[vhostMin]``     http.resTotalCompleteCount          Integer    The number of successful HTTP transactions from the origin server
.10.20. ``[vhostMin]``     http.res2xxAverage                  Integer    The average number of 2xx responses sent by the origin server
.10.21. ``[vhostMin]``     http.res2xxCompleteAverage          Integer    The average number of successful 2xx transactions from the origin server
.10.22. ``[vhostMin]``     http.res2xxTimeRes                  Integer    The average elapsed time until 2xx responses headers are received from the origin server (0.01ms)
.10.23. ``[vhostMin]``     http.res2xxTimeComplete             Integer    The average completion time of 2xx response HTTP transactions from the origin server (0.01ms)
.10.24. ``[vhostMin]``     http.res2xxCount                    Integer    The number of 2xx responses sent by the origin server
.10.25. ``[vhostMin]``     http.res2xxCompleteCount            Integer    The number of successful 2xx transactions from the origin server
.10.30. ``[vhostMin]``     http.res3xxAverage                  Integer    The average number of 3xx responses sent by the origin server
.10.31. ``[vhostMin]``     http.res3xxCompleteAverage          Integer    The average number of successful 3xx transactions from the origin server
.10.32. ``[vhostMin]``     http.res3xxTimeRes                  Integer    The average lapsed time until 3xx response headers are received from the origin server (0.01ms)
.10.33. ``[vhostMin]``     http.res3xxTimeComplete             Integer    The average completion time of 3xx response HTTP transactions from the origin server (0.01ms)
.10.34. ``[vhostMin]``     http.res3xxCount                    Integer    The number of 3xx responses sent by the origin server
.10.35. ``[vhostMin]``     http.res3xxCompleteCount            Integer    The number of successful 3xx transactions from the origin server
.10.40. ``[vhostMin]``     http.res4xxAverage                  Integer    The average number of 4xx responses sent by the origin server
.10.41. ``[vhostMin]``     http.res4xxCompleteAverage          Integer    The average number of successful 4xx transactions from the origin server
.10.42. ``[vhostMin]``     http.res4xxTimeRes                  Integer    The average lapsed time until 4xx response headers are received from the origin server (0.01ms)
.10.43. ``[vhostMin]``     http.res4xxTimeComplete             Integer    The average completion time of 4xx response HTTP transactions from the origin server (0.01ms)
.10.44. ``[vhostMin]``     http.res4xxCount                    Integer    The number of 4xx responses sent by the origin server
.10.45. ``[vhostMin]``     http.res4xxCompleteCount            Integer    The number of successful 4xx transactions from the origin server
.10.50. ``[vhostMin]``     http.res5xxAverage                  Integer    The average number of 5xx responses sent by the origin server
.10.51. ``[vhostMin]``     http.res5xxCompleteAverage          Integer    The average number of successful 5xx transactions from the origin server
.10.52. ``[vhostMin]``     http.res5xxTimeRes                  Integer    The average lapsed time until 5xx response headers are received from the origin server (0.01ms)
.10.53. ``[vhostMin]``     http.res5xxTimeComplete             Integer    The average completion time of 5xx response HTTP transactions from the origin server (0.01ms)
.10.54. ``[vhostMin]``     http.res5xxCount                    Integer    The number of 5xx responses sent by the origin server
.10.55. ``[vhostMin]``     http.res5xxCompleteCount            Integer    The number of successful 5xx transactions from the origin server
.10.60. ``[vhostMin]``     http.connectTimeoutAverage          Integer    The average number of origin server connection timeouts
.10.61. ``[vhostMin]``     http.receiveTimeoutAverage          Integer    The average number of origin server reception timeouts
.10.62. ``[vhostMin]``     http.connectAverage                 Integer    The average number of successful connections to the origin server
.10.63. ``[vhostMin]``     http.dnsQueryTime                   Integer    The average DNS query required time for connecting to the origin server
.10.64. ``[vhostMin]``     http.connectTime                    Integer    The average required time to connect the origin server (0.01ms)
.10.65. ``[vhostMin]``     http.connectTimeoutCount            Integer    The number of origin server connection timeouts
.10.66. ``[vhostMin]``     http.receiveTimeoutCount            Integer    The number of origin server reception timeouts
.10.67. ``[vhostMin]``     http.connectCount                   Integer    The number of successful origin server connections
.10.68. ``[vhostMin]``     http.closeAverage                   Integer    The average number of sockets closed by the origin server during transmission
.10.69. ``[vhostMin]``     http.closeCount                     Integer    The number of sockets closed by the origin server during transmission
.11                        portbypass                          OID        Traffic information of the port bypass origin server
.11.1. ``[vhostMin]``      portbypass.inbound                  Integer    The average traffic received from the origin server via port bypass (Bytes)
.11.2. ``[vhostMin]``      portbypass.outbound                 Integer    The average traffic sent to the origin server via port bypass (Bytes)
.11.3. ``[vhostMin]``      portbypass.sessionAverage           Integer    The average number of origin server sessions in port bypass.
.11.4. ``[vhostMin]``      portbypass.closedAverage            Integer    The average number of connections closed by the origin server during port bypass
.11.5. ``[vhostMin]``      portbypass.connectTimeoutAverage    Integer    The average number of connection timeouts of the port bypass origin server
.11.6. ``[vhostMin]``      portbypass.closedCount              Integer    The number of connections closed by the origin server during port bypass
.11.7. ``[vhostMin]``      portbypass.connectTimeoutCount      Integer    The number of connection timeouts of the port bypass origin server
========================== =================================== ========== ===================================================================



.. _snmp-cache-host-traffic-client:

cache.host.traffic.client
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11.11

Provides client traffic stats. 
Client traffic is divided into HTTP traffic, SSL traffic and Port bypass traffic. 
SNMP does not provide stats for each directory. 
Even if directory stats are configured, the values are accumulated.

========================== ========================================== ========== =============================================================
OID                        Name                                       Type       Description
========================== ========================================== ========== =============================================================
.1. ``[vhostMin]``         inbound                                    Integer    The average traffic received from clients (Bytes)
.2. ``[vhostMin]``         outbound                                   Integer    The average traffic sent to clients (Bytes)
.3. ``[vhostMin]``         sessionAverage                             Integer    The average number of client sessions
.4. ``[vhostMin]``         activesessionAverage                       Integer    The average number of client sessions that are being transferred
.10                        http                                       OID        Client HTTP traffic information
.10.1. ``[vhostMin]``      http.inbound                               Integer    The average HTTP traffic received from clients (Bytes)
.10.2. ``[vhostMin]``      http.outbound                              Integer    The average HTTP traffic sent to clients (Bytes)
.10.3. ``[vhostMin]``      http.sessionAverage                        Integer    The average number of client HTTP sessions 
.10.4. ``[vhostMin]``      http.reqHeaderSize                         Integer    The average HTTP Header traffic received from clients (Bytes)
.10.5. ``[vhostMin]``      http.reqBodySize                           Integer    The average HTTP Body traffic received from clients (Bytes)
.10.6. ``[vhostMin]``      http.resHeaderSize                         Integer    The average HTTP Header traffic sent to clients (Bytes)
.10.7. ``[vhostMin]``      http.resBodySize                           Integer    The average HTTP Body traffic sent to clients (Bytes)
.10.8. ``[vhostMin]``      http.reqAverage                            Integer    The average HTTP requests received from clients
.10.9. ``[vhostMin]``      http.reqCount                              Integer    The number of HTTP requests received from clients
.10.10. ``[vhostMin]``     http.resTotalAverage                       Integer    The average number of responses sent to clients
.10.11. ``[vhostMin]``     http.resTotalCompleteAverage               Integer    The average number of HTTP transactions completed by clients
.10.12. ``[vhostMin]``     http.resTotalTimeRes                       Integer    The average elapsed time of client responses (0.01ms)
.10.13. ``[vhostMin]``     http.resTotalTimeComplete                  Integer    The average time takes for clients to complete HTTP transactions (0.01ms)
.10.14. ``[vhostMin]``     http.resTotalCount                         Integer    The number of responses sent to clients
.10.15. ``[vhostMin]``     http.resTotalCompleteCount                 Integer    The number of HTTP transactions completed by clients
.10.20. ``[vhostMin]``     http.res2xxAverage                         Integer    The average number of 2xx responses sent to clients
.10.21. ``[vhostMin]``     http.res2xxCompleteAverage                 Integer    The average number of 2xx transactions completed by clients
.10.22. ``[vhostMin]``     http.res2xxTimeRes                         Integer    The average 2xx response time of clients (0.01ms)
.10.23. ``[vhostMin]``     http.res2xxTimeComplete                    Integer    The average 2xx response HTTP transaction completion time of clients (0.01ms)
.10.24. ``[vhostMin]``     http.res2xxCount                           Integer    The number of 2xx responses sent to clients
.10.25. ``[vhostMin]``     http.res2xxCompleteCount                   Integer    The number of 2xx transactions completed by clients
.10.30. ``[vhostMin]``     http.res3xxAverage                         Integer    The average number of 3xx responses sent to clients
.10.31. ``[vhostMin]``     http.res3xxCompleteAverage                 Integer    The average number of 3xx transactions completed by clients
.10.32. ``[vhostMin]``     http.res3xxTimeRes                         Integer    The average 3xx response time from clients (0.01ms)
.10.33. ``[vhostMin]``     http.res3xxTimeComplete                    Integer    The average 3xx response HTTP Transaction completion time of clients (0.01ms)
.10.34. ``[vhostMin]``     http.res3xxCount                           Integer    The number of 3xx responses sent to clients
.10.35. ``[vhostMin]``     http.res3xxCompleteCount                   Integer    The number of 3xx transactions completed by clients
.10.40. ``[vhostMin]``     http.res4xxAverage                         Integer    The average number of 4xx responses sent to clients
.10.41. ``[vhostMin]``     http.res4xxCompleteAverage                 Integer    The average number of 4xx transactions completed by clients
.10.42. ``[vhostMin]``     http.res4xxTimeRes                         Integer    The average 4xx response time from clients(0.01ms)
.10.43. ``[vhostMin]``     http.res4xxTimeComplete                    Integer    The average 4xx response HTTP Transaction completion time of clients(0.01ms)
.10.44. ``[vhostMin]``     http.res4xxCount                           Integer    The number of 4xx responses sent to clients
.10.45. ``[vhostMin]``     http.res4xxCompleteCount                   Integer    The number of 4xx transactions completed by clients
.10.50. ``[vhostMin]``     http.res5xxAverage                         Integer    The average number of 5xx responses sent to clients
.10.51. ``[vhostMin]``     http.res5xxCompleteAverage                 Integer    The average number of 5xx transactions completed by clients
.10.52. ``[vhostMin]``     http.res5xxTimeRes                         Integer    The average 5xx response time from clients (0.01ms)
.10.53. ``[vhostMin]``     http.res5xxTimeComplete                    Integer    The average 5xx response HTTP Transaction completion time of clients (0.01ms)
.10.54. ``[vhostMin]``     http.res5xxCount                           Integer    The number of 5xx responses sent to clients
.10.55. ``[vhostMin]``     http.res5xxCompleteCount                   Integer    The number of 5xx transactions completed by clients
.10.60. ``[vhostMin]``     http.reqDeniedAverage                      Integer    The average of denied requests
.10.61. ``[vhostMin]``     http.reqDeniedCount                        Integer    The number of denied requests
.11                        portbypass                                 OID        Traffic information of port bypass clients
.11.1. ``[vhostMin]``      portbypass.inbound                         Integer    The average traffic received from clients via port bypass (Bytes)
.11.2. ``[vhostMin]``      portbypass.outbound                        Integer    The average traffic sent to clients via port bypass (Bytes)
.11.3. ``[vhostMin]``      portbypass.sessionAverage                  Integer    The average number of client sessions in port bypass
.11.4. ``[vhostMin]``      portbypass.closedAverage                   Integer    The average number of connections closed by the client during port bypass
.11.5. ``[vhostMin]``      portbypass.closedCount                     Integer    The number of connections closed by the client during port bypass
.12                        ssl                                        OID        SSL client traffic information
.12.2. ``[vhostMin]``      ssl.inbound                                Integer    The average traffic received from clients via SSL (Bytes)
.12.3. ``[vhostMin]``      ssl.outbound                               Integer    The average traffic sent to clients via SSL (Bytes)
.13                        requestHitAverage                          OID        The average cache HIT result
.13.1. ``[vhostMin]``      requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.13.2. ``[vhostMin]``      requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.13.3. ``[vhostMin]``      requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.13.4. ``[vhostMin]``      requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.13.5. ``[vhostMin]``      requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.13.6. ``[vhostMin]``      requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.13.7. ``[vhostMin]``      requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.13.8. ``[vhostMin]``      requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.13.9. ``[vhostMin]``      requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.13.10. ``[vhostMin]``     requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.13.11. ``[vhostMin]``     requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.14                        requestHitCount                            OID        The number of cache HIT results
.14.1. ``[vhostMin]``      requestHitCount.TCP_HIT                    Integer    TCP_HIT
.14.2. ``[vhostMin]``      requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.14.3. ``[vhostMin]``      requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.14.4. ``[vhostMin]``      requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.14.5. ``[vhostMin]``      requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.14.6. ``[vhostMin]``      requestHitCount.TCP_MISS                   Integer    TCP_MISS
.14.7. ``[vhostMin]``      requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.14.8. ``[vhostMin]``      requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.14.9. ``[vhostMin]``      requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.14.10. ``[vhostMin]``     requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.14.11. ``[vhostMin]``     requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
========================== ========================================== ========== =============================================================



.. _snmp-cache-host-traffic-filesystem:

cache.host.traffic.filesystem
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11.20

Provides File I/O stats of the Host.

======================== ============================================ ========== =============================================
OID                      Name                                         Type       Description                                  
======================== ============================================ ========== =============================================
.1. ``[vhostMin]``       requestHitRatio                              Integer    Request Hit Ratio (100%)                      
.2. ``[vhostMin]``                                                               Request Hit Ratio (10,000%)                    
.3. ``[vhostMin]``       byteHitRatio                                 Integer    Byte Hit Ratio (100%)                         
.4. ``[vhostMin]``                                                               Byte Hit Ratio (10,000%)                       
.5. ``[vhostMin]``       outbound                                     Integer    The average traffic sent to File I/O (Bytes)                 
.6. ``[vhostMin]``       session                                      Integer    The average number of Thread during the File I/O process
.7                       requestHitAverage                            OID        The average cache HIT results
.7.1. ``[vhostMin]``     requestHitAverage.TCP_HIT                    Integer    TCP_HIT                                      
.7.2. ``[vhostMin]``     requestHitAverage.TCP_IMS_HIT                Integer    TCP_IMS_HIT                                  
.7.3. ``[vhostMin]``     requestHitAverage.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT                              
.7.4. ``[vhostMin]``     requestHitAverage.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT                             
.7.5. ``[vhostMin]``     requestHitAverage.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT                             
.7.6. ``[vhostMin]``     requestHitAverage.TCP_MISS                   Integer    TCP_MISS                                     
.7.7. ``[vhostMin]``     requestHitAverage.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS                             
.7.8. ``[vhostMin]``     requestHitAverage.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS                      
.7.9. ``[vhostMin]``     requestHitAverage.TCP_DENIED                 Integer    TCP_DENIED                                   
.7.10. ``[vhostMin]``    requestHitAverage.TCP_ERROR                  Integer    TCP_ERROR                                    
.7.11. ``[vhostMin]``    requestHitAverage.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT                             
.8                       requestHitCount                              OID        The number of cache HIT results                                  
.8.1. ``[vhostMin]``     requestHitCount.TCP_HIT                      Integer    TCP_HIT                                      
.8.2. ``[vhostMin]``     requestHitCount.TCP_IMS_HIT                  Integer    TCP_IMS_HIT                                  
.8.3. ``[vhostMin]``     requestHitCount.TCP_REFRESH_HIT              Integer    TCP_REFRESH_HIT                              
.8.4. ``[vhostMin]``     requestHitCount.TCP_REF_FAIL_HIT             Integer    TCP_REF_FAIL_HIT                             
.8.5. ``[vhostMin]``     requestHitCount.TCP_NEGATIVE_HIT             Integer    TCP_NEGATIVE_HIT                             
.8.6. ``[vhostMin]``     requestHitCount.TCP_MISS                     Integer    TCP_MISS                                     
.8.7. ``[vhostMin]``     requestHitCount.TCP_REFRESH_MISS             Integer    TCP_REFRESH_MISS                             
.8.8. ``[vhostMin]``     requestHitCount.TCP_CLIENT_REFRESH_MISS      Integer    TCP_CLIENT_REFRESH_MISS                      
.8.9. ``[vhostMin]``     requestHitCount.TCP_DENIED                   Integer    TCP_DENIED                                   
.8.10. ``[vhostMin]``    requestHitCount.TCP_ERROR                    Integer    TCP_ERROR                                    
.8.11. ``[vhostMin]``    requestHitCount.TCP_REDIRECT_HIT             Integer    TCP_REDIRECT_HIT                             
.10. ``[vhostMin]``      getattr.filecount                            Integer    (getattr function call) The number of FILE responses
.11. ``[vhostMin]``      getattr.dircount                             Integer    (getattr function call) The number of DIR responses
.12. ``[vhostMin]``      getattr.failcount                            Integer    (getattr function call) The number of failure responses
.13. ``[vhostMin]``      getattr.timeres                              Integer    (getattr function call) Response time (0.01ms)                 
.14. ``[vhostMin]``      open.count                                   Integer    The number of open function calls
.15. ``[vhostMin]``      open.timeres                                 Integer    The response time of the open function (0.01ms)                         
.16. ``[vhostMin]``      read.count                                   Integer    The number of read function calls
.17. ``[vhostMin]``      read.timeres                                 Integer    The response time of the read function (0.01ms)                         
.18. ``[vhostMin]``      read.buffersize                              Integer    The size of buffer requested by the read function (Bytes)                   
.19. ``[vhostMin]``      read.bufferfilled                            Integer    The amount of filled space of the buffer requested by the read function (Bytes)               
======================== ============================================ ========== =============================================



.. _snmp-cache-vhost:

cache.vhost
====================================

::
  
   OID = 1.3.6.1.4.1.40001.1.4.3.1

Provides virtual host information. The ``[vhostIndex]`` value starts at 1 and reaches up to the number of virtual hosts.

======================= ========= ========== ============================================
OID                     Name      Type       Description
======================= ========= ========== ============================================
.2. ``[vhostIndex]``    name      String     Virtual host name
.3. ``[vhostIndex]``    status    String     "Healthy" or "Inactive" or "Emergency"
.4. ``[vhostIndex]``    uptime    Integer    Virtual host running time (in seconds)
.10                     contents  OID        Content information (expansion)
.11                     traffic   OID        Stats (expansion)
======================= ========= ========== ============================================



.. _snmp-cache-vhost-contents:

cache.vhost.contents
---------------------

::
   
   OID = 1.3.6.1.4.1.40001.1.4.3.1.10

Provides content stats serviced by the virtual host.

========================= =================== ========== =============================
OID                       Name                Type       Description
========================= =================== ========== =============================
.1. ``[vhostIndex]``      memory              Integer    Memory caching size (KB)
.2. ``[vhostIndex]``      filesTotalCount     Integer    The number of files in service
.3. ``[vhostIndex]``      filesTotalSize      Integer    Total size of files in service (MB)
.10. ``[vhostIndex]``     filesCountU1KB      Integer    The number of files smaller than 1KB
.11. ``[vhostIndex]``     filesCountU2KB      Integer    The number of files smaller than 2KB
.12. ``[vhostIndex]``     filesCountU4KB      Integer    The number of files smaller than 4KB
.13. ``[vhostIndex]``     filesCountU8KB      Integer    The number of files smaller than 8KB
.14. ``[vhostIndex]``     filesCountU16KB     Integer    The number of files smaller than 16KB
.15. ``[vhostIndex]``     filesCountU32KB     Integer    The number of files smaller than 32KB
.16. ``[vhostIndex]``     filesCountU64KB     Integer    The number of files smaller than 64KB
.17. ``[vhostIndex]``     filesCountU128KB    Integer    The number of files smaller than 128KB
.18. ``[vhostIndex]``     filesCountU256KB    Integer    The number of files smaller than 256KB
.19. ``[vhostIndex]``     filesCountU512KB    Integer    The number of files smaller than 512KB
.20. ``[vhostIndex]``     filesCountU1MB      Integer    The number of files smaller than 1MB
.21. ``[vhostIndex]``     filesCountU2MB      Integer    The number of files smaller than 2MB
.22. ``[vhostIndex]``     filesCountU4MB      Integer    The number of files smaller than 4MB
.23. ``[vhostIndex]``     filesCountU8MB      Integer    The number of files smaller than 8MB
.24. ``[vhostIndex]``     filesCountU16MB     Integer    The number of files smaller than 16MB
.25. ``[vhostIndex]``     filesCountU32MB     Integer    The number of files smaller than 32MB
.26. ``[vhostIndex]``     filesCountU64MB     Integer    The number of files smaller than 64MB
.27. ``[vhostIndex]``     filesCountU128MB    Integer    The number of files smaller than 128MB
.28. ``[vhostIndex]``     filesCountU256MB    Integer    The number of files smaller than 256MB
.29. ``[vhostIndex]``     filesCountU512MB    Integer    The number of files smaller than 512MB
.30. ``[vhostIndex]``     filesCountU1GB      Integer    The number of files smaller than 1GB
.31. ``[vhostIndex]``     filesCountU2GB      Integer    The number of files smaller than 2GB
.32. ``[vhostIndex]``     filesCountU4GB      Integer    The number of files smaller than 4GB
.33. ``[vhostIndex]``     filesCountU8GB      Integer    The number of files smaller than 8GB
.34. ``[vhostIndex]``     filesCountU16GB     Integer    The number of files smaller than 16GB
.35. ``[vhostIndex]``     filesCountO16GB     Integer    The number of files bigger than 16GB
========================= =================== ========== =============================



.. _snmp-cache-vhost-traffic:

cache.vhost.traffic
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11

Provides cache service and traffic stats of the virtual host. 
All traffic stats are averaged and provided for a span of up to 60 minutes.
"min" stands for "minute" and is available up to 60. 
If the "min" is omitted or set to 0, real time stats are provided.

========================================= ================= =========== ==============================
OID                                       Name              Type        Description
========================================= ================= =========== ==============================
.1. ``[vhostMin]`` . ``[vhostIndex]``     requestHitRatio   Integer     Request Hit Ratio (100%)
.2. ``[vhostMin]`` . ``[vhostIndex]``                                   Request Hit Ratio (10,000%)
.3. ``[vhostMin]`` . ``[vhostIndex]``     bytesHitRatio     Integer     Bytes Hit Ratio (100%)
.4. ``[vhostMin]`` . ``[vhostIndex]``                                   Bytes Hit Ratio (10,000%)
.10                                       origin            OID         Origin traffic info (expansion)
.11                                       client            OID         Client traffic info (expansion)
========================================= ================= =========== ==============================



.. _snmp-cache-vhost-traffic-origin:

cache.vhost.traffic.origin
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.10

Provides origin server traffic stats. 
Origin server traffic is divided into HTTP traffic and Port bypass traffic.

============================================= ===================================== ========== =================================================================
OID                                           Name                                  Type       Description
============================================= ===================================== ========== =================================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                               Integer    The average traffic received from the origin server (Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                              Integer    The average traffic sent to the origin server (Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         sessionAverage                        Integer    The average number of sessions in the origin server
.4. ``[vhostMin]`` . ``[vhostIndex]``         activesessionAverage                  Integer    The average number of sessions that are being transferred out of all origin server sessions
.10                                           http                                  OID        Origin server HTTP traffic information
.10.1. ``[vhostMin]`` . ``[vhostIndex]``      http.inbound                          Integer    The average HTTP traffic received from the origin server (Bytes)
.10.2. ``[vhostMin]`` . ``[vhostIndex]``      http.outbound                         Integer    The average HTTP traffic sent to the origin server (Bytes)
.10.3. ``[vhostMin]`` . ``[vhostIndex]``      http.sessionAverage                   Integer    The average number of HTTP sessions in the origin server
.10.4. ``[vhostMin]`` . ``[vhostIndex]``      http.reqHeaderSize                    Integer    The average HTTP Header traffic sent to the origin server (Bytes)
.10.5. ``[vhostMin]`` . ``[vhostIndex]``      http.reqBodySize                      Integer    The average HTTP Body traffic sent to the origin server (Bytes)
.10.6. ``[vhostMin]`` . ``[vhostIndex]``      http.resHeaderSize                    Integer    The average HTTP Header traffic received from the origin server (Bytes)
.10.7. ``[vhostMin]`` . ``[vhostIndex]``      http.resBodySize                      Integer    The average HTTP Body traffic received from the origin server (Bytes)
.10.8. ``[vhostMin]`` . ``[vhostIndex]``      http.reqAverage                       Integer    The average HTTP requests sent to the origin server
.10.9. ``[vhostMin]`` . ``[vhostIndex]``      http.reqCount                         Integer    The number of HTTP requests sent to the origin server
.10.10. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalAverage                  Integer    The average number of responses sent by the origin server
.10.11. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteAverage          Integer    The average number of HTTP transactions completed by the origin server
.10.12. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeRes                  Integer    The average elapsed time until response headers are received from the origin server (0.01ms)
.10.13. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeComplete             Integer    The average time it takes for HTTP Transactions to complete (0.01ms)
.10.14. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCount                    Integer    The number of entire HTTP responses sent by the origin server
.10.15. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteCount            Integer    The number of HTTP transactions completed by the origin server
.10.20. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxAverage                    Integer    The average number of 2xx responses sent by the origin server
.10.21. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteAverage            Integer    The average number of 2xx transactions completed by the origin server
.10.22. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeRes                    Integer    The average elapsed time until 2xx response headers are received from the origin server (0.01ms)
.10.23. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeComplete               Integer    The average time it takes for 2xx response HTTP Transactions to complete from the origin server (0.01ms)
.10.24. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCount                      Integer    The number of 2xx responses sent by the origin server
.10.25. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteCount              Integer    The number of 2xx transactions completed by the origin server
.10.30. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxAverage                    Integer    The average number of 3xx responses sent by the origin server
.10.31. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteAverage            Integer    The average number of 3xx transactions completed by the origin server
.10.32. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeRes                    Integer    The average elapsed time until 3xx response headers are received from the origin server (0.01ms)
.10.33. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeComplete               Integer    The average time it takes for 3xx response HTTP Transactions to complete from the origin server (0.01ms)
.10.34. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCount                      Integer    The number of 3xx responses sent by the origin server
.10.35. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteCount              Integer    The number of 3xx transactions completed by the origin server
.10.40. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxAverage                    Integer    The average number of 4xx responses sent by the origin server
.10.41. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteAverage            Integer    The average number of 4xx transactions completed by the origin server
.10.42. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeRes                    Integer    The average elapsed time until 4xx response headers are received from the origin server (0.01ms)
.10.43. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeComplete               Integer    The average time it takes for 4xx response HTTP Transactions to complete from the origin server (0.01ms)
.10.44. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCount                      Integer    The number of 4xx responses sent by the origin server
.10.45. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteCount              Integer    The number of 4xx transactions completed by the origin server
.10.50. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxAverage                    Integer    The average number of 5xx responses sent by the origin server
.10.51. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteAverage            Integer    The average number of 5xx transactions completed by the origin server
.10.52. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeRes                    Integer    The average elapsed time until 5xx response headers are received from the origin server (0.01ms)
.10.53. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeComplete               Integer    The average time it takes for 5xx response HTTP Transactions to complete from the origin server (0.01ms)
.10.54. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCount                      Integer    The number of 5xx response sent by the origin server
.10.55. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteCount              Integer    The number of 5xx transactions completed by the origin server
.10.60. ``[vhostMin]`` . ``[vhostIndex]``     http.connectTimeoutAverage            Integer    The average number of origin server connection timeouts
.10.61. ``[vhostMin]`` . ``[vhostIndex]``     http.receiveTimeoutAverage            Integer    The average number of origin server reception timeouts
.10.62. ``[vhostMin]`` . ``[vhostIndex]``     http.connectAverage                   Integer    The average number of successful connections to the origin server
.10.63. ``[vhostMin]`` . ``[vhostIndex]``     http.dnsQueryTime                     Integer    The average DNS query time when connecting to the origin server
.10.64. ``[vhostMin]`` . ``[vhostIndex]``     http.connectTime                      Integer    The average required time to connect to the origin server (0.01ms)
.10.65. ``[vhostMin]`` . ``[vhostIndex]``     http.connectTimeoutCount              Integer    The number of origin server connection timeouts
.10.66. ``[vhostMin]`` . ``[vhostIndex]``     http.receiveTimeoutCount              Integer    The number of origin server reception timeouts
.10.67. ``[vhostMin]`` . ``[vhostIndex]``     http.connectCount                     Integer    The number of successful origin server connections
.10.68. ``[vhostMin]`` . ``[vhostIndex]``     http.closeAverage                     Integer    The average number of sockets closed by the origin server during transmission
.10.69. ``[vhostMin]`` . ``[vhostIndex]``     http.closeCount                       Integer    The number of sockets closed by the origin server during transmission
.11                                           portbypass                            OID        Traffic information of the port bypass origin server
.11.1. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.inbound                    Integer    The average traffic received from the origin server via port bypass (Bytes)
.11.2. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.outbound                   Integer    The average traffic sent to the origin server via port bypass (Bytes)
.11.3. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.sessionAverage             Integer    The average number of origin server sessions in port bypass.
.11.4. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedAverage              Integer    The average number of connections closed by the origin server during port bypass
.11.5. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.connectTimeoutAverage      Integer    The average number of connection timeouts of the port bypass origin server
.11.6. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedCount                Integer    The number of connections closed by the origin server during port bypass
.11.7. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.connectTimeoutCount        Integer    The number of connection timeouts of the port bypass origin server 
============================================= ===================================== ========== =================================================================



.. _snmp-cache-vhost-traffic-client:

cache.vhost.traffic.client
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.11

Provides client traffic stats. 
Client traffic is divided into HTTP traffic, SSL traffic, and Port bypass traffic. 
SNMP does not provide stats for each directory.
Even if directory stats are configured, the values are accumulated.

============================================= ========================================= ========== ==============================================================
OID                                           Name                                      Type       Description
============================================= ========================================= ========== ==============================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                                   Integer    The average traffic received from clients (Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                                  Integer    The average traffic sent to clients (Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         sessionAverage                            Integer    The number of average sessions in the client
.4. ``[vhostMin]`` . ``[vhostIndex]``         activesessionAverage                      Integer    The number of average sessions that are being transferred in the client
.10                                           http                                      OID        Client HTTP traffic information
.10.1. ``[vhostMin]`` . ``[vhostIndex]``      http.inbound                              Integer    The average HTTP traffic received from clients (Bytes)
.10.2. ``[vhostMin]`` . ``[vhostIndex]``      http.outbound                             Integer    The average HTTP traffic sent to clients (Bytes)
.10.3. ``[vhostMin]`` . ``[vhostIndex]``      http.sessionAverage                       Integer    The number of average HTTP sessions in the client
.10.4. ``[vhostMin]`` . ``[vhostIndex]``      http.reqHeaderSize                        Integer    The average HTTP Header traffic received from clients (Bytes)
.10.5. ``[vhostMin]`` . ``[vhostIndex]``      http.reqBodySize                          Integer    The average HTTP Body traffic received from clients (Bytes)
.10.6. ``[vhostMin]`` . ``[vhostIndex]``      http.resHeaderSize                        Integer    The average HTTP Header traffic sent to clients (Bytes)
.10.7. ``[vhostMin]`` . ``[vhostIndex]``      http.resBodySize                          Integer    The average HTTP Body traffic sent to clients (Bytes)
.10.8. ``[vhostMin]`` . ``[vhostIndex]``      http.reqAverage                           Integer    The average HTTP requests received from clients
.10.9. ``[vhostMin]`` . ``[vhostIndex]``      http.reqCount                             Integer    The number of HTTP requests received from clients
.10.10. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalAverage                      Integer    The average number of total responses sent to clients
.10.11. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteAverage              Integer    The average number of HTTP transaction completed by clients
.10.12. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeRes                      Integer    The average elapsed time of client responses (0.01ms)
.10.13. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeComplete                 Integer    The average time it takes for client HTTP Transactions to complete (0.01ms)
.10.14. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCount                        Integer    The number of responses sent to clients
.10.15. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteCount                Integer    The number of HTTP transactions completed by clients
.10.20. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxAverage                        Integer    The average number of 2xx responses sent to clients
.10.21. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteAverage                Integer    The average number of 2xx transactions completed by clients
.10.22. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeRes                        Integer    The average 2xx responses time of clients (0.01ms)
.10.23. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeComplete                   Integer    The average it takes for client's 2xx response HTTP Transactions to complete (0.01ms)
.10.24. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCount                          Integer    The number of 2xx responses sent to clients
.10.25. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteCount                  Integer    The number of 2xx transactions completed by clients
.10.30. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxAverage                        Integer    The average number of 3xx responses sent to clients
.10.31. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteAverage                Integer    The average number of 3xx transactions completed by clients
.10.32. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeRes                        Integer    The average 3xx response time from clients (0.01ms)
.10.33. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeComplete                   Integer    The average it takes for client's 3xx response HTTP Transactions to complete (0.01ms)
.10.34. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCount                          Integer    The number of 3xx responses sent to clients
.10.35. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteCount                  Integer    The number of 3xx transactions completed by clients
.10.40. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxAverage                        Integer    The average number of 4xx responses sent to clients
.10.41. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteAverage                Integer    The average number of 4xx transactions completed by clients
.10.42. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeRes                        Integer    The average 4xx response time from clients (0.01ms)
.10.43. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeComplete                   Integer    The average it takes for client's 4xx response HTTP Transactions to complete (0.01ms)
.10.44. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCount                          Integer    The number of 4xx responses sent to clients
.10.45. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteCount                  Integer    The number of 4xx transactions completed by clients
.10.50. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxAverage                        Integer    The average number of 5xx responses sent to clients
.10.51. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteAverage                Integer    The average number of 5xx transactions completed by clients
.10.52. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeRes                        Integer    The average 5xx response time from clients (0.01ms)
.10.53. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeComplete                   Integer    The average it takes for client's 5xx response HTTP Transactions to complete (0.01ms)
.10.54. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCount                          Integer    The number of 5xx responses sent to clients
.10.55. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteCount                  Integer    The number of 5xx transactions completed by clients
.10.60. ``[vhostMin]`` . ``[vhostIndex]``     http.reqDeniedAverage                     Integer    The average of denied requests
.10.61. ``[vhostMin]`` . ``[vhostIndex]``     http.reqDeniedCount                       Integer    The number of denied requests
.11                                           portbypass                                OID        Traffic information of the port bypass client
.11.1. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.inbound                        Integer    The average traffic received from clients via port bypass (Bytes)
.11.2. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.outbound                       Integer    The average traffic sent to clients via port bypass (Bytes)
.11.3. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.sessionAverage                 Integer    The average number of client sessions in port bypass
.11.4. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedAverage                  Integer    The average number of connections closed by the client during port bypass
.11.5. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedCount                    Integer    The number of connections closed by the client during port bypass
.12                                           ssl                                       OID        SSL client traffic information
.12.2. ``[vhostMin]`` . ``[vhostIndex]``      ssl.inbound                               Integer    The average traffic received from clients via SSL(Bytes)
.12.3. ``[vhostMin]`` . ``[vhostIndex]``      ssl.outbound                              Integer    The average traffic sent to clients via SSL(Bytes)
.13                                           requestHitAverage                         OID        The average cache HIT results
.13.1. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_HIT                 Integer    TCP_HIT
.13.2. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_IMS_HIT             Integer    TCP_IMS_HIT
.13.3. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REFRESH_HIT         Integer    TCP_REFRESH_HIT
.13.4. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REF_FAIL_HIT        Integer    TCP_REF_FAIL_HIT
.13.5. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_NEGATIVE_HIT        Integer    TCP_NEGATIVE_HIT
.13.6. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_MISS                Integer    TCP_MISS
.13.7. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REFRESH_MISS        Integer    TCP_REFRESH_MISS
.13.8. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_CLIENT_REFRESH_MISS Integer    TCP_CLIENT_REFRESH_MISS
.13.9. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_DENIED              Integer    TCP_DENIED
.13.10. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_ERROR               Integer    TCP_ERROR
.13.11. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REDIRECT_HIT        Integer    TCP_REDIRECT_HIT
.14                                           requestHitCount                           OID        The number of cache 
.14.1. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_HIT                   Integer    TCP_HIT
.14.2. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_IMS_HIT               Integer    TCP_IMS_HIT
.14.3. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REFRESH_HIT           Integer    TCP_REFRESH_HIT
.14.4. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REF_FAIL_HIT          Integer    TCP_REF_FAIL_HIT
.14.5. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_NEGATIVE_HIT          Integer    TCP_NEGATIVE_HIT
.14.6. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_MISS                  Integer    TCP_MISS
.14.7. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REFRESH_MISS          Integer    TCP_REFRESH_MISS
.14.8. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_CLIENT_REFRESH_MISS   Integer    TCP_CLIENT_REFRESH_MISS
.14.9. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_DENIED                Integer    TCP_DENIED
.14.10. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_ERROR                 Integer    TCP_ERROR
.14.11. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REDIRECT_HIT          Integer    TCP_REDIRECT_HIT
============================================= ========================================= ========== ==============================================================



.. _snmp-cache-vhost-traffic-filesystem:

cache.vhost.traffic.filesystem
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.20

Provides the File I/O of the virtual host.

=========================================== =========================================== ========== ==============================================
OID                                         Name                                        Type       Description
=========================================== =========================================== ========== ==============================================
.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitRatio                             Integer    Request Hit Ratio (100%)
.2. ``[vhostMin]`` . ``[vhostIndex]``                                                              Request Hit Ratio (10,000%)
.3. ``[vhostMin]`` . ``[vhostIndex]``       byteHitRatio                                Integer    Byte Hit Ratio (100%)
.4. ``[vhostMin]`` . ``[vhostIndex]``                                                              Byte Hit Ratio (10,000%)
.5. ``[vhostMin]`` . ``[vhostIndex]``       outbound                                    Integer    The average traffic sent to File I/O (Bytes)
.6. ``[vhostMin]`` . ``[vhostIndex]``       session                                     Integer    The average number of Thread under File I/O process
.7                                          requestHitAverage                           OID        The average cache HIT results
.7.1. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_HIT                   Integer    TCP_HIT
.7.2. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_IMS_HIT               Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REFRESH_HIT           Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REF_FAIL_HIT          Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_NEGATIVE_HIT          Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_MISS                  Integer    TCP_MISS
.7.7. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REFRESH_MISS          Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_CLIENT_REFRESH_MISS   Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_DENIED                Integer    TCP_DENIED
.7.10. ``[vhostMin]`` . ``[vhostIndex]``    requestHitAverage.TCP_ERROR                 Integer    TCP_ERROR
.7.11. ``[vhostMin]`` . ``[vhostIndex]``    requestHitAverage.TCP_REDIRECT_HIT          Integer    TCP_REDIRECT_HIT
.8                                          requestHitCount                             OID        The number of cache HIT results
.8.1. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_HIT                     Integer    TCP_HIT
.8.2. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_IMS_HIT                 Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REFRESH_HIT             Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REF_FAIL_HIT            Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_NEGATIVE_HIT            Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_MISS                    Integer    TCP_MISS
.8.7. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REFRESH_MISS            Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_CLIENT_REFRESH_MISS     Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_DENIED                  Integer    TCP_DENIED
.8.10. ``[vhostMin]`` . ``[vhostIndex]``    requestHitCount.TCP_ERROR                   Integer    TCP_ERROR
.8.11. ``[vhostMin]`` . ``[vhostIndex]``    requestHitCount.TCP_REDIRECT_HIT            Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]`` . ``[vhostIndex]``      getattr.filecount                           Integer    (getattr function call) The number of FILE responses
.11. ``[vhostMin]`` . ``[vhostIndex]``      getattr.dircount                            Integer    (getattr function call) The number of DIR responses
.12. ``[vhostMin]`` . ``[vhostIndex]``      getattr.failcount                           Integer    (getattr function call) The number of failure responses
.13. ``[vhostMin]`` . ``[vhostIndex]``      getattr.timeres                             Integer    (getattr function call) Response time (0.01ms)
.14. ``[vhostMin]`` . ``[vhostIndex]``      open.count                                  Integer    The number of open function calls
.15. ``[vhostMin]`` . ``[vhostIndex]``      open.timeres                                Integer    The response time of open function (0.01ms)
.16. ``[vhostMin]`` . ``[vhostIndex]``      read.count                                  Integer    The number of read function calls
.17. ``[vhostMin]`` . ``[vhostIndex]``      read.timeres                                Integer    The response time of read function (0.01ms)
.18. ``[vhostMin]`` . ``[vhostIndex]``      read.buffersize                             Integer    The size of buffer requested by the read function (Bytes)
.19. ``[vhostMin]`` . ``[vhostIndex]``      read.bufferfilled                           Integer    The amount of filled space of the buffer requested by the read function (Bytes)
=========================================== =========================================== ========== ==============================================


.. _snmp-cache-vhost-traffic-dims:                                                                                                         
                                                                                                                                                 
cache.vhost.traffic.dims                                                                                                                
---------------------                                                                                                                            
                                                                                                                                                 
::                                                                                                                                               
                                                                                                                                                 
   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.21
                                                                                                                                                 
Provides DIMS statistics of the virtual host.                                                                                                                        
                     
=========================================== =========================================== ========== ==============================================
OID                                         Name                                        Type       Description                                   
=========================================== =========================================== ========== ==============================================
.1. ``[vhostMin]`` . ``[vhostIndex]``       requests                                    Integer    DIMS conversion requests                          
.2. ``[vhostMin]`` . ``[vhostIndex]``       converted                                   Integer    succesful conversions                               
.3. ``[vhostMin]`` . ``[vhostIndex]``       failed                                      Integer    failed conversions                               
.4. ``[vhostMin]`` . ``[vhostIndex]``       avgsrcsize                                  Integer    origin image average size (Bytes)                 
.5. ``[vhostMin]`` . ``[vhostIndex]``       avgdestsize                                 Integer    converted image average size (Bytes)                
.6. ``[vhostMin]`` . ``[vhostIndex]``       avgtime                                     Integer    conversion time (ms)          
=========================================== =========================================== ========== ==============================================




.. _snmp-cache-view:

cache.view
====================================

::

   OID = 1.3.6.1.4.1.40001.1.4.11.1

Provides identical information to the virtual host stats. 
The ``[viewIndex]`` value starts at 1 and reaches up to the number of View.

- 1.3.6.1.4.1.40001.1.4.3 - Virtual host stats

- 1.3.6.1.4.1.40001.1.4.11 - View stats
