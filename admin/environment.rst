.. _env:

Chapter 2. Configuration
******************

This chapter is about configuration of STON Edge Server.
Understanding the configuration structure helps faster setting and troubleshooting.

STON Edge Server configuration consists of global configuration(server.xml) and virtual host configuration(vhosts.xml).

   .. figure:: img/conf_files.png
      :align: center

      Two XML files. That is all.

TXT configuration files contain exception handling for each virtual host, and particular function lists.
The following is the complete form of XML example. ::

   <Server>
       <VHostDefault>
           <Options>
               <CaseSensitive>ON</CaseSensitive>
           </Options>
       </VHostDefault>
   </Server>

A simplified format as below is to indicate the function from XML configuration structure. ::

   # server.xml - <Server><VHostDefault><Options>
   
   <CaseSensitive>ON</CaseSensitive>


.. note:
   
   license.xml is not a configuration file.
   
   
.. _api-conf-reload:

Reload settings
====================================

Setting changes are effective immediately after the reload API is called.
Service operation runs uncluttered, except for system and performance settings. ::

   http://127.0.0.1:10040/conf/reload

Any changes from settings are recorded in :ref:`admin-log-info`.

(proofread up to here)

Global Settings (server.xml)
====================================

server.xml is the global configuration file, which may be found in the execution file directory.
It is an XML format text file. ::

    # server.xml
    
    <Server>
        <Host> ... </Host>
        <Cache> ... </Cache>
        <VHostDefault> ... </VHostDefault>
    </Server>
    
This chapter covers the configuration structure and fundamental functionality.
Details on :ref:`access-control` or :ref:`snmp` are described in respective chapters. 

.. toctree::
   :maxdepth: 2


.. _env-host:

Administrator Settings
------------------------------------

The following explains how to configure administrator settings. ::

    # server.xml - <Server>
    
    <Host>
        <Name>stream_07</Name>
        <Admin>admin@example.com</Admin>
        <Manager Port="10040" HttpMethod="ON" Role="Admin" UploadMultipartName="confile">
            <Allow>192.168.1.1</Allow>
            <Allow Role="Admin">192.168.2.1-255</Allow>
            <Allow Role="User">192.168.3.0/24</Allow>
            <Allow Role="Looker">192.168.4.0/255.255.255.0</Allow>
        </Manager>
    </Host>

-  ``<Name>``
    Configure the server name. If left blank, the system name will be used.
    
-  ``<Admin>``
    Configure the administrator's information(email address or name). This item is only used for SNMP inquiry.
    
-  ``<Manager>``
    Configure the manager port and an ACL(Access Control List) for administrative purpose. 
    An ACL supports the IP, IP range, BitMask, and Subnet information. 
    If the IP address of the connected session is not authorized from the ``<Allow>`` list, the server will block the connection. 
    The IP that calls an API must be configured in the ``<Allow>`` list.
    
    Based on this access condition, an access authorization(Role) can be configured. 
    Any requests without authorization will be responded to **401 Unauthorized**. 
    If ``Role`` properties are not clearly declared in ``<Allow>`` conditions, the ``Role`` property from the ``<Manager>`` tag will be applied.
    
    - ``Admin`` can call the entire API.
    - ``User`` can only call :ref:`api_monitoring` , :ref:`api-graph` APIs.
    - ``Looker`` can only call the :ref:`api-graph` API.
    
    In addition, there are minor administrative properties, as follows.
    
    - ``HttpMethod``
    
      - ``ON (default)`` :ref:`api-etc-httpmethod` checks ACL when the API is called.
      
      - ``OFF`` :ref:`api-etc-httpmethod` doesn't do an ACL check when the API is called.
    
    - ``UploadMultipartName`` configures variable names of :ref:`api-conf-upload`.


.. _env-cache-storage:

Storage Settings
------------------------------------

This section explains how to configure the storage that will preserve cached contents.
If there is enough storage space, all content can be cached. 
Storage configuration is the most important setting among caching services. ::

    # server.xml - <Server>
    
    <Cache>
        <Storage DiskFailSec="60" DiskFailCount="10" OnCrash="hang">
            <Disk>/user/cache1</Disk>    
            <Disk>/user/cache2</Disk>    
            <Disk Quota="100">/user/cache3</Disk>
        </Storage>
    </Cache>
    
