.. _media:

Chapter 15. Media
******************

This chapter explains how to intelligently service media.
In many cases, content is processed in various formats due to various client environments and the diversity of service.
Therefore, identical content exists in the origin server in various formats.
This method wastes processing time and storage; moreover, it is difficult to maintain.


.. toctree::
   :maxdepth: 3



Reordering MP4/M4A Header
====================================

Usually in MP4 format, the header cannot be completed during the encoding process; instead, it will be attached at the end of the file when encoding is done. 
In order to change the header position, an extra process is required.
If the header is located at the end of the file, a player that does not support this format cannot Pseudo-Stream the file.
However, simply changing the header position will support Pseudo-Streaming.

This change of a header position only occurs during the transfer procedure and will not modify the original format.
Also, it does not consume additional storage. ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>

   <UpfrontMP4Header>OFF</UpfrontMP4Header>
   <UpfrontM4AHeader>OFF</UpfrontM4AHeader>   

-  ``<UpfrontMP4Header>``
   
   - ``OFF (default)`` Nothing happens.
   
   - ``ON`` If the extension is .mp4 and the header is located at the end of the file, the header will be removed to the front of file before transfer.

-  ``<UpfrontM4AHeader>``

   - ``OFF (default)`` Nothing happens.
   
   - ``ON`` If the extension is .m4a and the header is located at the end of file, the header will be moved to the front of file before transfer.

If the header of the content that was requested first needs to be moved to the front, preferentially download the necessary part.
This is a smart and fast method because clients are being serviced with a complete file, regardless of complicated processes in the server.

.. note::

   If the file is damaged or unable to analyze, it will be serviced as it is.


.. _media-trimming:

Trimming
====================================

Trimming extracts desired sections based on a time value.
It only trims during the transfer stage without modifying the original format.
Also, it does not consume additional storage. ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>
   
   <MP4Trimming StartParam="start" EndParam="end" AllTracks="off">OFF</MP4Trimming>
   <M4ATrimming StartParam="start" EndParam="end" AllTracks="off">OFF</M4ATrimming>   
   <MP3Trimming StartParam="start" EndParam="end">OFF</MP3Trimming>

-  ``<MP4Trimming>`` ``<MP3Trimming>`` ``<M4ATrimming>``
   
   - ``OFF (default)`` Nothing happens.
   
   - ``ON`` If the extension is mp4, .mp3, or .m4a, the file will be trimmed to the desired section..
     Configure which sections to be trimmed with ``StartParam`` and ``EndParam`` attributes.
     
   - ``AllTracks`` attribute
   
     - ``OFF (default)`` Trims only Audio/Video tracks. (Mod-H264 format)
     
     - ``ON`` Trims all tracks. Before using this setting, player compatibility must be checked.
     
Parameters can be fed via the client QueryString.     
For example, if you want to trim a certain section of a 10-minute video clip(/video.mp4), the desired time(units in second) can be specified in the QueryString. ::

   http://vod.wineosoft.co.kr/video.mp4                // 10 minutes : trim entire video clip
   http://vod.wineosoft.co.kr/video.mp4?end=60         // 1 minute : trim from 0 to 60 second section
   http://vod.wineosoft.co.kr/video.mp4?start=120      // 8 minutes : trim from 0 to 120 second section
   http://vod.wineosoft.co.kr/video.mp4?start=3&end=13 // 10 seconds : trim from 3 second to 13 second section

If the ``StartParam`` value is larger than the ``EndParam`` value, it is considered as an undefined section.
This feature is developed to support the skip function of a video player that is based on HTTP Pseudo-Streaming. 
STON recognizes the keyframe and the time to extract the section from the original file so the video can be played properly
instead of trimming the original file based on the offset, like when a Range request is processed. 

The file that will be relayed to the client is in the complete form of an MP4 file that has a recreated MP4 header, as shown below.

.. figure:: img/conf_media_mp4trimming.png
   :align: center
      
   The complete form of the file is transferred.

The extracted section is recognized as a separate file; therefore, the response 200 OK will be given. 
If the Range header is specified as shown below, the Range will be calculated from the extracted file to respond **206 Partial Content**.

.. figure:: img/conf_media_mp4trimming_range.png
   :align: center
      
   It is processed just like a general Range request.
   
