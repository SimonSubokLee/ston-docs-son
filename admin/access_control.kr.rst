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
     For denied requests, response code that is configured in ``DenialCode`` property will be returned.
     ``Default (default: Allow)`` If the property is set to ``Allow``, ACL is used as block list. 
     Otherwise, if it is set to ``Deny``, ACL is used as allowed list.
     
Denied requests are logged at :ref:`admin-log-access` as TCP_DENY.


.. _access-control-vhost_acl:

Virtual Host ACL
---------------------

Judges whether to allow/deny/redirect for the HTTP requests from all clients.
Different response codes can be configured for each condition.
For redirected requests, reply with **302 Moved temporarily**. 
ACL is saved at /svc/{virtual host name}/acl.txt. ::

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
   
IP, GeoIP, Header, URL can be used as conditions.

-  **IP**
   $IP[...] format is used, and IP, IP Range, Bitmask, Subnet forms are supported.

-  **GeoIP**
   $IP[...] format is used, and GeoIP has to be configured in order to use. 
     
-  **Header**
   $HEADER[Key : Value] format is used. 
   Value recognizes specific and patterned expressions. 
   An identifier exists like $HEADER[Key:], but if the Value is an empty string, this stands for empty header value. 
   If the ``Key`` is only specified without an identifier like $HEADER[Key], the condition is used to judge existence of header for ``Key``.
    
-  **URL**
   $URL[...] format is used and can be omitted. This recognizes specific and patterend expressions.
    
$ means "If it meets the condition, do~", but ! means "If it doesn't meet the condition, do~". 
The below negated conditions are supported. ::

   # If the country is not KOR, deny.
   !IP[KOR], deny
    
   # If referer header is missing, deny.
   !HEADER[referer], deny
   
   # /secure/ path is not a subordinate, allow.
   !URL[/secure/*], allow

