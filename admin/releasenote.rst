.. _release:

Appendix B: Release Notes
***************************

v2.1.x
====================================


2.1.0 (APR 15, 2015)
----------------------------

    | :ref:`adv_topics_indexing` added
    | Animated GIF :ref:`media-dims` supported
    | :ref:`media-dims` statistics supported

**Feature/Policy Update**

   -  Directory expression removed from :ref:`caching-purge` (purge, expire, hardpurge, expireafter)
        URL by directory expression (example.com/img/) caches the returned file from the origin.
        Directory expression (example.com/img/) merged with pattern (example.com/img/*)
   -  API expressions added
       | /monitoring/average.xml
       | /monitoring/average.json
       | /monitoring/realtime.xml
       | /monitoring/realtime.json
       | /monitoring/fileinfo.json
       | /monitoring/hwinfo.json
       | /monitoring/cpuinfo.json
       | /monitoring/vhostslist.json
       | /monitoring/geoiplist.json
       | /monitoring/ssl.json
       | /monitoring/cacheresource.json
       | /monitoring/origin.json
       | /monitoring/coldfiledist.json
   -  WM - resolv.conf editing removed


v2.0.x
====================================


2.0.6 (APR 28, 2015)
----------------------------

**Feature/Policy Update**

   -  WM - resolv.conf editing removed

**Bug Fix**

   - abnormal termiation from MP4 analysis with broken headers
   
   
2.0.5 (APR 1, 2015)
----------------------------

**Feature/Policy Update**

   - Serves trimmed MP4 by HLS
     The following expressions trim the original media file (/vod.mp4) from 0 to 60 seconds and serve in HLS.
     | /vod.mp4?start=0&end=60/**mp4hls/index.m3u8**
     | /vod.mp4**/mp4hls/index.m3u8**?start=0&end=60
     | /vod.mp4?start=0/**mp4hls/index.m3u8**?end=60
   - HLS index file (.m3u8) update
     **Before.** Version 1
     **After.** Version 3 (changeable back to version 1)

**Bug Fixes**

   - abnormal termination in HLS conversion with HTTP encoded special characters 
   - overloaded CPU for MP4 media with broken headers 
   - audio/video synchronization failure while serving MP4 with uneven audio keyframe in HLS
   - RRD - statistics bug: average response time shown in total
   - WM - forcing formatting new disks remoced 


2.0.4 (FEB 27, 2015)
----------------------------

**Feature/Policy Update**

   -  ``Hash`` algorithm update at :ref:`origin-balancemode`
   
     | **Before.** hash(URL) / servers
     | **After.** `Consistent Hashing <http://en.wikipedia.org/wiki/Consistent_hashing>`_
     |     
   - Client requested URI is usable as a parameter when redirecting by :ref:`access-control-vhost`.
   
**Bug Fix**

   - Disk full due to unremmoved caching files
   
   

2.0.3 (FEB 9, 2015)
----------------------------

**Feature/Policy Update**

   - DIMS internalization and enhancement
   - WM - traffic alert messages added
   
**Bug Fix**

   - WM - Virtual host generation failure


2.0.2 (Jan 28, 2015)
----------------------------

- Able to pass User-Agent header value from clients when requesting to the origin server.

**Bug Fixes**

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