-  ``<Storage>``
    Configure the disk to store contents. 
    The number of subordinate ``<Disk>`` is not limited.
    
    Since disks are the most problematic equipment, it is recommended to set specific fail conditions.
    If any disk operation is repeatedly failing for ``DiskFailCount (default: 10)`` times within ``DiskFailSec (default: 60)`` seconds, then a relevant disk is excluded from the service.
    The status of the excluded disk is set to "Invalid". 
    
    If all disks are excluded, the server will operate according to the ``OnCrash`` property.
    
    - ``hang (default)`` option will reactivate all excluded disks. 
      This option is more likely to protect the origin server rather than running a normal service.
      
    - ``bypass`` option bypasses all request to the origin server. 
      As soon as the disks are recovered, STON reactivates the caching service immediately.
      
    - ``selfkill`` quits STON.
    
The maximum caching capacity for all disks can be configured with a ``Quota (unit: GB)`` option.
Even if the ``Quota`` option is not specified, the LRU(Least Recently Used) algorithm automatically discards stale contents to make sure the disk is always available.

When configuring a storage, the most important thing to consider is the desired number of files to store.
As the the number of file increases, the I/O performance of the disk rapidly decreases and causes poor service quality.
The maximum number of files can be configured in the ``FileMaxCount (default: Disk * 2 million)`` of the``<Storage>`` tag in order to construct the desired service, quality and structure.
The following configuration is an example of caching 100 million contents with 5 disks.

    # server.xml - <Server>
    
    <Cache>
        <Storage FileMaxCount="100000000">
            <Disk>/user/cache1</Disk>
            <Disk>/user/cache2</Disk>
            <Disk>/user/cache3</Disk>
            <Disk>/user/cache4</Disk>
            <Disk>/user/cache5</Disk>
        </Storage>
    </Cache>
    
    

.. _env-cache-resource:

Memory Restriction
------------------------------------

Configure the maximum available memory and BodyRatio(the ratio of loaded data into the memory to data in the disk). ::

    # server.xml - <Server>
    
    <Cache>
        <SystemMemoryRatio>100</SystemMemoryRatio>
        <BodyRatio>50</BodyRatio>
    </Cache>
    
-  ``<SystemMemoryRatio> (default: 100%)``

   Configure the ratio of allocated memory for STON.
   For instance, when this property is set to 50(%) in the equipment with 16GB of memory, the server will run as if it has 8GB of system memory.
   This option is especially handy when connected with another process by using :ref:`filesystem`

-  ``<BodyRatio> (default: 50%)``

   STON improves service quality by caching as much Body data as necessary from disk into memory.
   This ratio can be optimized to enrich the quality of the service according to the service type.

      .. figure:: img/bodyratio1.png
         :align: center
   
         The allocated system memory ratio can be configured with the BodyRatio option.
         
   In the case of game content, while the amount of content is large, there are not many files.
   These services usually have a significant File I/O burden.
   Using a higher value for the ``<BodyRatio>`` enables more content data to reside in the memory.

      .. figure:: img/bodyratio2.png
         :align: center
   
         I/O frequency will be decreased if the BodyRatio value is increased.

    
    
Other Caching Settings
------------------------------------

This section configures other caching service options. ::

    # server.xml - <Server>
    
    <Cache>
        <Cleanup>
            <Time>02:00</Time>
            <Age>0</Age>
        </Cleanup>     
        <Listen>0.0.0.0</Listen>        
        <ConfigHistory>30</ConfigHistory>
    </Cache>

-  ``<Cleanup>``
    The server runs the system optimization once a day.
    Most optimization procedures include a disk cleanup application cause I/O load.
    In order to prevent service quality degradation, optimization is gradually performed.

    - ``<Time> (default: AM 2)`` This option sets the cleanup execution time. When a 24-hour format is used, for instance, 11:10 pm should be written as 23:10.
    
    - ``<Age> (default: 0, unit: day)`` If this value is greater than 0, content that has not been accessed during the period is discarded.
      This option helps reducing the possibility of insufficient disk space during the service by securing available space on the disk.

