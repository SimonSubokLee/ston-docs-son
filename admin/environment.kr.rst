.. _env:

Chapter 3. Configurations
******************

This chapter will explain about configuration structure and how to apply modified settings.
Clear understanding of configuration structure helps to arrange servers and to deal with server failing situations.

Configuration is divided into global configuration(server.xml) and virtual host configuration(vhosts.xml).

   .. figure:: img/conf_files.png
      :align: center

      There are only 2 xml files in configuration structure.

Two xml files configures almost all services.
Several txt files contain exception conditions for every virtual host, and particular function lists.
The following is the complete form of XML example. ::

   <Server>
       <VHostDefault>
           <Options>
               <CaseSensitive>ON</CaseSensitive>
           </Options>
       </VHostDefault>
   </Server>

In this manaul, simplified form will be used when explaining functions in XML file. ::

   # server.xml - <Server><VHostDefault><Options>
   
   <CaseSensitive>ON</CaseSensitive>


.. note:
   
   license.xml is not a configuration file.
   
   
.. _api-conf-reload:

Setting Reload
====================================

After changing setting, adminstrator should precisely call API. 
Other than system and performance related setting, most settings can be applied without suspending service. ::

   http://127.0.0.1:10040/conf/reload

Whenever modified setting is applied, changes are recorded in :ref:`admin-log-info`.


server.xml Global Settings
====================================

server.xml in the execution file directory is the global configuration file. 
It is an XML format text file. ::

    # server.xml
    
    <Server>
        <Host> ... </Host>
        <Cache> ... </Cache>
        <VHostDefault> ... </VHostDefault>
    </Server>
    
This section will cover structure of global configuration and simple functions.
:ref:`access-control` or :ref:`snmp` is located in the global configuration, but these major functions will be elaborated in respective chapters. 

.. toctree::
   :maxdepth: 2


.. _env-host:

Administrator Settings
------------------------------------

Configure settings for administrator. ::

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
    Configure the server name. 
    If left blank, system name will be used.
    
-  ``<Admin>``
    Configure administrator's information(email address or name). 
    This itme is only used for SNMP inquiry.
    
-  ``<Manager>``
    Configure manager port and ACL(Access Control List) for administrative purpose. 
    ACL supports IP, IP range, BitMask, Subnet information. 
    If the IP address of connected session is not authorized from the ``<Allow>`` list, the server will block the connection. 
    The IP that calls API must be configured in the ``<Allow>`` list.
    
    Based on the access condition, access authorization(Role) can be configured. 
    Any requests without authorization will be responsed with **401 Unauthorized**. 
    If ``Role`` properties are not clearly declared in ``<Allow>`` conditions, the ``Role`` property from ``<Manager>`` tag will be applied.
    
    - ``Admin`` can call entire API.
    - ``User`` only can call :ref:`api_monitoring` , :ref:`api-graph` API.
    - ``Looker`` only can call :ref:`api-graph` API.
    
    In addition, there are minor administrative properties as followings.
    
    - ``HttpMethod``
    
      - ``ON (default)`` :ref:`api-etc-httpmethod` checks ACL when the API is called.
      
      - ``OFF`` :ref:`api-etc-httpmethod` doesn't check ACL when the API is called.
    
    - ``UploadMultipartName`` configures variable names of :ref:`api-conf-upload`.


.. _env-cache-storage:

Storage Settings
------------------------------------

This section configures the storage that will preserve cached contents.
If there is enough space in the storage, all contents can be cached. 
Storage configuration is the most important setting among caching service. ::

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
    If any disk operation is repeatedly failing for ``DiskFailCount (default: 10)`` times within ``DiskFailSec (default: 60)`` seconds, then relevant disk is excluded from the service.
    The status of excluded disk is set to "Invalid". 
    
    If all disks are excluded, the server will operate according to the ``OnCrash`` property.
    
    - ``hang (default)`` option will reactivate all excluded disks. 
      This option is more likely to protect origin server rather than running a normal service.
      
    - ``bypass`` option bypasses all request to the origin server. 
      As soon as disks are recovered, STON processes service immediately.
      디스크가 복구되면 즉시 STON이 (어떠한 서비스를 처리하는지? 서비스를 위한 요청을 처리하는지?)서비스를 처리한다.
      
    - ``selfkill`` quits STON.
    
