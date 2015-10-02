.. _intro:

Chapter 1. Introduction
**********************************

.. toctree::
   :maxdepth: 2


Designing and Growing a Successful Service
===================
Successful service incorporates availability, speed and scalability. Kate Matsudaira emphasized these three principles too, from the article 'Scalable Web Architecture and Distributed Systems'.

**Availability**

A service must be always available. Ninety percent of users move on to competitors if failure occurs. The recovery has to be swift, even though a completely flawless system might not be possible.

**Speed**

Time correlates to revenue in business, and high latency from e-commerce means a drop in sales. Every 0.1 second of latency means decreased revenue by one percent. Forty-seven percent of Amazon.com customers want a website loaded on their screen in two seconds.

**Scalability**

Regardless of the user number, a service has to be reliable. Scalability includes scale-up, service maintenance, easy storage expansion and transaction processing capacity. Manageability, with regard to the ease of diagnosing and understanding problems along with the easy updates or modifications, is also an important factor.


It is the best to keep these principles with least resources such as time, training and money. As a successful service grows, it must be able to deal with more users and content while keeping the principles. Doing so can be a difficult task. With a couple of servers, a test or pilot service may start. As the service begins to grow, the number of servers also should increase accordingly. Content renewal must be meticulously carried out on one server at a time. It might be a laborious task, but managing the system is not such an impossible task up to this point.

As the service begins to expand with even more users and data, managing each server one by one becomes more difficult. Thus, high-cost storage for collecting data in one system should be introduced (NAS, SAN, DAS and etc.). High-priced but reliable storage systems make content renewal easier because servers can automatically acquire updated content from the storage. Now, what about the exploding service scale? More servers require more data from storage and cause data delivery overload on the storage. In order to resolve the data overload issue, a new storage system to support higher bandwidth is often be considered, which may be highly expensive. However, investing an excessive amount of the budget on storage may be questioned from time to time.

Data synchronization is claimed as another potential solution . Getting all data ready is impractical, so the storage system needs to be the one sorting out contents. Management is essential to achieve precise content control. Synchronization among a few servers might be easy, but the more servers and files to sync, the harder synchronization. The entire system expands and synchronization becomes slower, harder, and more unstable even. Content is constantly changing. Synchronization takes more time with more files to add or delete. Likewise, a bigger scale service inevitably requires a complicated synchronization managing system. Failure of the managing system may lead to total service failure.

A simple, quick and flexible method to deliver content to the servers would be preferred for bigger services. 


And the Data Delivery
=====================

A service may be classified into application and storage layers, as shown in the figure below. 

.. figure:: img/intro_2layers.png
   :align: center
      
The storage layer supervises data at the core. The application layer is on top of the storage layer. Within the application layer, the service logic is implemented and content delivery can also be processed for a small number of customers. The storage layer and application layer can create a decent early stage service.

.. figure:: img/intro_graph_1.png
   :align: center

As the service expands, the budget may change. In the early stage, logic development consumes a huge portion of budget. On the contrary, in the growing period, data management consumes most of the budget as the number of users increases. Content delivery becomes the main concern as the service matures, making it the biggest obstacle for service scale-out. How can the exploding bandwidth be covered? 

The Edge : Delivery Layer
==========================

.. figure:: img/intro_3layers.png
   :align: center
   
Content delivery can become an enormous burden when the service reaches maturity. Dozens of billions of shopping mall content and video service content have already reached terabytes long ago. The scalability of content delivery must be considered in order to expand the service.

The edge indicates the surface layer of the service where users experience speed and availability of the service. No matter the cost, content requested by users must be responded. Broken images or unavailable webpages on the user's screen fatally damages the reputation of the service. The burden of content delivery at the application layer and the storage layer will be reduced if the edge layer can deliver content.

Having an efficient and easily expandable edge layer eliminates the necessity of expanding other high cost layers. On the other hand, expanding the storage layer and the application layer is an inappropriate solution due to  high cost and low efficiency.

Then, how can the STON edge server promote content delivery faster and easier?


