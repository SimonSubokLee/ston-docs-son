.. _release:

Appendix B: Release Notes
***************************


v2.0.x
====================================

2.0.4 (FEB 27, 2015)
----------------------------

**Function/Policy Update**

   -  ``Hash`` algorithm update at :ref:`origin-balancemode`
   
     | **Before.** hash(URL) / servers
     | **After.** `Consistent Hashing <http://en.wikipedia.org/wiki/Consistent_hashing>`_
     |     
   - Client requested URI is usable as a parameter when redirecting by :ref:`access-control-vhost`.
   
**Bug Fix**

   - Disk full due to unremmoved caching files
   
   

2.0.3 (FEB 9, 2015)
----------------------------

**Fuction/Policy Update**

   - DIMS internalization and enhancement
   - WM - traffic alert messages added
   
**Bug Fix**

   - WM - Virtual host generation failure


2.0.2 (Jan 28, 2015)
----------------------------

- Able to pass User-Agent header value from clients when requesting to the origin server.

**Bug Fix**

   - Failed to trim MP4 files with MDAT length 1.
   - WM - failed to show other clustered servers' graph.
   - WM - showing other clustered server's status as the relevant one.



2.0.1 (DEC 30, 2014)
----------------------------

**Bug Fix**

   - No HitRatio graph value


2.0.0 (DEC 17, 2014)
----------------------------

- Disk space optimization: just as downloaded from the origins. (please refer to :ref:`origin_partsize` )
- :ref:`env-cache-resource` added
- TLS 1.1 support
- :ref:`https-aes-ni` support by AES-NI.
- ECDHE CipherSuite support (please refer to :ref:`https-ciphersuite` )
- :ref:`admin-log-dns` added
- Policy Update: Seprate TTLs for each IP if the origin server is configured by domain
- origin :ref:`origin_exclusion_and_recovery` added
- origin :ref:`origin-health-checker` added
- :ref:`adv_topics_sys_free_mem` added
- etc.

  - Supported operating system updaated: CentOS 6.2 or later, Ubuntu 10.01 or later
  - NSCD daemon included in the installation package
  - :ref:`media-dims` included
  - Restart required after :ref:`getting-started-reset`
  - ``<DNSBackup>`` removed
  - ``<MaxFileCount>`` removed
  - ``<Distribution>`` removed, integrated into :ref:`origin-balancemode` 