Maximum caching capacity for all disks can be configured with ``Quota (unit: GB)`` option.
Even if the ``Quota`` option is not specified, LRU(Least Recently Used) algorithm automatically discards stale contents to make sure the disk is always available.

When configuring a storage, the most important thing to consider is the number of file to keep in the storage.
As the the number of file increases, I/O performance of the disk rapidly decreases and causes poor service quality.
The maximum number of file can be configured in the ``FileMaxCount (default: Disk * 200 million)`` of ``<Storage>`` tag in order to construct desired service and quality and structure.
The following configuration example is caching 100 million contents with 5 disks.
최대 파일개수를 ``<Storage>`` 의 ``FileMaxCount (기본: Disk * 200백만--> 2백만인가요??)`` 속성으로 설정하여 원하는 서비스 품질과 형태를 구성할 수 있다.

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

Configure maximum available memory and BodyRatio(the ratio of loaded data on the memory to disk data). ::
사용할 최대 메모리와 BodyRatio(파일(디스크에 저장돼있는 파일??)로부터 메모리에 적재된 데이터의 비율)를 설정한다. ::

    # server.xml - <Server>
    
    <Cache>
        <SystemMemoryRatio>100</SystemMemoryRatio>
        <BodyRatio>50</BodyRatio>
    </Cache>
    
-  ``<SystemMemoryRatio> (default: 100%)``

   Configure the ratio of allocated memory for STON.
   For instance, if this property is set to 50(%) in the equipment with 16GB memory, the server runs as if it has 8GB system memory.
   This option is especially handy when connected with other process by using :ref:`filesystem`

-  ``<BodyRatio> (default: 50%)``

   STON improves service quality by caching as many as Body data from disk to memory.
   This ratio can be optimized to enrich quality of service according to the service type.

      .. figure:: img/bodyratio1.png
         :align: center
   
         Allocated system memory ratio can be configured with BodyRatio option.
         
   In case of game contents, the number of file is not many, but the size of contents is large.
   These services usually have a significant File I/O burden.
   Using higher value for ``<BodyRatio>`` enables more contents data to reside in the memory.

      .. figure:: img/bodyratio2.png
         :align: center
   
         I/O frequency will be decreased if BodyRatio value is increased.

    
    
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
    Most optimization consists of disk cleanup that causes I/O load.
    In order to prevent service quality degradation, optimization is gradually performed.
    하루에 한 번 시스템 최적화를 수행한다. 
    최적화의 대부분은 디스크정리 작업으로 I/O 부하가 발생한다.    
    서비스 품질저하를 방지하기 위해 최적화는 조금씩 점진적으로 수행된다.

    - ``<Time> (default: AM 2)`` This option sets the cleanup execution time. 24-hour format is used, for instance, 11:10 pm is be written as 23:10.
    
    - ``<Age> (default: 0, unit: day)`` If this value is greater than 0, contents that have not been accessed during the period are discarded.
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
    
    An Identical HASH value stands for identical configurations.
    :ref:`api-conf-restore` 가 호출되도 새로운 설정으로 저장된다(restore가 호출되면 현재의 설정이 새로운 설정으로 압축돼서 저장된다는 뜻인가요??). 
    Backup configuration file is only available for set amount of day from the time of Cleanup. 
    Configuration file can be saved for unlimited time.

    
    
Force Cleanup
------------------------------------

Execute cleanup by calling an API. ``<Age>`` parameter can be attached. ::

   http://127.0.0.1:10040/command/cleanup
   http://127.0.0.1:10040/command/cleanup?age=10
   
Cleanup is executed when there is insufficient disk space if ``<Age>`` parameter is set to 0.
If ``<Age>`` parameter is greater than 0, contents that have not been accessed during the period are discarded.
    
    
.. _env-vhostdefault:
    
Virtual Host Default Setting
------------------------------------

Administrator can configure each virtual host with different settings.
However, it is exhausting to repeat an identical setting for every additional virtual host.
All virtual hosts inherite ``<VHostDefault>``.

   .. figure:: img/vhostdefault.png
      :align: center
   
      And this is a single inheritance.

In the above figure, www.example.com does not overriding any value, hence it has A=1, B=2. 
On the contrary, img.example.com overrides value of B, therefore it has A=1, B=3.
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
    