Edge Server 101 : Caching
=========================================

.. figure:: img/intro_cache1.png
   :align: center

The scale of data delivery is proportional to the number of users and the size of content. At the edge layer, the service can detect the number of users and the particular content they are requesting. The effective process flow will be bottom-up style from the edge layer. Therefore, the edge server adopted an on-demand caching that responds to user's requests. In addition, a management system won't be necessary. A detailed operation procedure is described in the figure below.

.. figure:: img/intro_cache2.png
   :align: center
   
When the edge server is requested to deliver content for the first time, it obtains content from the storage layer first, then transfers it to the user. The transferred content are also saved in the edge server for the next use. From the second request for the content on, the edge server immediately retrieves saved content and deliver it to customers. The saved content is only valid for the pre-set TTL (Time-To-Live) period.

The edge server can process quite a large amount of content to deliver in this way. This method enables the delivery of massive data quickly with minimal expansion of the application and the storage layer. Therefore, any expandable services should consider the edge server.

The STON edge server is the software that aims for an unrestricted and unconditional environment. The server is designed to provide maximum performance on any type of hardware platform.



**CPU:** Optimized for Many-Core. Throughput is proportional to the number of CPU cores

**Memory:** A larger memory enables a faster processing and saves Disk I/O as well. 

**Disk:** Evenly distributed I/O for caching more data

**NIC:** Guaranteed bandwidth of either 4Gbps NIC Bonding or 10Gbps NIC


The STON edge server supports powerful live monitoring and logging. The administrator can check the current service status in real-time with statistical results updated every second. The server supports universal formats like JSON, XML, and SNMP to provide service statistics.

STON also provides simple setup for administrators because it is specifically designed to offer an edge server for administrators. The Web Management page provides an intuitive setting. Detailed server settings can be configured by editing only two XML files.


Benefits
======================
The following lists the effects of the edge server:

#. Provides easy and convenient service acceleration.
#. Shields the service origin from external access (Origin Shielding).
#. Allows other layers to concentrate on their fundamental roles.

Advantages of adopting the edge server are listed in the following application examples.


Gaming
----------------------------

Traditionally, gaming services require high bandwidth. In addition, there are a number of categories in the game service, from 'Masterpiece' games to casual ones. Gaming from smartphones is booming and services are diversified.

.. figure:: img/icons_game.png
   :align: center

- **High Bandwidth Throughput**
  
A universal method to acquire high bandwidth with a single server is bonding 1Gbps NIC (Network Interface Controller). As a result of bonding, up to 4Gbps can be achieved. In the recent market, 10Gbps NICs have come into wide use.

  ``STON`` guarantees full bandwidth for both 4Gbps NIC Bonding and 10Gbps NIC.
  
- **Guaranteed User Bandwidth**

  Every user wants to download games as fast as possible. Users who adopted fiber optic LAN might complain about the service if they get less than 100Mbps. Once a user decides to play a game, he or she wants to play it right away. As long as a server has remaining physical bandwidth, it has to uniformly guarantee maximum speed to every single user.
  
  ``STON`` guarantees maximum transmitting speed to all users. 
  
- **Processing Massive Volume File**

Nowadays most games have a very large volume of installation files and there are too many of them to fill a single DVD disk. Most masterpiece games consist of dozens of GB installation files. However, if the file size is too large to cache in the server memory, critical service failure is anticipated. The worst case is when all users are requesting different parts of a massive volume file from the server.
  
  ``STON`` supports unlimited caching file size. The STON edge server always guarantees powerful performance by properly swapping between memory and disk.
    
- **Processing Range Request**

  As files to deliver are getting heavier, the P2P solution--based on the grid delivery method--is widely being used. The P2P solution shreds a single file into small pieces to send or receive; therefore, it requests enormous HTTP range from the server. Theoretically, ten thousand clients could request different ranges from a 10GB file. Regardless of the requested range from clients, the service has to be prompt. On the other hand, the size of transferred data from the server cannot exceed the original file size.
  
  ``STON`` is loaded with a file system that is optimized for range request. In addition, the STON edge server guarantees fast response with multi-download. It will not waste a single byte of transmission from the origin server. 


