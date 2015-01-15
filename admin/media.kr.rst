.. _media:

Chapter 12. Media
******************

This chapter explains how to intelligently service media.
In many cases, contents are processed to various formats due to various client environments and diversity of service.
Therefore, identical contents exist in the origin server in various formats.
This method wastes processing time and storage, and moreover, it is difficult to maintain.


.. toctree::
   :maxdepth: 2



Changing MP4/M4A Header Position
====================================

Usually MP4 format, the header cannot be completed during encoding process, it will be attached at the end of file when encoding is done. 
In order to change the header position, an extra process is required.
If the header is located at the end of file, a player that does not support this format cannot Pseudo-Streaming the file.
However, simply changing header position will support Pseudo-Streaming.

The change of header position only occurs during the transfer procedure without modifying original format.
Also, it does not consume additional storage. ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>

   <UpfrontMP4Header>OFF</UpfrontMP4Header>
   <UpfrontM4AHeader>OFF</UpfrontM4AHeader>   

-  ``<UpfrontMP4Header>``
   
   - ``OFF (default)`` nothing happens.
   
   - ``ON`` If the extension is .mp4 and the header is located at the end of file, bring the header to the front of file before transfer.

-  ``<UpfrontM4AHeader>``

   - ``OFF (default)`` nothing happens.
   
   - ``ON`` If the extension is .m4a and the header is located at the end of file, bring the header to the front of file before transfer.

If the header of first requested contents needs to be moved to the front, preferentially download necessary part.
This is very intelligent and fast method.
Regardless of complicated processes in the server, clients are being serviced with a complete file that the header is located in front of file.

.. note::

   If the file is unable to analyze or damaged, it will be serviced as it is.


.. _media-trimming:

Trimming
====================================

Trimming extracts desired section based on time value.
It only trims at the transfer stage without modifying the original format.
Also, it does not consume additional storage. ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>
   
   <MP4Trimming StartParam="start" EndParam="end" AllTracks="off">OFF</MP4Trimming>
   <M4ATrimming StartParam="start" EndParam="end" AllTracks="off">OFF</M4ATrimming>   
   <MP3Trimming StartParam="start" EndParam="end">OFF</MP3Trimming>

-  ``<MP4Trimming>`` ``<MP3Trimming>`` ``<M4ATrimming>``
   
   - ``OFF (default)`` nothing happens.
   
   - ``ON`` If the extention is matching with one of these (.mp4, .mp3, .m4a), trim the file to service desired section.
     Trimming section can be configured with ``StartParam`` and ``EndParam`` attributes.
     
   - ``AllTracks`` attribute
   
     - ``OFF (default)`` trims only Audio/Video track. (Mod-H264 format)
     
     - ``ON`` trims all tracks. Player compatibility must be check before setting this.
     
Parameters can be fed via client QueryString.     
For example, if you want to trim a certain section of 10-minute video clip(/video.mp4), desired time(unit in second) can be specified in the QueryString. ::

   http://vod.wineosoft.co.kr/video.mp4                // 10 minutes : trim entire video clip
   http://vod.wineosoft.co.kr/video.mp4?end=60         // 1 minute : trim from 0 to 60 second section
   http://vod.wineosoft.co.kr/video.mp4?start=120      // 8 minutes : trim from 0 to 120 second section
   http://vod.wineosoft.co.kr/video.mp4?start=3&end=13 // 10 seconds : trim from 3 second to 13 second section

If ``StartParam`` value is larger than ``EndParam`` value, it is considered as undefined section.
This feature is developed to support skip function of a video player that is based on HTTP Pseudo-Streaming. 
Therefore, STON recognizes keyframe and time to extract section from the original file so the video can be played properly
instead of trimming the original file based on the offset like when the Range request is processed. 

The file that will be relayed to the client is the complete form of MP4 file that has a recreated MP4 header as below.

.. figure:: img/conf_media_mp4trimming.png
   :align: center
      
   Complete form of file is transferred.

The extracted section is recognized as a separate file, therefore, 200 OK will be responded. 
If the Range header is specified as below, Range will be calculated from extracted file to respond **206 Particial Content**.

.. figure:: img/conf_media_mp4trimming_range.png
   :align: center
      
   It is processed just like general Range request.
   