A QueryString expression is used for trimming parameters so there could possibly be confusion with :ref:`caching-policy-applyquerystring`. 
If the ``<ApplyQueryString>`` is set to ``ON``, all QueryStrings of a client request URL are recognized, but ``StartParam`` and ``Endparam`` are removed. ::

   GET /video.mp4?start=30&end=100
   GET /video.mp4?tag=3277&start=30&end=100&date=20130726
    
In the above example, if **start** and **end** are entered for ``StartParam`` and ``EndParam`` respectively.
These values are only used when extracting a section and will be removed when creating a Caching-Key or sending a request to the origin server. 
The above examples are recognized as indicated below. ::

   GET /video.mp4
   GET /video.mp4?tag=3277&date=20130726
    
Also, QueryString parameters can differ by expansion modules or CDN solutions. 

.. figure:: img/conf_media_mp4trimming_range.png
   :align: center
   
   Module/CDN references that are provided from the JW Player
   
In addition, both `ngx_http_mp4_module <http://nginx.org/en/docs/http/ngx_http_mp4_module.html>`_ of nginx and 
`Mod-H264-Streaming-Testing-Version2 <http://h264.code-shop.com/trac/wiki/Mod-H264-Streaming-Testing-Version2>`_ of lighttpd
use **start** as a QueryString.


.. _media-hls:

HLS (HTTP Live Streaming)
====================================

MP4 files are serviced with HLS(HTTP Live Streaming). 
The origin server does not need to split files for HLS service any more. 
Regardless of the location of the MP4 file header, real time conversion to .m3u8/.ts occurs when downloading the file. 

..  note::

    MP4HLS is not a transcoding that converts elementary streams(video or audio). 
    Therefore, encoded MP4 files that only follows HLS format can be played on the mobile devices without a problem.
    If the file is not properly encoded, the file might not play properly.
    The current(2014.2.20) video/audio encoding format by Apple is shown below.

    What are the specifics of the video and audio formats that are supported?
    Although the protocol specification does not limit video and audio formats, the current Apple implementation supports only the following formats:
    
    [Video]
    H.264 Baseline Level 3.0, Baseline Level 3.1, Main Level 3.1, and High Profile Level 4.1.
    
    [Audio]
    HE-AAC or AAC-LC up to 48 kHz, stereo audio
    MP3 (MPEG-1 Audio Layer 3) 8 kHz to 48 kHz, stereo audio
    AC-3 (for Apple TV, in pass-through mode only)
    
    Note: iPad, iPhone 3G, and iPod touch (2nd generation and later) support H.264 Baseline 3.1. If your app runs on older iPhone or iPod touch versions, however, you should use H.264 Baseline 3.0 for compatibility. If your content is intended solely for iPad, Apple TV, iPhone 4 and later, and Mac OS X computers, you should use Main Level 3.1.	
   

The existing method requires original files for Pseudo-Streaming and HLS, as shown below. 
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
The file is temporarily created in memory when it is being serviced and automatically discarded when discarded thereafter. ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>

   <MP4HLS Status="Inactive" Keyword="mp4hls">
      <Index>index.m3u8</Index>
      <Sequence>0</Sequence>
      <Duration>10</Duration>
   </MP4HLS>   
    
-  ``<MP4HLS>``   
   is activated when the ``Status`` attribute is ``Active``.

For example, if the service address is as follows, Pseudo-Streaming can be processed to the address. ::

    http://www.example.com/video.mp4
    
The virtual host recognizes the ``Keyword`` string that is defined in the ``<MP4HLS>`` in order to proceed with the HLS service. 
If the following URL is called, an index.m3u8 file is created from the /video.mp4 file. 
The index file name can be configured with a string in the ``<Index>``. ::

   http://www.example.com/video.mp4/mp4hls/index.m3u8
    
A generated index.m3u8 looks like the example below. ::

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
The original file can only be split by a KeyFrame of Video.
The following four cases can exist:

-  The **KeyFrame** interval is smaller than the ``<Duration>`` value.   
   If the KeyFrame is 3 seconds and the ``<Duration>`` is 20 seconds,
   the largest multiple of 3(KeyFrame) that does not exceed 20(Duration) will be used to split the video (18 seconds in this case).
   
