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
    Current(2014.2.20) video/audio encoding format by Apple is shown as below.
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

Dynamically generated image is called by pre-defined keywords and process options after the URL of original image. 
Processed image is cached as it is until the image of the origin server is modified. 
For example, the following expressions can process /img.jpg file in the origin server. 
(Let's say "12AB" is a pre-defined keyword.) ::

   http://image.example.com/img.jpg    // Original image
   http://image.example.com/img.jpg/12AB/resize/500x500/
   http://image.example.com/img.jpg/12AB/crop/400x400/
   http://image.example.com/img.jpg/12AB/composite/watermark1/

If ``<Dims>`` is not configured, it is inactivated. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" Port="8500" />

-  ``<Dims>``

   - ``Status`` activates DIMS ( ``Active`` or ``Inactive`` )   
   - ``Keyword`` distinguishes the original and DIMS   
   - ``Port`` WM connection port
   
In order to utilize ``<Dims>``, :ref:`wm` must be running.


Crop
-----------------------

Crop desired section of an image from top left corner. 
Section is specified by **widthxheight{+-}x{+-}y{%}** format. 
The following example crops 100(width) by 200(height) section of image from x=20, y=30. ::

   http://image.example.com/img.jpg/dims/crop/100x300+20+30/
    

Generating Thumbnails
-----------------------

This section explains how to generate thumbnails. 
The size and option can be expressed with **widthxheight{%} {@} {!} {<} {>}**.
Normally, maximum value is applied for width and height of the image. 
Even if the image is stretched or shrunk, the width and height ratio is maintained. 
If the image has to have a specific size, an exclamation mark(!) is added at the end of the expression. 
**640X480!** expression will generate 640x480 size thumbnail image. 
If either width or height is specified only, the omitted value is automatically determined by the width and height ratio. 

For example, **/thumbnail/100/** expression will determine the height of thumbnail with the width attribute, 
and **/thumbnail/x200/** expression will determine the width of thumbnail by the height attribute. 
The size of thumbnail can be expressed with percentage(%) of the original image. 
In order to stretch the image, use a value that exceeds 100(eg. 125%). 
Likewise, in order to shrink the image, use a value less than 100.
You should be aware that the URL encoding rule encodes % character as %25. 

For example, 50% expression will be encoded as 50%25. 
The below is an example that generates a thumbnail of width=78 and height=110. ::

   http://image.example.com/img.jpg/dims/thumbnail/78x110/
    

Resizing
-----------------------

This function resizes the image size. 
The size of image is expressed with **width x height** format. 
The ratio will be kept the same after the resizing. 
The below is an example that resizes an original image to width=200 and height=200. ::

   http://image.example.com/img.jpg/dims/resize/200x200/


Converting Format
-----------------------

This function converts the image format. 
Supported formats are "png", "jpg", "gif". 
The following is an example that converts JPG to PNG. ::

   http://image.example.com/img.jpg/dims/format/png/


Adjusting Image Quality
-----------------------

This function adjusts the image quality. 
It is useful as it can reduce the volume of transferring image. 
Effective values are from 0 to 100. 
The following example will adjust the quality of image to 25%. ::

   http://image.example.com/img.jpg/dims/quality/25/
  

Image Synthesis
-----------------------

This function systhesize two images.
Not like the previous functions, the image composite condition has to be pre-determined. 
This function is helpful when applying watermark on the image.. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" port="8500">
      <Composite Name="water1" File="/img/small.jpg" />
      <Composite Name="water2" File="/img/medium.jpg" Gravity="se" Geometry="+0+0" Dissolve="50" />
      <Composite Name="water_ratio" File="/img/wmark_s.png" Gravity="s" Geometry="+0+15%" Dissolve="100" />
   </Dims>   
    
-  ``<Composite>``

    Configure the image composite condition. The condition is determined by attributes.
    
    -  ``Name`` designates a name to be called. 
       '/' character cannot be used. 
       This option is located behind the "/composite/" of URL.
       
    -  ``File`` designates a path of the image to be synthesized. 
    
    -  ``Gravity (default: c)`` There are 9 composite points (nw, n, ne, w, c, e, sw, s, se).
       
       .. figure:: img/conf_dims2.png
          :align: center
       
          Gavity Reference Points
          
    -  ``Geometry (default: +0+0)`` Based on the ``Gravity``, ``Geometry`` decides where the composite image is located at. 
       {+-}x{+-}y. The red dots are reference points that stands for +0+0 of the ``Gravity`` attribute.
       붉은색 원은 Gravity속성에 따라 +0+0이 의미하는 기준점으로 +x+y의 
       값이 커질수록 이미지 안쪽으로 배치된다(??값이 커질수록 이미지 안쪽으로 배치되는 것 같지않아 혼동되어 생략했습니다).
       The green arrow stands for increasing x axis, and the purple one stands for increasing y axis. 
       Using -x-y will locate the composite image outside of the boundary and will be invisible in the result image. 
       This method looks quite complicated, but it automatically calculates the size of image so results are consistent. 
       In addition, you can use a percentage option(%) like +x%+y% to locate the image.

    -  ``Dissolve (default: 50)`` Transparency of the image to be synthesized(0~100).

If you configure the ``<Composite>``, ``Name`` property can be used to synthesize image. ::

    http://image.example.com/img.jpg/dims/composite/water1/

    

Conditional Process of Original Image
-----------------------

You can dynamically apply different processing options based on the condition of original image. 
For example, if you want to decrease the image quality by 50% for images that are smaller than 1024 X 768, 
and resize the image to 1024 X 768 for images that are bigger than 1024 X 768, ``<ByOriginal>`` can be used as below. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Dims Status="Active" Keyword="dims" port="8500">
      <ByOriginal Name="size1">
         <Condition Width="1024" Height="768">/quality/25/</Condition>
         <Condition>/resize/1024x768/</Condition>
      </ByOriginal>
   </Dims>   

-  ``<ByOriginal>``
   can be called by ``Name`` attribute. 
   Various ``<Condition>`` can be configured.
   
-  ``<Condition>``
   When condition is met, apply configured value. 
   ("상관없음"으로 판단한다는게 무슨뜻인지??)설정하는 속성은 "상관없음"으로 판단한다.

   -  ``Width`` The condition is met if the width is smaller than this value.   
   -  ``Heigth`` The condition is met if the height is smaller than this value.

``<Condition>`` is applied in the order. 
Therefore specific conditions have to come first.
The function can be called as following. ::

   http://image.example.com/img.jpg/dims/byoriginal/size1/
    
Different ``<Composite>`` conditions can be applied based on the size of image.
The following example shows how to set various pre-defined ``Name`` in ``<Composite>``. ::

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
    
The following expression will generate composite image based on the size of original image. ::

   http://image.example.com/img.jpg/dims/byoriginal/size_water/



ETC
-----------------------

Combining above functions will allow you to generate complex image. 
For example, generating thumbnail(78x110), JPG format converting to PNG, and 50% image quality options can be processed with a single call. ::

   http://image.example.com/img.jpg/dims/thumbnail/78x110/format/png/quality/50/
    
DIMS uses URL for image processing. 
Therefore, you should be careful for other options that might affect to the URL to avoid any unexpected result.

-  If :ref:`caching-policy-applyquerystring` is set to ``OFF``, the QueryString prior to the keyword will be ignored. ::
   
      http://image.example.com/img.jpg?session=5234&type=37/dims/resize/200x200/
      
   The above example will be identified as it is if the attribute is ``ON``.
   However, if the attribute is ``OFF``, the above URL will be identified as below. ::
      
      http://image.example.com/img.jpg/dims/resize/200x200/
      
-  If :ref:`caching-policy-casesensitive` is set to ``OFF``, all characters in the URL will be converted to lower characters.
   Therefore, upper characters are included in the DIMS keyword, the keyword will not be recognized. 
   Using lower characters for the keyword is recommended.
