.. _access-control:

Chapter 10. Access Control
******************

This chapter explains how to deny unwanted client accesses.
Normally ACL(Access Control List) saves block list for access deny, but for the convenience of configuration, allowed list might be used.

Access control is divided into server access control that is excuted during connection stage and virtual host access control that is configured for each virtual host.
You will have to decide efficient time to block access.
All access control is logged.

.. toctree::
   :maxdepth: 2


.. _access-control-serviceaccess:

Server Access Control
====================================

At the moment a client accesses to the server, the server dicides whether to block the connection or not based on the IP address.
This occurs during the connection stage, it is the most certain and quick method.
This is configured in the global setting(server.xml) and has the highest priority. ::

   # server.xml - <Server><Host>

   <ServiceAccess Default="Allow">
      <Deny>192.168.7.9-255</Deny>
      <Deny>192.168.8.10/255.255.255.0</Deny>
   </ServiceAccess>

-  ``<ServiceAccess>``   
   This tag configures ACL based on IP and supports 4 formats(IP, IP Range, Bitmask, Subnet).
   This is sequence sensitive and the configuration on top has priority. 
   ``Default (default: Allow)`` property is adopted when there is no matching condition.
   If you set this property to ``Deny``, you should specify allowed conditions in ``<Allow>``.
   
Blocked IP is logged at :ref:`admin-log-deny`.


.. _access-control-geoip:

GeoIP
====================================

GeoIP can be used to deny accesses from pre-populated regional selection. 
Binary Databases of `GeoIP Databases <http://dev.maxmind.com/geoip/legacy/downloadable/>`_
is linked to `GEOIP_MEMORY_CACHE and GEOIP_CHECK_CACHE <http://dev.maxmind.com/geoip/legacy/benchmarks/>`_ to apply changes in real time. ::

   # server.xml - <Server><Host>

   <ServiceAccess GeoIP="/var/ston/geoip/">
      <Deny>AP</Deny>
      <Deny>GIN</Deny>
   </ServiceAccess>
    
GeoIP Databases path is configured in the ``GeoIP`` property of ``<ServiceAccess>``.
`ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ and 
`ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ country codes are supported.

.. note::

   GeoIP has a reserved file name, saved local directory must be used. 
   Also, changes are automatically reflected to the service, there is no need to reload configuraion.
   
   
If GeoIP is configured, check the files in the related directory. 
If GeoIP is not configured, 404 NOT FOUND is returned. ::
   
   http://127.0.0.1:10040/monitoring/geoiplist
   
The result is returned in JSON format. ::

   {
       "version": "2.0.0",
       "method": "geoiplist",
       "status": "OK",
       "result":
       {
           "path" : "/usr/ston/geoip/",
           "files" :
           [
               {
                   "file" : "GeoIP.dat", 
                   "size" : 766255
               },
               {
                   "file" : "GeoLiteCity.dat", 
                   "size" : 12826936
               }
           ]
       }
   }
   

.. _access-control-vhost:

Virtual Host Access Control
====================================

Each virtual hosts control accesses.
When a client sends HTTP request, virtual hosts decide whether to deny access or not.
Because if HTTP request is not sent, virtual host cannot be found. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <AccessControl Default="Allow" DenialCode="401">OFF</AccessControl>   
    
-  ``<AccessControl>``
   
   - ``OFF (default)`` ACL is inactivated. All client requests are allowed.
    
   - ``ON`` ACL is activated. 
     차단된 요청에 대해서는 ``DenialCode`` 속성에 설정된 응답코드로 응답한다.
     ``Default (기본: Allow)`` 속성이 ``Allow`` 라면 ACL은 거부목록이 된다. 
     반대로 ``Deny`` 라면 ACL은 허가목록이 된다.
     
Deny된 요청은 :ref:`admin-log-access` 에 TCP_DENY로 기록된다.


.. _access-control-vhost_acl:

가상호스트 ACL
---------------------

모든 클라이언트 HTTP요청에 대하여 허용/거부/Redirect 여부를 판단한다.
각 조건마다 별도로 응답코드를 설정할 수도 있다.
Redirect된 요청에 대해서는 **302 Moved temporarily** 로 응답한다. 
ACL은 /svc/{가상호스트 이름}/acl.txt에 설정한다. ::

   # /svc/www.example.com/acl.txt
   # 구분자는 콤마(,)이며 {조건},{키워드 = allow | deny | redirect} 순서로 표기한다.
   # deny일 경우 키워드 뒤에 응답코드를 명시할 수 있다.
   # 명시하지 않으면 ``<AccessControl>`` 의 ``DenialCode`` 를 사용한다.
   # redirect일 경우 키워드 뒤에 이동시킬 URL을 명시한다. (Location헤더의 값으로 명시)
   # n 개의 조건을 결합(AND)하기 위해서는 &를 사용한다.
   
   $IP[192.168.1.1], allow
   $IP[192.168.2.1-255]
   $IP[192.168.3.0/24], deny
   $IP[192.168.4.0/255.255.255.0]
   $IP[AP] & !HEADER[referer], allow
   $IP[GIN], redirect, /page/illegal_access.html
   $HEADER[cookie: *ILLEGAL*], deny, 404
   $HEADER[via: Apache]
   $HEADER[x-custom-header]
   $HEADER[referer:], redirect, http://another-site.com
   !HEADER[referer] & !HEADER[user-agent] & !HEADER[host], deny
   $URL[/source/public.zip], allow
   $URL[/source/*]
   /profile.zip, deny, 500
   /secure/*.dat
   
조건은 IP, GeoIP, Header, URL 4가지로 설정이 가능하다.

-  **IP**
   $IP[...]로 표기하며 IP, IP Range, Bitmask, Subnet 네 가지 형식을 지원한다.

-  **GeoIP**
   $IP[...]로 표기하며 반드시 GeoIP설정이 되어 있어야 동작한다. 
     
-  **Header**
   $HEADER[Key : Value]로 표기한다. 
   Value는 명확한 표현과 패턴을 인식한다. 
   $HEADER[Key:]처럼 구분자는 있지만 Value가 빈 문자열이라면 요청 헤더의 값이 비어 있는 경우를 의미한다. 
   $HEADER[Key]처럼 구분자 없이 Key만 명시되어 있다면 Key에 해당하는 헤더의 존재유무를 조건으로 판단한다.
    
-  **URL**
   $URL[...]로 표기하며 생략이 가능합니다. 명확한 표현과 패턴을 인식합니다.
    
$는 "조건에 맞다면 ~ 한다"를 의미하지만 !는 "조건에 맞지 않는다면 ~ 한다"를 의미한다. 
다음과 같이 부정조건으로 지원한다. ::

   # 국가가 KOR이 아니라면 deny한다.
   !IP[KOR], deny
    
   # referer헤더가 존재하지 않는다면 deny한다.
   !HEADER[referer], deny
   
   # /secure/ 경로 하위가 아니라면 allow한다.
   !URL[/secure/*], allow