E-commerce
----------------------------

In the case of an online shopping mall, accessibility of the website is directly connected to the amount of total sales. Mobile shopping via smartphones became a general trend instead of traditional PC-based online shopping. A service will be facing a dead end if it cannot manage various shopping environments and an infinitely growing number of files. 

.. figure:: img/icons_shopping.png
   :align: center

- **Zillions of Tiny Files**

  In order to keep billions of files that are infinitely increasing, an expensive storage is needed. However, the edge server takes account of economic feasibility, so this solution is not preferred. There could be a service that consists of a billion 1KB files, and caching all of them is not possible. Therefore, a method that minimizes the load of the origin server and keeps frequently requested files has to be developed.
  
  ``STON`` utilizes resources of available memory and disk space for caching. Access frequency of all files are managed in real time, and by the LRU (Least Recently Used) algorithm, in which a  file that has not been used recently is discarded first.

- **Millions of Users**

  An online shopping mall can handle a tremendous amount of simultaneous requests from millions of users. An abrupt event could cause a dramatic increase of website access (Burst). In the case of a burst condition, the server has to survive and return to stable status after the burst.
  
  ``STON`` guarantees CPU scalability (The performance of the solution is proportionate to the number of resources). Flexible HTTP Keep-Alive handling and socket managing guarantee stability under the burst condition. 
  
- **Swift Response**

  Pleasant online shopping experiences come from fast loading web pages. Users are impatient with loading signs. If the page is not fully loaded in three seconds, users start to build up negative images of their shopping experiences and might move on to other shopping sites. Generally, the main page is roughly made up of a hundred files, and these files have to be loaded flawlessly in one second. 
  
  ``STON`` guarantees swift response by real time file indexing. Also, seamless replacement of updated files on the background enhances service quality by decoupling content from the origin server. Logs and statistics are supported for all HTTP responses (First byte response, complete transaction) for detecting any performance degradation in real time.
  
- **Page TTL**

  Most users start at the main page and move on to the upper category page, then to the lower category page, and finally to the detailed product page. Each page has different exposure frequencies, as well as refreshing cycles. Hence, both smart page caching and refresh technique are needed.
  
  ``STON`` can allocate separate TTL to independent URL. In addition, the STON edge server provides various suitable refreshing methods, such as Purge, Expire, ExpireAfter, and HardPurge.
  
  

Media
----------------------------

Exclusive protocols for media are losing strength, while the simple but powerful combination of HTTP and MP4 is gaining influence. Streaming based on HTTP protocol will come to the forefront if variable connection statuses of mobile devices are considered.

.. figure:: img/icons_media.png
   :align: center

- **Perception of Media**

  A media file should not be recognized as one huge chunk of file. Reducing bandwidth and linking various additional functions are only possible when the format of the media file is correctly recognized. If the server requires the entire file to extract the file format, users will be wasting a lot of time until the server acquires the entire file. Most likely, users won't even wait for that.
  
  ``STON`` supports MP4, MP3, M4A, and FLV formats. As soon as the server starts downloading a media file, it preferentially caches required sections for HTTP Pseudo Streaming.
  
- **Reorganizing Media Header**

  If the header is located at the end of file, HTTP Pseudo Streaming is not available. An exclusive media player is required for this type of file, but installing a subsidiary program is an obnoxious extra task for all users.
  
  ``STON`` An encoded MP4 file followed by a header needs an additional process to reorganize the header location to be in the front of file structure. For smooth service, the STON edge server automatically relocates the header to the front.

- **Adjustable Bandwidth**

  Not all users watch the whole video clip. An effective streaming method is to provide a reasonable amount of bandwidth for smooth playback. An identical media is served with different bitrates from 360p to 1080p.
  
  ``STON`` optimizes media bandwidth with Bandwidth-Throttling. 

