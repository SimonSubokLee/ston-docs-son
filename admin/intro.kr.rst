.. _intro:

Chapter 1. STON Edge Server
**********************************

.. toctree::
   :maxdepth: 2


Principles for Designing Successful Service
===================
Successful service incorporates availability, speed, and scalability. Kate Matsudaira who wrote an article 
“Scalable Web Architecture and Distributed Systems” also emphasizes these three principles.

**Availability**

A service must be always available. 90% of users move onto other competing services when there is a service failure. 
Although flawless system may not possible, recovery of system failure has to be swift.

**Speed**

Time correlates to revenue in business, and low latency leads to drop in sales. 
Every 0.1 second of latency causes 1% decreased revenue. 
47% of Amazon.com customers want to see loaded website on their screen in 2 seconds.

**Scalability**

Regardless of the number of users, a service has to be reliable. Scaling up and maintaining the service, 
ease of storage expansion and transaction processing capacity are included in Scalability. 
Manageability that concerns with ease of diagnosing, understanding problems and ease of making updates or 
modification is also an important factor.

Principles with the least required resources are evaluated as the most efficient ones. 
Resources include time, effort, trainings as well as money.

A successful service grows up and it has to be able to deal with more users and contents. 
As it grows more, principles are getting difficult to abide. 
Then, how these principles could be kept easily with the least expense?


The Expansion of Service
===============

Usually test or pilot service starts with a couple of servers. 
As the service grows up a little, the number of servers also increases accordingly. 
Contents renewal is meticulously carried out on one server at a time. 
It might be a laborious task, but managing the system is not a big deal at this point.

The service begins to expand with more users and more data to accumulate. 
From this point, managing each server one by one is getting more difficult. 
Hence, high cost storage for collecting data in one system is introduced(NAS, SAN, DAS, etc). 
High priced, but reliable storages make contents renewal easier because servers automatically acquire updated contents from the storage.

Now let’s see when the service is exploding. 
Increased number of servers requires more data from storage and causes data transfer overload on the storage. 
In order to resolve the data transfer overload issue, extremely expensive storage that can support higher bandwidth is considered. 
However, investing excessive budget on the storage will be questioned.

Another solution candidate is synchronization. 
Preparing entire data for the server is impractical so the storage needs to sort out contents. 
Management technique is essential to achieve precise contents control. 
Synchronization among a few servers might be simple, but the more servers and files to sync with, the harder synchronization will be. 
As the system expands, synchronization becomes slower, harder and unstable.

Contents are constantly changing. Synchronization takes more time when there are more files to add or delete. 
Likewise, a bigger scale service inevitably has complicated synchronization managing system. 
Failure of the managing system leads to the total service failure.

A simple method that can deliver contents to the server in a quick and flexible way is necessary for the extensive service.



The Scalability of Service and Data Transfer
=====================

A service can be classified into application and storage layers shown as the below figure.

.. figure:: img/intro_2layers.png
   :align: center
      
The storage layer supervises data at the core. 
The application layer is on top of the storage layer and the service logic is implemented in the layer. 
Contents transfer for small scale customers can be processed in this application layer. 
The storage layer and application layer can construct a decent early stage service.

.. figure:: img/intro_graph_1.png
   :align: center

As the service expands, use of budget is changing. In the early stage, logic development consumes huge portion of budget. 
On the contrary, in the growing period, data management consumes the most of budget as the number of user increases. 
Contents transfer is a main concern when the service arrives at puberty. 
How can the exploding bandwidth be covered? Contents transfer is the biggest obstacle in the aspect of service scale-out.

The Edge : Transfer layer
==========================

.. figure:: img/intro_3layers.png
   :align: center
   
Contents transfer becomes an enormous burden when the service grows up. 
Dozens of billions of shopping mall contents and video service contents already have been reached to TB long ago. 
The scalability of contents transfer must be considered in order to expand the service.

The edge indicates the surface layer of the service where users experience speed and availability of the service. 
Contents requested by users 'must' be transferred at any cost. 
Broken images or unavailable webpage on the user's screen fatally damages reputation of the service. 
Contents transfer burden at the application layer and the storage layer will be reduced if the edge layer can transfer contents.

Efficient and easily expandable edge layer eliminates the necessity of expanding other high cost layers. 
On the other hand, expanding storage layer and application layer is an inappropriate solution of high cost with low efficiency.