For example, :ref:`media` function locates under the ``<Media>`` tag.


.. _env-vhost:

vhosts.xml Virtual Host Settings
====================================

The vhosts.xml file located in the execution file directory is recognized as a virtual host configuration file.
There could be unlimited number of virtual hosts. ::

    # vhosts.xml
    
    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="vod.example.com"> ... </Vhost>
    </Vhosts>
    
    
.. _env-vhost-create-destroy:
    
Create/Remove Virtual Host
------------------------------------

Virtual host ``<Vhost>`` can be configured under the ``<Vhosts>``. ::

    # vhosts.xml - <Vhosts>
    
    <Vhost Status="Active" Name="www.example.com">
        <Origin>
            <Address>10.10.10.10</Address>
        </Origin>
    </Vhost>

-  ``<Vhost>`` configures virtual host.
    
   - ``Status (default: Active)`` Inactive value stops virtual host service. However, cached contents are kept.
   - ``Name`` is a name of virtual host. An identical name cannot be used for different virtual hosts.
    
In order to remove a specific virtual host, delete the relavant ``<Vhost>`` tag. 
All contents in removed virtual hosts are subject to removal. 
Recreating removed virtual host doesn't recover deleted contents.


.. _env-vhost-find:
    
Discovering Virtual Host(검색. 가상호스트를 검색할 수 있도록 하는 설정을 뜻하는거죠?)
------------------------------------

The following is the simplest form of HTTP request. ::

    GET / HTTP/1.1
    Host: www.example.com

General Web servers find virtual host with Host header.
If one virtual host needs to be serviced with different names, use ``<Alias>`` option. ::

    # vhosts.xml - <Vhosts>
    
    <Vhost Name="www.example.com">
        <Alias>www2.example.com</Alias>
        <Alias>*.sub.example.com</Alias>
    </Vhost>

-  ``<Alias>``

   This option configures alias of the virtual host.
   Unlimited number of alias could be created with this option.
   Both specific domain name such as www2.example.com and 패턴표현?? are supported for alias.
   개수는 제한이 없다.
   명확한 표현(www2.example.com)과 패턴표현(*.sub.example.com)을 지원한다.
   패턴은 복잡한 정규표현식이 아닌 prefix에 * 표현을 하나만 붙일 수 있는 간단한 형식만을 지원한다.


가상호스트 검색 순서는 다음과 같다.

1. ``<Vhost>`` 의 ``Name`` 과 일치하는가?
2. 명시적인 ``<Alias>`` 와 일치하는가?
3. 패턴 ``<Alias>`` 를 만족하는가?


.. _env-vhost-defaultvhost:    
    
Default 가상호스트
------------------------------------

요청을 처리할 가상호스트를 찾지못한 경우 선택될 가상호스트를 지정할 수 있다. 
요청을 처리하고 싶지 않다면 설정하지 않아도 된다. ::

    # vhosts.xml
    
    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Default>www.example.com</Default>
    </Vhosts>

-  ``<Default>``

   기본 가상호스트 이름을 설정한다. 
   반드시 ``<Vhost>`` 의 ``Name`` 속성과 똑같은 문자열로 설정해야 한다.


.. _env-vhost-listen:
    
서비스주소
------------------------------------
서비스 주소를 설정한다. ::

    # vhosts.xml - <Vhosts>
    
    <Vhost Name="www.example.com">
        <Listen>*:80</Listen>
    </Vhost>
    
-  ``<Listen> (기본: *:80)``

   {IP}:{Port} 형식으로 서비스 주소를 설정한다.
   *:80 표현은 모든 NIC로부터의 80포트로 오는 요청을 처리한다는 의미다.
   예를 들어 특정 IP(1.1.1.1)의 90포트로 서비스하고 싶다면 다음과 같이 설정한다. ::
   
       # vhosts.xml - <Vhosts>
       
       <Vhost Name="www.example.com">
           <Listen>1.1.1.1:90</Listen>
       </Vhost>
    
.. note:

   서비스 포트를 열지 않으려면 ``OFF`` 로 설정한다. ::
   
      # vhosts.xml - <Vhosts> 
      
      <Vhost Name="www.example.com">
         <Listen>OFF</Listen>
      </Vhost>
    

