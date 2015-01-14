.. _bandwidth_control:

Chapter 11. Bandwidth
******************

This chapter explains various bandwidth control methods for each virtual host.
In the old days, the focus was on limiting bandwidth so it wouldn't exceed some level.
Nowadays, on the other hand, effectively controlling bandwidth is more important.
Moreover, you can analyze contents in real time to use optimized bandwidth.


.. toctree::
   :maxdepth: 2

Virtual Host Bandwidth Restriction
====================================

Limits the maximum bandwidth of virtual host.
This physcial method has the highest priority. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <TrafficCap Session="0">0</TrafficCap>   
    
-  ``<TrafficCap> (default: 0 Mbps)``
   Configure the maxinum bandwidth of virtual host in Mbps. 
   Setting this value to 0 will not limit bandwidth. 
   ``Session (default: 0 Kbps)`` property configures maximum bandwidth of each client session.

For example, if you set ``<TrafficCap>`` to 50 (Mbps), it has the same effect with having 50Mbps NIC.
The sum of bandwidth of all clients that are connected to the relative virtual host cannot exceed 50Mbps. 

``Session`` works as below.

1. Even if ``Session`` is configured, the sum of bandwidth of all clients cannot exceed ``<TrafficCap>``.
2. Even if `Bandwidth Throttling`_ is configured, the maximum speed of each client session cannot exceed ``Session``.


.. _bandwidth-control-bt:

Bandwidth Throttling
====================================

BT(Bandwidth Throttling) dynamically controls client transfer bandwidth for each session.
Media file usually includes V(Video) and A(Audio) headers as below figure.

.. figure:: img/conf_media_av.png
   :align: center
      
   Header is not a subject of BT.

Header gets bigger when the play time is longer or key frame cycle is shorter.
Therefore, if the media file can be recognized, header has to be transferred withtout limiting bandwidth for smooth playback.
BT starts after the header transfer is completed as below.

.. figure:: img/conf_bandwidththrottling2.png
   :align: center
      
   Operational Scenario
   
::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BandwidthThrottling>
      <Settings>
         <Bandwidth Unit="kbps">1000</Bandwidth>
         <Ratio>100</Ratio>
         <Boost>5</Boost>      
      </Settings>
      <Throttling>OFF</Throttling> 
   </BandwidthThrottling>   
    
``<BandwidthThrottling>`` configures default operation underneath the tag.

-  ``<Settings>``
   
   Configures default operation.
   
   -  ``<Bandwidth> (default: 1000 Kbps)``   
      configures client transfer bandwidth. 
      ``Unit`` property configures default units ( ``kbps`` , ``mbps`` , ``bytes`` , ``kb`` , ``mb`` ).
   
   -  ``<Ratio> (default: 100 %)``    
      configures ``<Bandwidth>`` property based on the ratio.
   
   -  ``<Boost> (default: 5 seconds)``   
      transfers data with unlimited speed for set amount of time.
      The amount of data can be calculated with ``<Boost>`` X ``<Bandwidth>`` X ``<Ratio>``.
         
-  ``<Throttling>``

   -  ``OFF (default)`` Does not apply BT.  
   -  ``ON`` If condition list is met, apply BT.


Bandwidth Throttling Condition List
--------------------------

Configures BT condition list.
BT is applied when the condition list is met.
Check in sequence of configuration if configurations are matching with condition.
Transfer policy is saved at /svc/{virtual host name}/throttling.txt. ::

   # /svc/www.example.com/throttling.txt
   # Comma is an identifier, and {condition},{Bandwidth},{Ratio},{Boost} format is used.
   # Other than {condition} field, everything else can be omitted.
   # Omitted field adopts default value from ``<Settings>``.
   # All expressions are identical to acl.txt setting.
   # The unit of {Bandwidth} uses ``Unit`` property of ``<Settings>`` ``<Bandwidth>``.
   
   # After transferring 3 seconds of data to the client with unlimited speed, then limit the transfer speed to 3Mbps(3000Kbps = 2000Kbps X 150%).
   $IP[192.168.1.1], 2000, 150, 3
   
   # This only defines bandwidth. Transfer 5 seconds(default) of data to the client with unlimited speed, then limit the transfer speed to 800 Kbps.
   !HEADER[referer], 800
   
   # This only defines boost. Transfer 10 seconds of data to the client with unlimited speed, then limit the transfer speed to 1000 Kbps.
   HEADER[cookie], , , 10
   
   # This does not apply BT for files with m4a extension.
   $URL[*.m4a], no