QueryString expression is used for trimming parameters so there could be a confusion with :ref:`caching-policy-applyquerystring`. 
If ``<ApplyQueryString>`` is set to ``ON``, all querystrings of client request URL are recognized, but ``StartParam`` and ``Endparam`` are removed. ::

   GET /video.mp4?start=30&end=100
   GET /video.mp4?tag=3277&start=30&end=100&date=20130726
    
From above example, if **start** and **end** are entered for ``StartParam`` and ``EndParam`` respectively,
these values are only used when extracting section and will be removed when creating Caching-Key or sending a request to the origin server. 
Above examples are recognized as below. ::

   GET /video.mp4
   GET /video.mp4?tag=3277&date=20130726
    
Also, QueryString parameters can be differ by expansion modules or CDN solutions. 

.. figure:: img/conf_media_mp4trimming_range.png
   :align: center
   
   Module/CDN references that are provided from JW Player
   
In addition, both `ngx_http_mp4_module <http://nginx.org/en/docs/http/ngx_http_mp4_module.html>`_ of nginx and 
`Mod-H264-Streaming-Testing-Version2 <http://h264.code-shop.com/trac/wiki/Mod-H264-Streaming-Testing-Version2>`_ of lighttpd
use **start** as QueryString.


.. _media-hls:

HLS (HTTP Live Streaming)
====================================

MP4 files are serviced with HLS(HTTP Live Streaming). 
The origin server does not need to split files for HLS service any more. 
Regardless of the location of MP4 file header, real time conversion to .m3u8/.ts occurs when downloading the file. 

..  note::

    MP4HLS is not a transcoding that converts elementary streams(Video or Audio). 
    If the file is not properly encoded, the file might not played properly.
    Current(2014.2.20) video/audio encoding format from Apple is as below.
    그러므로 HLS에 적합한 형식으로 인코딩된 MP4파일에 한해서 원활한 단말 재생(단말 재생?? 단말기/모바일기기에서 재생??)이 가능하다. 
    인코딩이 적합하지 않을 경우 화면이나 깨지거나 소리가 재생되지 않을 수 있다. 
    현재(2014.2.20) Apple에서 밝히고 있는 Video/Audio 인코딩 규격은 다음과 같다.

    What are the specifics of the video and audio formats supported?
    Although the protocol specification does not limit the video and audio formats, the current Apple implementation supports the following formats:
    
    [Video]
    H.264 Baseline Level 3.0, Baseline Level 3.1, Main Level 3.1, and High Profile Level 4.1.
    
    [Audio]
    HE-AAC or AAC-LC up to 48 kHz, stereo audio
    MP3 (MPEG-1 Audio Layer 3) 8 kHz to 48 kHz, stereo audio
    AC-3 (for Apple TV, in pass-through mode only)
    
    Note: iPad, iPhone 3G, and iPod touch (2nd generation and later) support H.264 Baseline 3.1. If your app runs on older versions of iPhone or iPod touch, however, you should use H.264 Baseline 3.0 for compatibility. If your content is intended solely for iPad, Apple TV, iPhone 4 and later, and Mac OS X computers, you should use Main Level 3.1.	
   

Existing method requires original files for Pseudo-Streaming and HLS as below. 
In this case, STON also duplicates original files to service clients. 
However, as the play time gets longer, more derived files will be created, and it will get harder to manage.

.. figure:: img/conf_media_mp4hls1.png
   :align: center
   
   Laborious HLS
   
``<MP4HLS>`` dynamically creates required files from original files for HLS service.

.. figure:: img/conf_media_mp4hls2.png
   :align: center
   
   Intelligent HLS

All .m3u8/.ts files are derived from original files and do not consume additional storage. 
The file is temporarily created on the memory when it is being serviced, and automatically discarded when it is not serviced. ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>

   <MP4HLS Status="Inactive" Keyword="mp4hls">
      <Index>index.m3u8</Index>
      <Sequence>0</Sequence>
      <Duration>10</Duration>
   </MP4HLS>   
    
-  ``<MP4HLS>``   
   is activated when ``Status`` attribute is ``Active``.

For example, if the service address is following, Pseudo-Streaming can be processed to the address. ::

    http://www.example.com/video.mp4
    
Virtual host recognizes ``Keyword`` string that is defined in the ``<MP4HLS>`` in order to proceed HLS service. 
If the following URL is called, index.m3u8 file is created from /video.mp4 file. 
The index file name can be configured with a string in the ``<Index>``. ::

   http://www.example.com/video.mp4/mp4hls/index.m3u8
    