.. _env-vhost-txt:

가상호스트-예외조건 (.txt)
---------------------------------------

서비스 중 다음과 같이 예외적인 상황이 필요할 때가 있다.

- 모든 POST요청은 허용하지 않지만, 특정 URL에 대한 POST요청은 허가한다.
- 모든 GET요청은 STON이 응답하지만, 특정 IP대역에 대해서는 원본서버로 바이패스한다.
- 특정 국가에 대해서는 전송속도를 제한한다.

이와같은 예외조건은 XML에 설정하지 않는다. 
모든 가상호스트는 독립적인 예외조건을 가진다.
예외조건은 ./svc/가상호스트/ 디렉토리 하위에 TXT로 존재한다.
관련 기능에 대해 설명할 때 예외조건도 함께 다룬다.


가상호스트 목록확인
====================================

가상호스트 목록을 조회한다. ::

   http://127.0.0.1:10040/monitoring/vhostslist
   
결과는 JSON형식으로 제공된다. ::

   {
      "version": "1.1.9",
      "method": "vhostslist",
      "status": "OK",
      "result": [ "www.example.com","www.foobar.com", "site1.com" ] 
   }


.. _api-conf-show:

설정 확인
====================================

서비스 중인 설정파일을 확인한다. 
txt파일들은 가상호스트(vhost)를 명확하게 지정해주어야 한다. ::

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

설정 목록
====================================

백업된 설정목록을 열람한다. ::

    http://127.0.0.1:10040/conf/latest
    http://127.0.0.1:10040/conf/history
    
결과는 JSON 형식으로 제공된다. 
빠르게 마지막 설정상태만 확인하고 싶은 경우는 /conf/latest를 사용할 것을 권장한다. ::

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
    
-  ``id`` 설정의 고유 아이디 (Reload할때마다 +1)
-  ``conf-date`` 설정 변경날짜
-  ``conf-time`` 설정 변경시간
-  ``type`` 설정이 반영된 형태
   - ``loaded`` STON이 시작될 때
   - ``modified`` 설정이 (관리자 또는 WM에 의해) 변경될 때
   - ``uploaded`` 설정파일 API를 통해 업로드 되었을 때
   - ``restored`` 설정파일이 API를 통해 복구되었을 때
-  ``size`` 설정파일 크기
-  ``hash`` 설정파일을 SHA-1으로 hash한 값


.. _api-conf-restore:

설정 복구
====================================

hash값 또는 id를 기준으로 원하는 시점의 설정으로 되돌린다. 
hash와 id가 모두 명시된 경우 hash값이 우선한다. 
정상적으로 Rollback된 경우 200 OK, 실패한 경우 500 Internal Error로 응답한다. ::

    http://127.0.0.1:10040/conf/restore?hash=...
    http://127.0.0.1:10040/conf/restore?id=...


.. _api-conf-download:
    
설정 다운로드
====================================

hash값 또는 id를 기준으로 원하는 시점의 설정을 다운로드 한다. 
Content-Type은 "application/x-compressed"로 명시된다. 
hash와 id가 모두 명시된 경우 hash값이 우선하며, 해당 시점의 설정이 존재하지 않는 경우 
404 NOT FOUND로 응답한다. ::

    http://127.0.0.1:10040/conf/download?hash=...
    http://127.0.0.1:10040/conf/download?id=...

.. _api-conf-upload:

설정 업로드
====================================

설정파일을 HTTP Post방식(Multipart 지원)으로 업로드 한다. ::

    http://127.0.0.1:10040/conf/upload

다음과 같이 주소, Content-Length, Content-Type(="multipart/form-data")이 명확하게 선언되어 있어야 한다. ::

    POST /conf/upload
    Content-Length: 16455
    Content-Type: multipart/form-data; boundary=......
    
업로드가 완료되면 압축을 해지한 뒤 즉시 반영시킨다.

Multipart방식에서는 "confile"을 기본 이름으로 사용한다. 
이 값은 ``<Manager>`` 의 ``UploadMultipartName`` 속성에서 설정할 수 있다. ::

    <form enctype="multipart/form-data" action="http://127.0.0.1:10040/conf/upload" method="POST">
        <input name="confile" type="file" />
        <input type="submit" value="Upload" />
    </form>
    
    