If you analyze media files(MP4, M4A, MP3), you will get bandwidth from encoding rate. 
The extension of being accessed contents should be one of .mp4, .m4a, .mp3. 
In order to extract bandwidth dynamically, append **x** to the bandwidth as below
동적으로 Bandwidth를 추출하려면 다음과 같이 Bandwidth뒤에 **x** 를 붙인다. ::

   # /vod/*.mp4 파일에 대한 접근이라면 bandwidth를 구한다. 구할 수 없다면 1000을 bandwidth로 사용한다.
   $URL[/vod/*.mp4], 1000x, 120, 5

   # user-agent헤더가 없다면 bandwidth를 구한다. 구할 수 없다면 500을 bandwidth로 사용한다.
   !HEADER[user-agent], 500x

   # /low_quality/* 파일에 대한 접근이라면 bandwidth를 구한다. 구할 수 없다면 기본 값을 bandwidth로 사용한다.
   $URL[/low_quality/*], x, 200


QueryString 우선조건
--------------------------

약속된 QueryString을 사용하여 ``<Bandwidth>`` , ``<Ratio>`` , ``<Boost>`` 를 동적으로 설정한다. 
이 설정은 BT조건보다 우선한다. 

::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <BandwidthThrottling>
      <Settings>
         <Bandwidth Param="mybandwidth" Unit="mbps">2</Bandwidth>
         <Ratio Param="myratio">100</Ratio>
         <Boost Param="myboost">3</Boost> 
      </Settings>
      <Throttling QueryString="ON">ON</Throttling>
   </BandwidthThrottling>   
    
-  ``<Bandwidth>`` , ``<Ratio>`` , ``<Boost>`` 의 ``Param``

    각각의 의미에 맞게 QueryString 키를 설정한다.
   
-  ``<Throttling>`` 의 ``QueryString``

   - ``OFF (기본)`` QueryString으로 조건을 재정의하지 않는다.
   
   - ``ON`` QueryString으로 조건을 재정의한다.
   
위와 같이 설정되어 있다면 다음과 같이 클라이언트가 요청한 URL에 따라 BT가 동적으로 설정된다. ::

    # 10초의 데이터를 속도 제한없이 전송한 후 1.3Mbps(1mbps X 130%)로 클라이언트에게 전송한다.
    http://www.winesoft.co.kr/video/sample.wmv?myboost=10&mybandwidth=1&myratio=130
    
반드시 모든 파라미터를 명시할 필요는 없다. ::

    http://www.winesoft.co.kr/video/sample.wmv?myratio=150
    
위와 같이 일부 조건이 생략된 경우 나머지 조건(여기서는 bandwidth, boost)을 결정하기 위해 조건목록을 검색한다.
여기서도 적합한 조건을 찾지 못하는 경우 ``<Settings>`` 에 설정된 기본 값을 사용한다.
QueryString이 일부 존재하더라도 조건목록에서 미적용옵션(no)이 설정되어 있다면 
BT는 적용되지 않는다.

QueryString을 사용하므로 자칫 :ref:`caching-policy-applyquerystring` 과 혼동을 일으킬 소지가 있다. 
:ref:`caching-policy-applyquerystring` 이 ``ON`` 인 경우 클라이언트가 요청한 URL의 QueryString이 
모두 인식되지만 ``BoostParam`` , ``BandwidthParam`` , ``RatioParam`` 은 제외된다. ::

   GET /video.mp4?mybandwidth=2000&myratio=130&myboost=10
   GET /video.mp4?tag=3277&myboost=10&date=20130726
        
예를 들어 위같은 입력은 BT를 결정하는데 쓰일 뿐 Caching-Key를 생성하거나 원본서버로 요청을 보낼 때는 제거된다. 
즉 각각 다음과 같이 인식된다. ::

    GET /video.mp4
    GET /video.mp4?tag=3277&date=20130726
    
    