Generated index.m3u8 looks like below. ::

   #EXTM3U
   #EXT-X-TARGETDURATION: 10
   #EXT-X-MEDIA-SEQUENCE: 0
   #EXTINF:10,
   /video.mp4/mp4hls/0.ts
   #EXTINF:10,
   /video.mp4/mp4hls/1.ts
   #EXTINF:10,
   /video.mp4/mp4hls/2.ts
   
   ... (skip)...
    
   #EXTINF:10,
   /video.mp4/mp4hls/161.ts
   #EXTINF:9,
   /video.mp4/mp4hls/162.ts
   #EXT-X-ENDLIST
    
#EXT-X-TARGETDURATION can be configured with ``<Duration>``.
The original file can only be splited by KeyFrame of Video.
The following four cases can exist.

-  The **KeyFrame** interval is smaller than the ``<Duration>`` value.   
   If the KeyFrame is 3 seconds and the ``<Duration>`` is 20 seconds,
   the largest multiple of 3(KeyFrame) that does not exceeding 20(Duration) will be used to split the video (18 seconds in this case).
   
-  The **KeyFrame** interval is close to the ``<Duration>`` value.   
   If the KeyFrame is 9 seconds and the ``<Duration>`` is 10 seconds,
   the largest multiple of 9(KeyFrame) within 10(Duration) will be used to split the video (9 seconds in this case).
   
-  The **KeyFrame** interval is bigger than the ``<Duration>`` value.
   The video is splitted using KeyFrame.
   
-  **Video** is not exist.
   ``<Duration>`` is used to split the video.
   
#EXT-X-MEDIA-SEQUENCE defines starting number of .ts file and configured with ``<Sequence>``.

Let's take a look at the following client request and see how STON responses to it. ::

   GET /video.mp4/mp4hls/99.ts HTTP/1.1
   Range: bytes=0-512000
   Host: www.wineosft.com

1.	``STON`` Initial loading. (Nothing has been cached yet.)
#.	``Client`` HTTP Range request. (Requests the first 500KB of 100th file.)     
#.	``STON`` creates caching object of /video.mp4 file.
#.	``STON`` downloads necessary portion from the origin server to analyze /video.mp4 file.
#.	``STON`` downloads necessary portion from the origin server to service 100th(99.ts) file.
#.	``STON`` creates 100th(99.ts) file and proceed Range service.
#.	``STON`` discards the 99.ts file after completing service.



.. _media-dims:

DIMS
====================================

DIMS(Dynamic Image Management System) function processes an original image to various forms. 
DIMS is an expansion based on `mod_dims <https://code.google.com/p/moddims/wiki/WebserviceApi>`_. 
Total 6 forms(crop, thumbnail, resize, reformat, quality, composite) are available, and these forms can be complexed.

.. figure:: img/dims.png
   :align: center
   
   Various dynamic image processing

이미지는 동적으로 생성되며 원본 이미지 URL뒤에 약속된 키워드와 가공옵션을 붙여서 호출한다. 
가공된 이미지는 캐싱되어 원본서버 이미지가 바뀌지 않는 이상 다시 가공되지 않는다. 
예를 들어 원본 파일이 /img.jpg라면 다음과 같은 형식으로 이미지를 가공할 수 있다. 
("12AB"는 약속된 Keyword이다.) ::

   http://image.example.com/img.jpg    // 원본 이미지
   http://image.example.com/img.jpg/12AB/resize/500x500/
   http://image.example.com/img.jpg/12AB/crop/400x400/
   http://image.example.com/img.jpg/12AB/composite/watermark1/

``<Dims>`` 는 별도로 설정하지 않으면 모두 비활성화되어 있다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" Port="8500" />

-  ``<Dims>``

   - ``Status`` DIMS활성화 ( ``Active`` 또는 ``Inactive`` )   
   - ``Keyword`` 원본과 DIMS를 구분하는 키워드   
   - ``Port`` WM접속포트
   
``<Dims>`` 동작을 위해서는 :ref:`wm` 이 반드시 동작하고 있어야 한다.


잘라내기
-----------------------

좌상단을 기준으로 원하는 영역만큼 이미지를 잘라낸다. 
영역은 **widthxheight{+-}x{+-}y{%}** 로 표현한다. 
다음은 좌상단 x=20, y=30을 기준으로 width=100, height=200만큼 잘라내는 예제다. ::

   http://image.example.com/img.jpg/dims/crop/100x300+20+30/
    