Then, how can STON edge server promote contents transfer faster and easier?




How Edge server works : Cache
=========================================

.. figure:: img/intro_cache1.png
   :align: center

The scale of data transfer is proportional to the number of users and the size of contents. 
At the edge layer the service can detect the number of users and particular contents they are requesting. 
Effective process flow will be bottom-up style from the edge layer. 
Therefore, the edge server adopted On-demand cache transfer method that responses to user's requests. 
In addition, management system won't be necessary. Detailed operation procedure is described in the below figure.

.. figure:: img/intro_cache2.png
   :align: center
   
When the edge server is requested to transfer contents for the first time, 
it obtains contents from the storage layer first, then transfer them to the user. 
The transferred contents are also saved in the edge server for the next use. 
From the second request for the contents, the edge server immediately retrieves saved contents and transfers them to customers. 
The saved contents are only valid for the pre-set TTL(Time-To-Live) period.

The edge server can process quite large amount of contents transfer in this way. 
This method enables to transfer massive data quickly with minimal expansion of the application and the storage layer. 
Therefore, any expandable services should consider the edge server.

The STON edge server is the software that aims for unrestricted and unconditional environment. 
The server is designed to provide maximum performance on any types of hardware platform.


**CPU:** Optimized for Many-Core. Throughput is proportional to the number of CPU cores.

**Memory:** Process gets faster with large Memory and saves Disk I/O.

**Disk:** Evenly distributed I/O for caching more data.

**NIC:** Guarantee bandwidth of either 4Gbps NIC Bonding or 10Gbps NIC.


The STON edge server supports a powerful live monitoring and logging. 
Administrator can check the current service status in real-time with statistics result updated by every second. 
The server supports universal formats like JSON, XML, SNMP to provide service statistics.

The STON also provides simple setup for administrators because its design idea is to offer an edge server for administrators. 
Web Management page provides an intuitive setting. Detailed server setting can be configured by editing only two XML files.


Effects of the Edge Server
======================
The followings are the effects of the edge server.

#. Easy and convenient service acceleration.
#. Shields the service origin from external access(Origin Shielding).
#. Supports the service to play a fundamental role.

Advantages of adopting the edge server are listed on the following application examples.


Games
----------------------------

Traditionally game services require excessive bandwidth. 
In addition, there are various categories in the game service from "Masterpiece games" to casual games. 
Especially explosive growth in smartphone games and proliferation thereof diversified the game service form.

.. figure:: img/icons_game.png
   :align: center

- **High Bandwidth Throughput**
  
  A universal method to acquire high bandwidth with a single server is bonding 1Gbps NIC(Network Interface Controller). 
  As a result of bonding, up to 4Gbps can be achieved. 
  In the recent market, 10Gbps NICs have been come into wide use.

  ``STON`` guarantees full bandwidth for both 4Gbps NIC Bonding and 10Gbps NIC.
  
- **Guaranteed User Bandwidth**

  Every user wants to download games as fast as possible. 
  Users who adopted fiber optic LAN might complain for the service if they get less than 100Mbps. 
  Once a user decides to play a game, he or she wants to play it right away. 
  As long as a server has a remaining physical bandwidth, it has to uniformly guarantee maximum speed to every single user. 
  
  ``STON`` guarantees maximum transmitting speed to all users. 
  
- **Processing Massive Volume File**

  Nowadays most games have very large volume of installation files and there are so many of them to fill a single DVD disk. 
  Most "masterpiece games" consist of Dozens of GB installation files. 
  Howerver, if the file size is too large to cache in the server memory, critical service failure is anticipated. 
  The worst case is when all users are requesting different parts of a massive volume file to the server.
  
  ``STON`` supports unlimited caching file size. 
  The STON edge server always guarantees powerful performance by properly swapping between memory and disk.
    
- **Processing Range Request**

  As files to transfer are getting heavier, P2P solution based on the grid delivery method is widely being used. 
  The P2P solution shreds a single file into small pieces to send or receive, therefore it requests enormous HTTP range to the server. 
  Theoretically, ten thousands of clients could request different ranges from a 10GB file. 
  Regardless of requested range from clients, the service has to be prompt. 
  On the other hand, transferred data size from the server cannot exceed the original file size.
  
  ``STON`` is loaded with the file system that is optimized for range request. 
  In addition, The STON edge server guarantees fast response with multi download.
  It will not waste a single byte transmission from the origin server. 