- **Extracting Section**

  Some preview/highlight/share services only provide a specific portion of a video clip. Extracting the entire file for the service will waste too much process time and storage. Further, every single user might request different sections of a media file. Some media players even implement a skip function with segment playback.
  
  ``STON`` trims a media file to extract segments to serve as just a complete file.

  
  
News / Community
----------------------------

There are several interesting points to check out on websites that have a significant number of loyal users. These websites attract users with a similar matter of concern, so users stay on for a very long time; thus, vigorous communications are ongoing. The service patterns of these websites vary by their subjects and it is tricky to meet their service requirements. 

.. figure:: img/icons_news.png
   :align: center

- **304 Not Modified**
  
  Since these users are very loyal to the website, most files are already cached in local storage. Hence, requesting the updated status of cached files is more dominant than actual file transfer.
  
  ``STON`` guarantees frequently accessed files to reside in the memory. "Update check" requests are processed immediately.
  
- **Bypass**

  Some pages always contain non-cacheable areas, such as user specific pages, new postings, and replies. Even in these cases, a single domain is usually delegated to Reverse-Proxy instead of separating a page into multiple domains.
  
  ``STON`` elaborately classifies bypass targets based on various conditions. Also, the server maintains login sessions by using Origin Affinity and Private function.
  
- **Unstable Origin**

  Small or mid-size websites and personal websites cannot afford expensive equipment, infrastructures, and labor force. Origin server failure is relatively frequent and economic feasibility to improve service quality is extremely low.
  
  ``STON`` detects overload or service failure to execute automatic exclusion/recovery of the origin server. When the origin server fails, the server automatically extends TTL to decrease origin server dependency.
  
- **Image Processing**

  An identical image sometimes needs to be displayed in different ways according to user environments. Images in the search result will be displayed as thumbnails, and some websites might want their own watermark on the image. Processing every image into a specific format is a waste of storage, time, and effort.
  
  ``STON`` supports DIMS function that can generate desired image formats from a single image in the server by URL call.
  
  
File-based Caching Server
----------------------------

The edge server is built on the reverse-proxy structure. The fundamental concept of reverse-proxy is to copy/modify/manage files from the remote server to local storage. If qualified, STON can interwork with a service server, storage centralization and synchronization issues can be resolved. In addition, both service development time reduction and service reliability improvement can be attained.

.. figure:: img/icons_file.png
   :align: center

- **File I/O Support**

  If a specific protocol is required, then the server is subordinate to the module. Even if the server is interworking with the module, performance degradation could spoil everything. Therefore, I/O interface between the module and server has to be simplified.
  
  ``STON`` adopts standard File I/O. A dedicated server and STON are connected by Linux Kernel (VFS) in order to guarantee high performance.
  
- **Connect to Web Server**
  
  Standard Reverse-Proxy might not be available if any exclusive expansion module is installed on the standard web server (Apache, Lighttpd, or NginX). For example, it is hard for a file service or a payment service that is linked with DB/WAS to expand its service.
  
  ``STON`` If DocumentRoot of Apache is set to STON, Apache will perceive STON as a physical disk and nothing needs to be configured.

- **Connect to Wowza**

  In the media service field, Wowza is considered as a standard. However, the Http caching function of Wowza is not only inconvenient but also very limited. In addition, other "exclusive" protocols besides HTTP are fading away from the market.

  ``STON`` can be mounted as a local disk. Moreover, all functions--such as MP4 header convert and trimming--are available.

- **Resource Restriction**

  A server that acquires files from Back-end and delivers them to front-end users has to consider file synchronization issues. Dedicated servers like game servers and SNS servers had these problems during the developing process. These servers have to survive for a very long time without stopping the service; therefore, the use of memory and disks has to be strictly limited.

  ``STON`` can limit the use of memory and disks. In addition, when STON is mounted as a disk, all functions work in the same manner, so complicated services can be configured with a minimal solution.


STON is being developed alongside the following services that actively utilize these attributes.

.. figure:: img/intro_reference.png
   :align: center
