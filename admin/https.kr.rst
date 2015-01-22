.. _https:

Chapter 9. HTTPS
******************

This chapter explains how to configure HTTPS.
HTTPS supports SSL 3.0, TLS 1.0 and TLS 1.1, but for a security reason, SSL 2.0 is allowed only for upgrade.
HTTPS is only used in between clients and STON. 
STON does not use HTTPS to communicate with the origin server.
In terms of security and performance, it will be inappropriate if STON relays HTTPS.
If HTTPS has to be used for communicating with the origin server, using :ref:`bypass-port` is recommended.


.. toctree::
   :maxdepth: 2



Service Configuration
====================================

As long as specific IP or port is not designated, default binding service address is "*:443".
This can be configured in the global configuration(server.xml). ::

   # server.xml - <Server>

   <Https>
      <Cert>/usr/ssl/cert.pem</Cert>
      <Key>/usr/ssl/certkey.pem</Key>
      <CA>/usr/ssl/CA.pem</CA>
   </Https>
    
   <Https Listen="1.1.1.1:443">
      <Cert>/usr/ssl_ip_port/cert.pem</Cert>
      <Key>/usr/ssl_ip_port/certkey.pem</Key>
      <CA>/usr/ssl_ip_port/CA.pem</CA>
   </Https>
    
   <Https Listen="*:886">
      <Cert>/usr/ssl_port/cert.pem</Cert>
      <Key>/usr/ssl_port/certkey.pem</Key>
      <CA>/usr/ssl_port/CA.pem</CA>
   </Https>   
   
-  ``<Https>`` Configures HTTPS
   
   -  ``<Cert>`` A server certification
   
   -  ``<Key>`` A private key of server certification. Encrypted format is not supported.
   
   -  ``<CA>`` CA(Certificate Authority) Chain certification(체인?? 인증서: 네 chain 맞습니다)
  
Even if the same port is being serviced, more specific expression has a priority. 

For example, if there are multiple NIC like the above example, 
the client that came through 1.1.1.1:443 will be serviced with a specific certification, 
and the client that came through 1.1.1.4:443 will be serviced with a general certification.
Even if you overwrite a certification with an existing file name, it'll be reflected when reloading.

.. note::

   Cerification format is PEM(Privacy Enhanced Mail), and only supports RSA asymmetric key algorithm.


.. _https-aes-ni:

SSL/TLS Acceleration
====================================

SSL/TLS is accelerated by CPU(AES-NI).
A CPU that supports AES-NI, SSL/TLS will preferentially choose AES algorithm.
If AES-NI is recognized, the following will be recorded in the Info.log file. ::

   AES-NI : ON (SSL/TLS accelerated)
   
Administrator can select whether to use AES-NI or not. ::

   # server.xml - <Server><Cache>

   <AES-NI>ON</AES-NI>   

-  ``<AES-NI> (default: ON)`` Use AES-NI.



.. _https-ciphersuite:

CipherSuite Selection
====================================

The followings are supported CipherSuites.

- RSA_WITH_RC4_SHA
- RSA_WITH_RC4_MD5
- RSA_WITH_AES_128_CBC_SHA
- RSA_WITH_AES_256_CBC_SHA
- RSA_WITH_3DES_EDE_CBC_SHA
- ECDHE_RSA_WITH_AES_128_CBC_SHA
- ECDHE_RSA_WITH_AES_256_CBC_SHA

You can choose which CipherSuite to use by configuring ``CipherSuite`` property in ``<Https>``. ::

   # server.xml - <Server>

   <Https CipherSuite="ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP">
      <Cert>/usr/ssl/cert.pem</Cert>
      <Key>/usr/ssl/certkey.pem</Key>
      <CA>/usr/ssl/CA.pem</CA>
   </Https>   

-  ``CipherSuite`` Abides by `SSL CipherSuite expression of Apache mode_ssl <http://httpd.apache.org/docs/2.2/mod/mod_ssl.html#sslciphersuite>`_.



.. _https-ciphersuite-query:

CipherSuit Inquiry
====================================

Inquiries CipherSuite configuration result. 
CipherSuite expression abides by `OpenSSL 1.0.0E <https://www.openssl.org/docs/apps/ciphers.html>`_. ::

   http://127.0.0.1:10040/monitoring/ssl?ciphersuite=...

The result is returned in JSON format. ::

  {
      "version": "2.0.0",
      "method": "ssl",
      "status": "OK",
      "result":
      [
          {
              "Name" : "AES128-SHA", 
              "Ver" : "SSLv3", 
              "Kx" : "RSA", 
              "Au" : "RSA", 
              "Enc" : "AES(128)", 
              "Mac" : "SHA1"
          },
          {
              "Name" : "AES256-SHA", 
              "Ver" : "SSLv3", 
              "Kx" : "RSA", 
              "Au" : "RSA", 
              "Enc" : "AES(256)", 
              "Mac" : "SHA1"
          }
      ]
  }