Online Shopping Malls
----------------------------

In case of an online shopping mall, accessibility of the website is directly connected to the total sales. 
Mobile shopping using smartphones became a general trend instead of traditional PC based online shopping. 
A service will be facing a dead end if it cannot manage various shopping environments and infinitely growing number of files. 

.. figure:: img/icons_shopping.png
   :align: center

- **Zillions of Tiny Files**

  In order to keep billions of files that are infinitely increasing, an expensive storage is needed. 
  However, the Edge server takes count of economic feasibility so this solution is not preferred. 
  There could be a service that consists of a billion of 1KB files, and caching all of them is not possible. 
  Therefore, a method that minimizes load of origin server and keeps frequently requested files has to be developed.

  ``STON`` utilizes resources of available memory and disk space for caching. 
  Access frequency of all files are managed in real time, and by the LRU(Least Recently Used) algorithm, a file that has not been used recently is discarded first. 

- **Millions of Users**

  An online shopping mall can handle tremendous amount of simultaneous requests from millions of users. 
  An abrupt event could cause dramatic increase of website access(Burst)
  In case of burst condition, the server has to survive and return to stable status after the burst.
  
  ``STON`` guarantees CPU scalability (The performance of solution is proportionate to the number of resources). 
  Flexible HTTP Keep-Alive handling and socket manage guarantees stability under the burst condition. 
  
- **Swift Response**

  Pleasant online shopping experiences come from fast loading web pages.
  Users are impatient of loading signs.
  If the page is not fully loaded in 3 seconds, users start to build up negative images on their shopping experiences and might move on to another shopping sites.
  Generally main page is roughly made up of a hundred files, and these files have to be loaded flawlessly in one second. 
  
  ``STON`` guarantees swift response by real time file indexing. 
  Also, seamless replacement of updated files on the background enhances service quality by decoupling contents from origin server.
  Log and statistics are supported for all HTTP responses(First byte response, complete transaction) for detecting any performance degradation in real time.
  
- **Page TTL**

  Most users start from main page and move on to the upper category page, then to the lower category page, and finally to the detail product page. 
  Each page has different exposure frequencies, as well as refreshing cycles. 
  Hence, smart page caching and refresh technique are needed.
  
  ``STON`` can allocate separate TTL to independent URL. 
  In addition, the STON edge server provides various suitable refreshing methods such as Purge, Expire, ExpireAfter, and HardPurge.
  
  

Media
----------------------------

Exclusive protocols for media is losing their strength while simple, but powerful combination of HTTP and MP4 is gaining influence. 
Streaming based on HTTP protocol will come to the fore if variable connection statuses of mobile devices are considered.

.. figure:: img/icons_media.png
   :align: center

- **Perception of Media**

  A media file should not be recognized as one huge chunk of file. 
  Reducing bandwidth and linking various additional functions are only possible when the format of media file is correctly recognized. 
  If the server requires entire file to extract file format, users will be wasting a lot of time until the server acquires entire file.
  Most likely, users won't even wait for that.
  
  ``STON`` supports MP4, MP3, M4A, FLV formats. 
  As soon as the server starts downloading a media file, it preferentially caches required section for HTTP Pseudo Streaming.
  
- **Reorganizing Media Header**

  If the header is located at the end of file, HTTP Pseudo Streaming is not available.
  An exclusive media player is required for this type of files, but installing subsidiary program is obnoxious for all users.
  
  ``STON`` An encoded MP4 file followed by a header needs additional process to reorganize header location to the front of file structure. 
  The STON edge server automatically relocates header to the front for smooth service.

- **Adjustable Bandwidth**

  Not all users watch the whole video clip. 
  An effective streaming method is to provide reasonable amount of bandwidth for smooth playback.
  An identical media is served with different bitrates from 360p to 1080p.
  
  ``STON`` optimizes media file transfer bandwidth with Bandwidth-Throttling. 

- **Extracting Section**

  Some preview/highlight/share services only provide a specific portion of a video clip.
  Extracting entire file for the service will be too much waste of process time and storage.
  Further, every single user might request different section of a media file.
  Some media players even implement skip function with segment playback.
  
  ``STON`` trims a media file to extract segments to serve just as a complete file.

  
  
