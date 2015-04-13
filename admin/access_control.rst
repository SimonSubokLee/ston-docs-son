.. _access-control:

Chapter 10. Access Control
******************

This chapter explains how to deny access to unwanted clients. 
Normally an ACL (Access Control List) saves a block list for denying access to specified clients, 
but an allowed list can be used for a more convenient configuration.

Access control is divided into server access control, which is executed during the connection stage, and virtual host access control, which is configured for each virtual host. 
You will have to decide on an efficient time frame to block access. 
All access control is logged.

.. toctree::
   :maxdepth: 2


.. _access-control-serviceaccess:

Server Access Control
====================================

At the moment a client accesses the server, the server decides whether to block the connection or not based on the IP address. 
This process occurs during the connection stage and is the quickest, most certain method. 
It is configured in the global setting (server.xml) and has the highest priority. ::

   # server.xml - <Server><Host>

   <ServiceAccess Default="Allow">
      <Deny>192.168.7.9-255</Deny>
      <Deny>192.168.8.10/255.255.255.0</Deny>
   </ServiceAccess>

-  ``<ServiceAccess>``   
   This tag configures an ACL based on the IP address and supports four formats(IP, IP Range, Bitmask, and Subnet).
   It is sequence sensitive and the configuration on top has priority. 
   The ``Default (default: Allow)`` property is adopted when there is no matching condition.
   If you set this property to ``Deny``, you should specify any allowed conditions in ``<Allow>``.
   
Blocked IP addresses are logged at :ref:`admin-log-deny`.


.. _access-control-geoip:

GeoIP
====================================

GeoIP can be used to deny access from pre-populated regional selection. 
Binary Databases of `GeoIP Databases <http://dev.maxmind.com/geoip/legacy/downloadable/>`_
are linked to `GEOIP_MEMORY_CACHE and GEOIP_CHECK_CACHE <http://dev.maxmind.com/geoip/legacy/benchmarks/>`_ to apply changes in real time. ::

   # server.xml - <Server><Host>

   <ServiceAccess GeoIP="/var/ston/geoip/">
      <Deny>AP</Deny>
      <Deny>GIN</Deny>
   </ServiceAccess>
    
GeoIP Databases paths are configured in the ``GeoIP`` property of ``<ServiceAccess>``.
`ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ and 
`ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ country codes are supported.

.. note::

   Since the GeoIP has a reserved file name, you have to configure to use a pre-defined path. 
   Also, because service changes are automatic, reloading the configuration is unnecessary.
   
   
If a GeoIP is configured, check the files in the related directory. 
If a GeoIP is not configured, a 404 NOT FOUND response will be returned. ::
   
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

Each virtual host controls access.
When a client sends an HTTP request, virtual hosts decide whether to deny access or not.
If an HTTP request is not sent, a virtual host cannot be found. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <AccessControl Default="Allow" DenialCode="401">OFF</AccessControl>   
    
-  ``<AccessControl>``
   
   - ``OFF (default)`` An ACL is inactivated. All client requests are allowed.
    
   - ``ON`` An ACL is activated. 
     For denied requests, the response code configured in the ``DenialCode`` property will be returned.
     ``Default (default: Allow)`` If the property is set to ``Allow``, an ACL is used as a block list. 
     Otherwise, if it is set to ``Deny``, an ACL is used as an allowed list.
     
Denied requests are logged at :ref:`admin-log-access` as TCP_DENY.


.. _access-control-vhost_acl:

Virtual Host ACL
---------------------

A virtual host ACL judges whether to allow/deny/redirect HTTP requests from all clients.
Different response codes can be configured for each condition.
For redirected requests, reply with **302 Moved temporarily**. 
An ACL is saved at /svc/{virtual host name}/acl.txt. ::

   # /svc/www.example.com/acl.txt
   # Comma(,) is an identifier, and {condition},{keyword = allow | deny | redirect} format is used.
   # In case of deny, response code can be appended to the keyword.
   # If not specified, adopt ``DenialCode`` from ``<AccessControl>``.
   # In case of redirect, specify redirected URL after the keyword. (state as a value of Location header)
   # In order to combine(AND) multiple conditions, use &.
   
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
   
IPs, GeoIPs, Headers, and URLs can be used as conditions when..

-  **IP**
   $IP[...] format is used, IP, IP Range, Bitmask, and Subnet forms are supported.

-  **GeoIP**
   $IP[...] format is used, GeoIP must be configured. 
     
-  **Header**
   $HEADER[Key : Value] format is used, specific and patterned expressions are recognized. 
   If an identifier exists, like $HEADER[Key:], but the Value is an empty string, it means an empty header value. 
   If the ``Key`` is only specified without an identifier, like $HEADER[Key], the ``condition`` is used to judge the existence of a header for ``Key``.
    
-  **URL**
   $URL[...] format is used and this attribute can be omitted. This attribute recognizes specific and patterned expressions.
    
$ means "If it meets the condition, do~", but ! means "If it doesn't meet the condition, do~". 
The negated conditions shown below are supported. ::

   # If the country is not KOR, deny.
   !IP[KOR], deny
    
   # If referer header is missing, deny.
   !HEADER[referer], deny
   
   # /secure/ if the path is not a subordinate, allow.
   !URL[/secure/*], allow