Thumbnail 생성
-----------------------

Thumbnail 을 생성한다. 
크기와 옵션은 **widthxheight{%} {@} {!} {<} {>}** 로 표현한다.
기본적으로 이미지의 가로와 세로는 최대값을 사용한다. 
이미지를 확대 또는 축소하여도 가로 세로 비율은 유지된다. 
정확하게 지정한 크기로 이미지를 조절할 때는 크기 뒤에 느낌표(!)를 추가한다. 
**640X480!** 라는 표현은 정확하게 640x480 크기의 Thumbnail을 생성한다는 뜻이다. 
만약 가로 또는 세로 크기만 지정된 경우, 생략된 값은 가로/세로 비율에 의해 자동결정 된다. 

예를 들어 **/thumbnail/100/** 은 가로 크기에 맞추어 세로 크기가 결정되며 
**/thumbnail/x200/** 은 세로 크기에 맞추어 가로 크기가 결정된다. 
가로/세로 크기를 이미지의 크기에 맞추어 백분율(%)로 표현할 수 있다. 
이미지 크기를 늘리려면, 100 보다 큰 값(예 : 125 %)을 사용한다. 
이미지 크기를 줄이려면 100 미만의 비율을 사용한다.
URL Encoding규칙에 따라 %문자가 %25로 인코딩 됨을 명심해야 한다. 

예를 들어 50%라는 표현은 50%25로 인코딩 된다. 
다음은 width=78, height=110크기의 Thumbnail을 생성하는 예제다. ::

   http://image.example.com/img.jpg/dims/thumbnail/78x110/
    

Resizing
-----------------------

이미지 크기를 변경한다. 
크기는 **width x height** 로 표현한다. 
이미지는 변경되어도 비율은 유지된다. 
다음은 원본 이미지를 width=200, height=200크기로 변경하는 예제다. ::

   http://image.example.com/img.jpg/dims/resize/200x200/


Format 변경
-----------------------

이미지 포맷을 변경한다. 
지원되는 포맷은 "png", "jpg", "gif" 이다. 
다음은 JPG를 PNG로 변환하는 예제다. ::

   http://image.example.com/img.jpg/dims/format/png/


품질 변경
-----------------------

이미지 품질을 조절한다. 
이 기능은 전송되는 이미지 용량을 줄일 수 있어서 효과적이다. 
유효 범위는 0부터 100까지다. 
다음은 이미지 품질을 25%로 조절하는 예제다. ::

   http://image.example.com/img.jpg/dims/quality/25/
  

합성
-----------------------

두 이미지를 합성한다.
앞서 설명한 기능과는 다르게 합성조건은 미리 설정되어 있어야 한다. 
주로 워터마크 효과를 내기 위해 사용된다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" port="8500">
      <Composite Name="water1" File="/img/small.jpg" />
      <Composite Name="water2" File="/img/medium.jpg" Gravity="se" Geometry="+0+0" Dissolve="50" />
      <Composite Name="water_ratio" File="/img/wmark_s.png" Gravity="s" Geometry="+0+15%" Dissolve="100" />
   </Dims>   
    
-  ``<Composite>``

    이미지 합성조건을 설정한다. 속성에 의해 정해지며 별도의 값을 가지지 않는다.
    
    -  ``Name`` 호출될 이름을 지정한다. 
       '/'문자는 입력할 수 없다. 
       URL의 "/composite/" 뒤에 위치합니다.
       
    -  ``File`` 합성할 이미지파일 경로를 지정한다. 
    
    -  ``Gravity (기본: c)`` 합성할 위치는 좌측상단부터 9가지의 포인트(nw, n, ne, w, c, e, sw, s, se)가 존재합니다.
       
       .. figure:: img/conf_dims2.png
          :align: center
       
          Gavity 기준점
          
    -  ``Geometry (기본: +0+0)`` ``Gravity`` 기준으로 합성할 이미지 위치를 설정한다. 
       {+-}x{+-}y. 붉은색 원은 Gravity속성에 따라 +0+0이 의미하는 기준점으로 +x+y의 
       값이 커질수록 이미지 안쪽으로 배치된다. 
       초록색 화살표는 +x, 보라색 화살표는 +y가 증가하는 방향이다. 
       -x-y를 사용하면 대상 이미지의 바깥에 위치하게 되어 결과 이미지에서는 보여지지 않는다. 
       이 속성은 다소 복잡해 보이지만 이미지 크기를 자동으로 계산하여 배치하므로 
       일관된 결과물을 얻을 수 있어서 효과적이다. 
       또한 +x%+y% 처럼 %옵션을 주어 비율로 배치할 수도 있다.

    -  ``Dissolve (기본: 50)`` 합성할 이미지의 투명도(0~100).

``<Composite>`` 을 설정했다면 ``Name`` 속성을 사용하여 이미지를 합성할 수 있다. ::

    http://image.example.com/img.jpg/dims/composite/water1/

    

원본이미지 조건판단
-----------------------

원본 이미지 조건에 따라 동적으로 가공 옵션을 다르게 적용할 수 있다. 
예를 들어 1024 X 768 이하의 이미지는 품질을 50%로 떨어트리고 그 이상의 
이미지는 1024 X 768로 크기변환을 하려면 다음과 같이 ``<ByOriginal>`` 을 설정한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Dims Status="Active" Keyword="dims" port="8500">
      <ByOriginal Name="size1">
         <Condition Width="1024" Height="768">/quality/25/</Condition>
         <Condition>/resize/1024x768/</Condition>
      </ByOriginal>
   </Dims>   

-  ``<ByOriginal>``
   ``Name`` 속성으로 호출한다. 
   하위에 다양한 조건의 ``<Condition>`` 을 설정한다.
   
-  ``<Condition>``
   조건을 만족할 때 설정된 값을 적용한다. 
   설정하는 속성은 "상관없음"으로 판단한다.

   -  ``Width`` 가로길이가 설정 값보다 작으면 적용된다.   
   -  ``Heigth`` 세로길이가 설정 값보다 작으면 적용된다.

``<Condition>`` 은 명시된 순서대로 적용된다. 
그러므로 작은 이미지 조건을 먼저 배치해야 한다.
다음과 같이 호출한다. ::

   http://image.example.com/img.jpg/dims/byoriginal/size1/
    
또 다른 예로 이미지 크기에 따라 다른 ``<Composite>`` 조건을 줄 수 있다. 
이런 경우 다음과 같이 사전에 정의된 ``<Composite>`` 의 ``Name`` 으로 설정한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Dims Status="Active" Keyword="dims" port="8500">
      <Composite Name="water1" File="/img/small.jpg" />
      <Composite Name="water2" File="/img/medium.jpg" Gravity="se" Geometry="+0+0" Dissolve="50" />
      <Composite Name="water3" File="/img/big.jpg" Gravity="se" Geometry="+10+10" Dissolve="50" />
      <ByOriginal Name="size_water">
         <Condition Width="400">/composite/water1/</Condition>
         <Condition Width="800">/composite/water2/</Condition>
         <Condition>/composite/water3/</Condition>
      </ByOriginal>
   </Dims>   
    
다음과 같이 호출하면 원본 이미지 크기에 따라 합성이 적용된다. ::

   http://image.example.com/img.jpg/dims/byoriginal/size_water/



기타
-----------------------

이상의 기본기능을 결합하여 복합적인 이미지 가공을 할 수 있다. 
예를 들어 Thumbnail생성(78x110), 포맷을 JPG에서 PNG로 변환, 품질 50% 이상의 옵션을 한번의 호출로 실행할 수 있다. ::

   http://image.example.com/img.jpg/dims/thumbnail/78x110/format/png/quality/50/
    
DIMS는 URL을 이용하여 이미지 가공이 이루어진다. 
그러므로 URL에 영향을 주는 다른 옵션들 때문에 원하지 않는 결과가 얻어지지 않도록 주의해야 한다.

-  :ref:`caching-policy-applyquerystring` 이 ``OFF`` 라면 키워드 이전의 QueryString이 무시된다. ::
   
      http://image.example.com/img.jpg?session=5234&type=37/dims/resize/200x200/
      
   위와 같은 호출에 이 설정이 ``ON`` 이라면 입력된 URL 그대로 인식되지만 OFF라면 다음과 같이 인식된다. ::
      
      http://image.example.com/img.jpg/dims/resize/200x200/
      
-  :ref:`caching-policy-casesensitive` 이 ``OFF`` 라면 모든 URL을 소문자로 변환하여 처리한다.
   그러므로 DIMS 키워드에 대문자가 포함되었다면 키워드를 인식하지 못한다. 
   항상 키워드는 소문자로 사용하는 것이 좋다.