-  The **KeyFrame** interval is close to the ``<Duration>`` value.   
   If the KeyFrame is 9 seconds and the ``<Duration>`` is 10 seconds,
   the largest multiple of 9(KeyFrame) within 10(Duration) will be used to split the video (9 seconds in this case).
   
-  The **KeyFrame** interval is bigger than the ``<Duration>`` value.
   The video is split using the KeyFrame.
   
-  **Video** does not exist.
   ``<Duration>`` is used to split the video.
   
#EXT-X-MEDIA-SEQUENCE defines the starting number of the .ts file and configures with ``<Sequence>``.

Let's take a look at the following client request and see how STON would respond. ::

   GET /video.mp4/mp4hls/99.ts HTTP/1.1
   Range: bytes=0-512000
   Host: www.wineosft.com

1.	``STON`` Initial loading. (Nothing has been cached yet.)
#.	``Client`` HTTP Range request. (Requests the first 500KB of 100th file.)     
#.	``STON`` creates a caching object of the /video.mp4 file.
#.	``STON`` downloads the necessary portion from the origin server to analyze the /video.mp4 file.
#.	``STON`` downloads the necessary portion from the origin server to service the 100th(99.ts) file.
#.	``STON`` creates the 100th(99.ts) file and proceeds with the Range service.
#.	``STON`` discards the 99.ts file after completing the service.



.. _media-dims:

DIMS
====================================

The DIMS(Dynamic Image Management System) function processes an original image into various forms. 
DIMS is an expansion based on `mod_dims <https://code.google.com/p/moddims/wiki/WebserviceApi>`_. 
A total of six forms(crop, thumbnail, resize, reformat, quality, and composite) are available, and these forms can be combined.

.. figure:: img/dims.png
   :align: center
   
   Various dynamic image processing

A dynamically generated image is called by pre-defined keywords and process options after the URL of an original image. 
The processed image is cached as it is until the image in the origin server is modified. 
For example, the following expressions can process an /img.jpg file in the origin server. 
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

   - ``Status`` Activates DIMS ( ``Active`` or ``Inactive`` )   
   - ``Keyword`` Distinguishes the original and DIMS   
   - ``MaxSourceSize (default: 10 megabytes)`` Maximum original image size (in megabytes)
   - ``OnFailure`` STON may return messages or redirect to the origin in case of image conversion failure.   
     
     - ``message (default)`` Returns 500 Internal Error. STON may indicate the detail as one of the followings:
     
       - ``The original file was not successfully downloaded.`` The original image download is incomplete.
       - ``The original file size is too large.`` The original image size is bigger than ``MaxSourceSize`` value.
       - ``The original file loading failed.`` The originial image is not loaded.
       - ``Image converting failed or invalid DIMS command.`` Unsupported image or invalid command.
     
     - ``redirect`` Redirects to the original image URL.
   

Optimization
-----------------------

Optimization reduces the image size while maintaining its quality.
Only JPEG, JPEG-2000 and Lossless-JPEG are supported.
Some original images may have little or no size reduction, if optimized by any other tool or application already. ::

   http://image.example.com/img.jpg/dims/optimize

Optimization command is recommended to be placed at the end of command for the desired conversion. ::

   http://image.example.com/img.jpg/dims/resize/100x100/optimize
   
Optimization may consume vast system recource. 
The following is a performance test result for sample-size images, for 0% HitRatio condition.