Multi Domain Configuration
====================================

SSL configuration can cause problems when running multiple services simultaneously with one server.
Most Web/Cache servers decide which virtual host will be used for the service by examining Host header in the HTTP request 

.. figure:: img/ssl_alert.png
   :align: center
      
   General HTTPS Communication
   
Generally SSL identifies the domain(winesoft.co.kr) of the server that a client(Browser) is trying to access by confirming a certificate. 
When the certificate is rejected for identification(wrong certificate, expired certificate, etc), user will be asked whether to trust website or not(sometimes the website might be blocked). 
If the client decides to trust the website, SSL communication will be established without a proper identification.

.. figure:: img/faq_ssl1.jpg
   :align: center
      
   Client will decide whether to trust website or not.
   
It will not be a problem if there is only one virtual host that uses SSL in the server. 
However, a server that runs multiple virtual hosts at the same time could be an issue. 
The problem occurs when the server transmits the certificate to a client("2. Certificate Transmission" of the "General HTTPS Communication")
because the server does not know which Host the client is trying to reach. 

The following shows typical solutions for this issue.

=================== ====================================== ========================================================================
Methods	            Pros	                               Cons
=================== ====================================== ========================================================================
SNI                 Works with server configuration(default)	           Does not support Windows XP and IE6
Multi Certificate	Works with replaced certificate	               The subject of main domain or service has to be the same and reissuing the certificate could be too frequent
Multi Port          Works with modified port	               The web page has to specifiy HTTPS ports
Multi NIC	        Works with server configuration(most popular)    NIC and additional IP configuration are required
=================== ====================================== ========================================================================


SNI (Server Name Indication)
--------------------------

This method uses `SNI(Server Name Indication) <http://en.wikipedia.org/wiki/Server_Name_Indication>`_ expansion field of SSL/TLS. 
When the client requests SSL connection to the server, target virtual host should be specified like the Host header of HTTP request. 
This is the most delicate method, but there are some compatibility issues. 
The followings are the client list that does not support SNI.
(source: `Wikipedia - Server Name Indication <http://en.wikipedia.org/wiki/Server_Name_Indication#Client_side>`_ ).

- Internet Explorer (any version) on Windows XP or Internet Explorer 6 or earlier
- Safari on Windows XP
- BlackBerry Browser
- Windows Mobile up to 6.5
- Android default browser on Android 2.x[34] (Fixed in Honeycomb for tablets and Ice Cream Sandwich for phones)
- wget before 1.14
- Java before 1.7

SNI is unavailable in reality, therefore STON does not support SNI.



Multi Certificate
--------------------------

Multiple domains or a wildcard(eg. *.winesoft.co.kr) is specified in the certificate to identify multiple domains with a single certificate.

.. figure:: img/faq_ssl2.jpg
   :align: center
      
   Multiple domains can be identified with a single certificate.
   
This method is effective if subjects of the service are the same, otherwise this method is realistically unavailable. 
Replacing certificates will work for this method, therefore you don't need to configure anything from STON.
[ Refer `DigiCert <http://www.digicert.com/wildcard-ssl-certificates.htm>`_].



Multi Port
--------------------------

SSL basically use 443 port.
If you configure the SSL port that is not overlapping, you can install multiple certificates. 
Client can enable SSL communication by specifying port as below. ::

    https://winesoft.co.kr:543/
    
STON can configure multiple certificates by specifying port in the Listen property as below. ::

   # server.xml - <Server>

   <Https> ..A Company Certificate.. </Https>  
   <Https Listen="*:543"> ..B Company Certificate.. </Https>  
   <Https Listen="*:544"> ..C Company Certificate.. </Https>
    
This method is economically effective, but every web page link has to be specified with HTTPS port.


Multi NIC
--------------------------

If there are multiple server NICs, different IPs can be assigned to each NIC. 
Therefore, each server IP can have separate certificates in order to decide which certificate to use based on the server IP that a client is connected. 
The following shows how to configure multiple certificates by specifying IP in the Listen property. ::

   # server.xml - <Server>

   <Https Listen="10.10.10.10"> ..A Company Certificate.. </Https>  
   <Https Listen="10.10.10.11"> ..B Company Certificate.. </Https>  
   <Https Listen="10.10.10.12"> ..C Company Certificate.. </Https>

This is the most popular method.