News / Community
----------------------------

There are several interesting points to check in the websites that has a significant number of loyal users.
These websites attract users with a similar matter of concern so users stay on the website for very long time while vigorous communications are ongoing. 
Service patterns of these websites vary by their subjects and it is picky to meet their service requirements. 

.. figure:: img/icons_news.png
   :align: center

- **304 Not Modified**
  
  Since these users are very loyal to the website, most files are already cached in local storage.
  Hence, requesting updated status of cached files is dominant than actual file transfer.
  
  ``STON`` guarantees frequently accessed files to reside in the memory.
  "Update check" request is processed immediately.
  
- **Bypass**

  Page always contains non-cacheable area such as user specific pages, new postings and replies.
  Even this case, usually a single domain is delegated to Reverse-Proxy instead of separating a page into multiple domains.

  ``STON`` elaborately classifies bypass targets based on various conditions.
  Also, the server maintains login session by using Origin Affinity and Private function.
  
- **Unstable Origin**

  Small or mid size websites and personal websites cannot afford expensive equipment, infrastructures and labor force.
  Origin server failure is relatively frequent and economic feasibility to improve service quality is extremely low.
  
  ``STON`` detects overload or service failure to execute automatic exclusion/recovery.
  원본서버 과부하 또는 장애를 판단하여 자동으로 배제/복구가 이루어진다. 
  When the origin server fails, the server automatically extends TTL to decrease origin server dependency.
  
- **Image Processing**

  An identical image sometimes needs to be displayed in different ways according to user environments.
  Images in the search result will be displayed as thumbnails, and some websites might want their own watermark on the image.
  Processing every image into a specific format is waste of storage, time, and effort.
  
  ``STON`` supports DIMS function that can generate desired image format from a single image in the server by URL call.
  
  
File Based Server
----------------------------

The Edge server builds on the Reverse-Proxy structure. 
Fundamental concept of Reverse-Proxy is to copy/modify/manage files from the remote server to the local storage.
If qualified STON can interwork with a service server, storage centralization and synchronization issue can be resolved.
In addition, both developing time reduction and service reliability improvement can be attained.

.. figure:: img/icons_file.png
   :align: center

- **File I/O Support**

  If a specific protocol is required, then the server is subordinate to the module.
  Even if the server is interworking with the module, performance degradation could spoil everything.
  Therefore, I/O interface between module and server has to be simplified.
  
  ``STON`` adopts standard File I/O. 
  A dedicated server and STON is connected by Linux Kernel(VFS) in order to guarantee high performance.
  
- **Connect to Web Server**
  
  Standard Reverse-Proxy might not be available if any exclusive expansion module is installed on the standard web server(Apache, Lighttpd, NginX).
  For example, a file service or a payment service that are linked with DB/WAS is hard to expand its service.
  
  ``STON`` If DocumentRoot of Apache is set to STON, Apache will perceive STON as a physical disk and nothing needs to be configured.

- **Connect to Wowza**

  In the media service field, Wowza is considered as a standard. 
  However, Http caching function of Wowza is not only inconvenient but also very limited. 
  Also other "exclusive" protocols except HTTP is fading away from the market. 

  ``STON`` (STON을?) 로컬 디스크로 Mount할 수 있을 뿐만 아니라 MP4헤더 변환, Trimming등 
  모든 기능을 사용할 수 있다.

- **리소스 제약**

  Back-end에 존재하는 파일을 Front-End의 사용자에게 전달하는 서버라면 항상 
  파일 동기화가 문제된다. 
  게임서버, SNS서버 등 전용서버의 개발 이슈는 항상 존재한다. 
  이런 서버의 경우 중단 없이 장기간 운영 되야 하므로 
  메모리, 디스크 사용이 엄격하게 제한되어야 한다.

  ``STON`` 최대 메모리, 디스크 사용량을 제한할 수 있다. 
  또한 디스크로 Mount하여도 다른 모든 기능은 동일하게 동작하여 복합적인 서비스를 
  최소한의 솔루션으로 구성할 수 있다.


STON은 이러한 특성들을 적극 활용하는 다음 서비스들과 함께 성장하고 있다. 

.. figure:: img/intro_reference.png
   :align: center