-  ``<Listen>``
    Assign IP lists for all virtual hosts to Listen. 
    The default Listen setting of *:80 for all virtual hosts stands for 0.0.0.0:80. 
    The following is a configuration example of enabling particular IP addresses. ::

       # server.xml - <Server>
       
       <Cache>
         <Listen>10.10.10.10</Listen>
         <Listen>10.10.10.11</Listen>
         <Listen>127.0.0.2</Listen>
       </Cache>    

-  ``<ConfigHistory> (default: 30 days)``
    STON backs up all configurations when there is a change in the setting. 
    Configuration files are compressed into one file and saved at ./conf/.
    "date_time_HASH.tgz" naming format is used for the compressed file. ::
    
       20130910_174843_D62CA26F16FE7C66F81D215D8C52266AB70AA5C8.tgz
    
    An identical HASH value stands for identical configurations.
    Even if :ref:`api-conf-restore` is called, the restored configuration will be saved as a new configuration.
    A backup configuration file is only available for a set amount of day from the time of Cleanup. 
    A configuration file can be saved for an unlimited amount of time.

    
    
Force Cleanup
------------------------------------

Execute cleanup by calling an API. An ``<Age>`` parameter can be attached. ::

   http://127.0.0.1:10040/command/cleanup
   http://127.0.0.1:10040/command/cleanup?age=10
   
Cleanup is executed when there is insufficient disk space if the ``<Age>`` parameter is set to 0.
If the ``<Age>`` parameter is greater than 0, content that has not been accessed during the period is discarded.
    
    
.. _env-vhostdefault:
    
Virtual Host Default Setting
------------------------------------

Administrators can configure each virtual host with different settings.
However, it is exhausting to repeat an identical setting for every additional virtual host.
All virtual hosts inherit ``<VHostDefault>``.

   .. figure:: img/vhostdefault.png
      :align: center
   
      This is a single inheritance.

In the figure above, www.example.com does not override any value; hence, it has the value of A=1 and B=2. 
On the contrary, img.example.com overrides the value of B; therefore it has A=1 and B=3.
Administrators normally keep services that have similar service attributes in one server.
For this reason, inheritance of the default setting is enormously effective.

``<VHostDefault>`` consists of function-based 5 subordinate tags. ::

    # server.xml - <Server>
    
    <VHostDefault>
        <Options> ... </Options>  
        <OriginOptions> ... </OriginOptions>  
        <Media> ... </Media>  
        <Stats> ... </Stats>  
        <Log> ... </Log>
    </VHostDefault>
    
For example, the :ref:`media` function's location is under the ``<Media>`` tag.


.. _env-vhost:

Virtual Host Settings (vhosts.xml)
====================================

The vhosts.xml file located in the execution file directory is recognized as a virtual host configuration file.
There can be an unlimited number of virtual hosts. ::

    # vhosts.xml
    
    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="vod.example.com"> ... </Vhost>
    </Vhosts>
    
    
.. _env-vhost-create-destroy:
    
Create/Remove Virtual Host
------------------------------------

Virtual host ``<Vhost>`` can be configured under ``<Vhosts>``. ::

    # vhosts.xml - <Vhosts>
    
    <Vhost Status="Active" Name="www.example.com">
        <Origin>
            <Address>10.10.10.10</Address>
        </Origin>
    </Vhost>

-  ``<Vhost>`` configures the virtual host.
    
   - ``Status (default: Active)`` An inactive value stops the virtual host service. However, cached contents are kept.
   - ``Name`` is a name of a virtual host. An identical name cannot be used for different virtual hosts.
    
In order to remove a specific virtual host, delete the relevant ``<Vhost>`` tag. 
All content in removed virtual host are subject to removal. 
Recreating removed virtual hosts doesn't recover deleted contents.


.. _env-vhost-find:
    
Discovering a Virtual Host
------------------------------------

The following is the simplest form of an HTTP request. ::

    GET / HTTP/1.1
    Host: www.example.com

General Web servers find the virtual host with a Host header.
If one virtual host needs to be serviced with different names, use the ``<Alias>`` option. ::

    # vhosts.xml - <Vhosts>
    
    <Vhost Name="www.example.com">
        <Alias>www2.example.com</Alias>
        <Alias>*.sub.example.com</Alias>
    </Vhost>