-  ``OS`` CentOS 6.2 (Linux version 2.6.32-220.el6.x86_64 (mockbuild@c6b18n3.bsys.dev.centos.org) (gcc version 4.4.6 20110731 (Red Hat 4.4.6-3) (GCC) ) #1 SMP Tue Dec 6 19:48:22 GMT 2011)
-  ``CPU`` `Intel(R) Xeon(R) CPU E3-1230 v3 @ 3.30GHz (8 processors) <http://www.cpubenchmark.net/cpu.php?cpu=Intel+Xeon+E3-1230+v3+%40+3.30GHz>`_
-  ``RAM`` 16GB
-  ``HDD`` SMC2108 SAS 275GB X 3EA

====== =========== ================== ==================== ==================== ================
size   conversion  response time(ms)  client traffic(Mbps) origin traffic(Mbps) saved traffic(%)
====== =========== ================== ==================== ==================== ================
16KB   720         19.32               46.32                   92.62              49.99
32KB   680         20.68               86.42                   165.08             47.65
64KB   285         50.16               80.67                   150.96             46.56
128KB  274         57.80               164.35                  276.52             40.56
256KB  210         80.74               99.42                   432.35             77.00
512KB  113         156.18              160.54                  436.04             63.18
1MB    20          981.07              90.62                   179.88             49.62
====== =========== ================== ==================== ==================== ================

DIMS may save about 50% traffic on average.
Optimization is heavy process for hardware resource and image size is most important as seen in the table.

Therefore considerate configuration is highly recommmended.
Optimization may be beneficial for service with some :ref:`adv_topics_req_hit_ratio`.
CPU resource must be allocated accordingly for service quality.


Crop
-----------------------

To crop the desired section of an image from the top left corner, the section is specified by the **widthxheight{+-}x{+-}y{%}** format. 
The following example crops a 100(width) by 200(height) section of an image from x=20, y=30. ::

   http://image.example.com/img.jpg/dims/crop/100x300+20+30/
    

Generating Thumbnails
-----------------------

This section explains how to generate thumbnails. 
The size and option can be expressed with **widthxheight{%} {@} {!} {<} {>}**.
Normally, a maximum value is applied for the width and height of the image. 
Even if the image is stretched or shrunk, the width and height ratio is maintained. 
If the image has be a specific size, an exclamation mark(!) is added at the end of the expression. 
A **640X480!** expression will generate a 640x480 size thumbnail image. 
If either the width or height is specified only, the omitted value is automatically determined by the width and height ratio. 

For example, the **/thumbnail/100/** expression will determine the height of the thumbnail based upon the width attribute, 
and the **/thumbnail/x200/** expression will determine the width of thumbnail based upon the height attribute. 
The thumbnail size can be expressed with a percentage(%) of the original image. 
In order to stretch the image, use a value that exceeds 100(eg. 125%). 
Likewise, in order to shrink the image, use a value less than 100.
You should be aware that the URL encoding rule encodes the % character as %25. 

For example, a 50% expression will be encoded as 50%25. 
Below is an example that generates a thumbnail with the dimensions of width=78 and height=110. ::

   http://image.example.com/img.jpg/dims/thumbnail/78x110/
    

Resizing
-----------------------

This function resizes the image. 
The size of an image is expressed in the **width x height** format. 
The ratio will be kept the same after the resizing. 
Below is an example that resizes an original image to width=200 and height=200. ::

   http://image.example.com/img.jpg/dims/resize/200x200/


Converting Format
-----------------------

This function converts the image format. 
Supported formats are "png", "jpg", "gif". 
The following is an example that converts "jpg" to "png". ::

   http://image.example.com/img.jpg/dims/format/png/


Adjusting Image Quality
-----------------------

This function adjusts the image quality. 
It is useful as it can reduce the volume of the transferring image. 
Effective values are from 0 to 100. 
The following example will adjust the quality of an image to 25%. ::

   http://image.example.com/img.jpg/dims/quality/25/
  

Image Composition
-----------------------

This function composites two or more images.
Unlike the previous functions, the image composite condition has to be pre-determined. 
This function is helpful when applying a watermark on the image.. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" port="8500">
      <Composite Name="water1" File="/img/small.jpg" />
      <Composite Name="water2" File="/img/medium.jpg" Gravity="se" Geometry="+0+0" Dissolve="50" />
      <Composite Name="water_ratio" File="/img/wmark_s.png" Gravity="s" Geometry="+0+15%" Dissolve="100" />
   </Dims>   
    
-  ``<Composite>``

    Configures the image composite condition, which is determined by attributes.
    
    -  ``Name`` Designates what the image will be called. 
       The "/" character cannot be used. 
       This option is located behind the "/composite/" of the URL.
       
    -  ``File`` Designates a path for the image to be synthesized. 
    
    -  ``Gravity (default: c)`` There are nine composite points (nw, n, ne, w, c, e, sw, s, and se).
       
       .. figure:: img/conf_dims2.png
          :align: center
       
          Gravity Reference Points
          
    -  ``Geometry (default: +0+0)`` Based on the ``Gravity``, ``Geometry`` decides where the composite image should be located. 
       {+-}x{+-}y. The red dots are reference points that stand for +0+0 of the ``Gravity`` attribute.
       The green arrow stands for the increasing x-axis, and the purple one stands for the increasing y-axis. 
       Using -x-y will locate the composite image outside of the boundary, but it will be invisible in the result image. 
       This method looks quite complicated, but it automatically calculates the size of the image so that results are consistent. 
       In addition, you can use a percentage option(%) like +x%+y% to locate the image.

    -  ``Dissolve (default: 50)`` The level of transparency of the image to be synthesized(0~100).

If you configure the ``<Composite>``, the ``Name`` property can be used to synthesize the image. ::

    http://image.example.com/img.jpg/dims/composite/water1/

    

Conditional Process of Original Image
-----------------------

You can dynamically apply different processing options based on the condition of the original image. 
For example, if you want to decrease the image quality by 50% for images that are smaller than 1024 X 768, 
and resize the image to 1024 X 768 for images that are bigger than 1024 X 768, ``<ByOriginal>`` can be used, as shown below. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Dims Status="Active" Keyword="dims" port="8500">
      <ByOriginal Name="size1">
         <Condition Width="1024" Height="768">/quality/25/</Condition>
         <Condition>/resize/1024x768/</Condition>
      </ByOriginal>
   </Dims>   

-  ``<ByOriginal>``
   Can be called by the ``Name`` attribute. 
   Various ``<Condition>`` configurations can be made.
   
-  ``<Condition>``
   When a condition is met, apply the configured value. 

   -  ``Width`` The condition is met if the width is smaller than this value.   
   -  ``Heigth`` The condition is met if the height is smaller than this value.
   If a condition is not specified, the conversion will occur regardless of the original image size.

``<Condition>`` is applied in order. 
Therefore, specific conditions have to come first.
The function can be called as follows. ::

   http://image.example.com/img.jpg/dims/byoriginal/size1/
    
Different ``<Composite>`` conditions can be applied based on the size of the image.
The following example shows how to set various pre-defined ``Name`` creations in a ``<Composite>``. ::

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
    
The following expression will generate a composite image based on the size of original image. ::

   http://image.example.com/img.jpg/dims/byoriginal/size_water/


.. _media-dims-anigif:

Animated GIF
-----------------------

Animated GIFs are also converted through DIMS functionality as the following order:

1. The animated GIF's frames are loaded as multple images.
2. Each image is converted.
3. All the frames are put back together in a single animated GIF.

More frames mean more time and resource to consume, which might slow down the service.
The FirstFrameOnly option accelerate the conversion, while converting only the first frames. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Dims FirstFrameOnly="OFF" />
   
-  ``FirstFrameOnly (default: OFF)`` Converts only the first frame of animated GIFs if ON

`` FirstFrameOnly `` option is available on URLs as follow. ::

   http://image.example.com/img.jpg/dims/firstframeonly/on/resize/200x200/
   http://image.example.com/img.jpg/dims/firstframeonly/off/resize/200x200/

Commands from URLs as above have priority over configuration.


ETC
-----------------------

Combining the above functions will allow you to generate a acomplex image. 
For example, a single call can generate a thumbnail(78x110), convert JPG format to PNG, and create 50% image quality options. ::

   http://image.example.com/img.jpg/dims/thumbnail/78x110/format/png/quality/50/
    
DIMS uses a URL for image processing. 
Therefore, in order to avoid any unexpected results, you should be leery of other options that might affect the URL.

-  If :ref:`caching-policy-applyquerystring` is set to ``OFF``, the QueryString prior to the keyword will be ignored. ::
   
      http://image.example.com/img.jpg?session=5234&type=37/dims/resize/200x200/
      
   The above example will be identified as it is if the attribute is ``ON``.
   However, if the attribute is ``OFF``, the above URL will be identified as shown below. ::
      
      http://image.example.com/img.jpg/dims/resize/200x200/
      
-  If :ref:`caching-policy-casesensitive` is set to ``OFF``, all characters in the URL will be converted to lowercase characters.
   Therefore, using lowercase characters for the keyword is recommended.
   When uppercase characters are included in the DIMS keyword, the keyword will not be recognized. 