-  ``<Alias>``

   This option configures an alias of the virtual host.
   An unlimited number of aliases can be created with this option.
   Aliases can be configured for both specific domain names, such as www2.example.com and patterned domain names such as *.sub.example.com.
   For patterned domain names, a simple format with an asterisk as a domain prefix is supported.


When discovering a virtual host, follow the procedures below.

1. Does the ``Name`` of the ``<Vhost>`` match?
2. Does the specific ``<Alias>`` match?
3. Does the patterned ``<Alias>`` match?


.. _env-vhost-facadevhost:    

Façade Virtualhost
------------------------------------

``<Alias>`` may not provide separate statistics and logs from the virtual host.
If separate :ref:`monitoring_stats_vhost_client` and :ref:`admin-log-access` are required, a façade virtual host is configurable, while sharing the virtual host. ::

    # vhosts.xml - <Vhosts>
    
    <Vhost Name="example.com">
       ...
    </Vhost>
    
    <Vhost Name="another.com" Status="facade:example.com">
       ...
    </Vhost>

Set ``Status`` to ``facade:`` + ``virtualhost``.
For the example, :ref:`monitoring_stats_vhost_client` and :ref:`admin-log-access` are collected in another.com requested by client, not example.com. 


.. _env-vhost-sub-path:
    
Sub-Path
------------------------------------

A single virtual host may have multiple sub-paths, which are configruable into different virtual hosts. ::

   # vhosts.xml - <Vhosts>
   
   <Vhost Name="sports.com">
     <Sub Status="Active">
       <Path Vhost="baseball.com">/baseball/<Path>
       <Path Vhost="football.com">/football/<Path>
       <Path Vhost="photo.com">/*.jpg<Path>
     </Sub>
   </Vhost>

   <Vhost Name="baseball.com" />
   <Vhost Name="football.com" />
   <Vhost Name="photo.com" />

-  Matched ``<Sub>`` paths or patterns will be directed to the configured virtual hosts.
   Unmatched ones remain as in the virtual host.
    
   - ``Status (default: Active)`` Ignored if inactive.

   -  ``<Path>`` Forwards the requests if matched to the client requeted URIs.
      In path or pattern. ::
      
         <Path Vhost="baseball.com">baseball<Path>
         <Path Vhost="photo.com">*.jpg<Path>
      
      The above inputs are recognized as /baseball/ and /*.jpg respectively.

For an example, the following client request is directed to the virtualhost football.com :: 

   GET /football/rank.html HTTP/1.1
   Host: sports.com


.. _env-vhost-defaultvhost:    
    
Default Virtual Host
------------------------------------

When a virtual host is not found for a request, the administrator can appoint a specific virtual host to respond.
If a default virtual host is not appointed, the request will be abandoned. ::

    # vhosts.xml
    
    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Default>www.example.com</Default>
    </Vhosts>

-  ``<Default>``

   Configure the default virtual host name. 
   This must be configured with an identical character array from the ``Name`` property in the ``<Vhost>`` tag.


.. _env-vhost-listen:
    
Service Address
------------------------------------
This section explains how to configure a service address. ::

    # vhosts.xml - <Vhosts>
    
    <Vhost Name="www.example.com">
        <Listen>*:80</Listen>
    </Vhost>
    
-  ``<Listen> (default: *:80)``

   The {IP}:{Port} format is used for configuring service addresses.
   The *:80 expression means that all requests that arrive at the port 80 from NIC will be processed.
   If a service should only process requests from a specific IP address(1.1.1.1) and port(90), the following setting will do ::

       # vhosts.xml - <Vhosts>
       
       <Vhost Name="www.example.com">
           <Listen>1.1.1.1:90</Listen>
       </Vhost>
    
.. note:

   In order to close the service port, set the ``Listen`` tag to ``OFF``. ::
   
      # vhosts.xml - <Vhosts> 
      
      <Vhost Name="www.example.com">
         <Listen>OFF</Listen>
      </Vhost>
    

.. _env-vhost-txt:

Virtual Host - Exceptions (.txt)
---------------------------------------

During the service, there are some cases when the following exceptions should be allowed.

- Basically all POST requests are not allowed, but a POST request from a specific URL can be allowed.
- STON responds to all GET requests, but requests from a specific IP band can be bypassed to the origin server.
- The transmission speed for specific countries can be limited.

These exceptions will not be configured in the XML. 
All virtual hosts have independent exception settings, and the settings are saved as TXT files under ./svc/virtualhost/  directory.
Exceptions will be explained more in the relevant sections.


Checking the Virtual Host List
====================================

The following command queries a virtual host list. ::

   http://127.0.0.1:10040/monitoring/vhostslist
   
The result is returned in JSON format. ::

   {
      "version": "1.1.9",
      "method": "vhostslist",
      "status": "OK",
      "result": [ "www.example.com","www.foobar.com", "site1.com" ] 
   }


.. _api-conf-show:

Confirm Configuration
====================================

The next step is to browse the service's configuration files.
Each txt file must clearly appoint virtual hosts(vhost). ::

    http://127.0.0.1:10040/conf/server.xml
    http://127.0.0.1:10040/conf/vhosts.xml
    http://127.0.0.1:10040/conf/querystring.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/bypass.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/ttl.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/expires.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/acl.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/headers.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/throttling.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/postbody.txt?vhost=www.example.com


.. _api-conf-history:

Configuration History
====================================

This section explains how to browse backup configuration histories. ::

    http://127.0.0.1:10040/conf/latest
    http://127.0.0.1:10040/conf/history
    
The result is also returned in JSON format. 
In order to quickly review the latest history, using the /conf/latest option is recommended. ::

    {
        "history" : 
        [
            {
                "id" : "5",
                "conf-date" : "2013-11-06",
                "conf-time" : "15:26:37",
                "type" : "loaded",
                "size" : "16368",
                "hash" : "D62CA26F16FE7C66F81D215D8C52266AB70AA5C8",
                "ver": "1.2.8"
            },
            {
                "id" : "6",
                "conf-date" : "2013-11-07",
                "conf-time" : "07:02:21",
                "type" : "modified",
                "size" : "27544",
                "hash" : "F81D215D8C52266AB70AA5C8D62CA26F16FE7C66",
                "ver": "1.2.8"
            }
        ]
    }
    
-  ``id`` Unique identification number (Reload will result +1)
-  ``conf-date`` Configuration modified date
-  ``conf-time`` Configuration modified time
-  ``type`` How a modified setting is applied. There are four options as below.
   - ``loaded`` When STON is loaded
   - ``modified`` When a configuration is modified (by administrator or WM)
   - ``uploaded`` When a configuration is uploaded via API
   - ``restored`` When a configuration is restored via API
-  ``size`` Size of a configuration file
-  ``hash`` Hash value of the configuration file using the SHA-1 algorithm.


.. _api-conf-restore:

Restore Configuration
====================================

This section explains how to restore a configuration of the time when a hash value or an id was created.
If both the hash value and id are stated in the command, the hash value takes priority over the id.
"200 OK" will be returned when the configuration is successfully rolled back, and "500 Internal Error" will be returned for any failures. ::

    http://127.0.0.1:10040/conf/restore?hash=...
    http://127.0.0.1:10040/conf/restore?id=...


.. _api-conf-download:
    
Configuration Download
====================================

This section explains how to download a configuration of the time when a hash value or an id is created.
The Content-Type will be specified as "application/x-compressed".
If both the hash value and the id are stated in the command, the hash value takes priority over the id.
If neither a hash value is stated nor an id is found, the command will return "404 NOT FOUND". ::

    http://127.0.0.1:10040/conf/download?hash=...
    http://127.0.0.1:10040/conf/download?id=...

.. _api-conf-upload:

Configuration Upload
====================================

The following command will upload a configuration file using the HTTP Post method (Multipart supported). ::

    http://127.0.0.1:10040/conf/upload

Both Content-Length and Content-Type(="multipart/form-data") have to be clearly stated in the command as shown below. ::

    POST /conf/upload
    Content-Length: 16455
    Content-Type: multipart/form-data; boundary=......
    
As soon as the upload is completed, the configuration file will be extracted and applied to the system right away.

In a multipart method, "confile" is used as a default name.
This name can be changed in the ``UploadMultipartName`` property of the ``<Manager>`` tag. ::

    <form enctype="multipart/form-data" action="http://127.0.0.1:10040/conf/upload" method="POST">
        <input name="confile" type="file" />
        <input type="submit" value="Upload" />
    </form>
    
    
