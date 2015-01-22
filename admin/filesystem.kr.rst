.. _filesystem:

Chapter 13. File System
******************

This chapter explains how to utilize STON like a local disk.
STON is based on `FUSE <http://fuse.sourceforge.net/>`_ and mounted on the Linux VFS(Virtual File System).
All files in the mounted path will be cached at the moment of access, but other processes do not notice this.
You can view this system as **a ReadOnly disk with Caching function**.

.. figure:: img/conf_fs1.png
   :align: center
      
   `Fuse <http://upload.wikimedia.org/wikipedia/commons/0/08/FUSE_structure.svg>`_ architecture

When the Linux Kernel directly forwards the file I/O function call to STON, 
no other elements(physical file I/O, socket communication, etc) interfere the process.
This architecture enables extremely high performance.
The memory caching of STON will enhance access performance more than the physical disk.


.. toctree::
   :maxdepth: 2

Mount
====================================

Mount is configured in the global setting(server.xml). ::

   # server.xml - <Server><Cache>

   <FileSystem Mount="/cachefs" DotDir="OFF" Separator="^">OFF</FileSystem>   
    
-  ``<FileSystem>``

   -  ``OFF (default)`` doesn't do anything.
   
   -  ``ON`` mounts STON to the path of ``Mount`` property.
   
STON keeps the previous HTTP structure, but developed an expanded file system that accesses to the cache module. 
Therefore, wherever the access is coming from, caching only happens once and serviced with either HTTP or file I/O. 
The File System is another method to access cache module. 
   
.. figure:: img/conf_fs2.png
   :align: center
      
   HTTP and File I/O share cache module.

Not only HTTP, but also File I/O can access contents in the origin server.
By utilizing this method, you can increase the availability of solutions that are based on local files.

.. figure:: img/conf_fs3.png
   :align: center
      
   Any servers are OK
   
The followings are supported functions by STON File System.

========= =============== ===========
FUSE	  C	              LINUX
========= =============== ===========
open	  fopen	          open
release	  fclose	      close
read	  fseek, fread	  seek, read
getattr	  fstat	          stat
unlink	  remove	      unlink
========= =============== ===========

File I/O goes through several internal steps. 
In order to achieve best performance, you should thoroughly understand each step.



Searching for a Virtual Host
====================================

The first step is searching for a virtual host to access. 
The Host header in the HTTP request helps to find virtual host. ::

    GET /ston.jpg HTTP/1.1
    host: example.com
    
File System에서는 첫 번째 경로로 이 문제(무엇이 문제인지 명시되지 않았음??)를 해결한다. 
For example, if the STON is mounted to the /cachefs path, the below path can be used to access local files. ::

    /cachefs/example.com/ston.jpg
        
:ref:`env-vhost-find` works in the same way.
If *.example.com is configured as an ``<Aliss>`` of example.com, all following accesses refer to the identical file. ::

    /cachefs/example.com/ston.jpg
    /cachefs/img.example.com/ston.jpg
    /cachefs/example.example.com/ston.jpg
    
For instance, in order to link example.com to Apache server, you have to set the DocumentRoot as /cachefs/example.com/.


File/Directory
====================================

This section explains how to configure file systems for each virtual host. 
All virtual hosts can also be uniformly configured with a default virtual host. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <FileSystem Status="Active" DotDir="OFF">                    
      <FileStatus>200</FileStatus>
      <DirStatus>301, 302, 400, 401, 403</DirStatus>
      <Unlink>Purge</Unlink>
   </FileSystem>   
    
-  ``<FileSystem>``
   If ``Status`` attribute is set to ``Inactive``, File System cannot access to the file/directory. 
   Set this to `Active`.

-  ``<FileStatus>``
   Configures HTTP response code of the origin server that will be identified as a file. 
   Usually 2000000 is used, but there is no restriction on this value.
   
-  ``<DirStatus>``
    Configures HTTP response code of the origin server that will be identified as a directory. 
    Default values are 302, 400, 401, 403.
    
-  ``<Unlink>``
   Select how to deal with a file removal request from ``Purge``, ``Expire``, ``HardPurge``.

Each origin server can interpret HTTP response codes in multiple ways. 
Therefore, you have to configure how to interpret each HTTP response code. 

If the file is existing in the origin server, it will be replied with **200 OK** in most cases. 
In case of directory access, **403 Forbidden** can be replied or redirected to other page with **302 Found**. 
If the comma(,) is used to identify each response code name, the Body of corresponding HTTP response code is identifed as a file or a directory. 
Any response codes that are not configured will be considered as not existing code, and the File I/O for the code will fail.


File Attribute
====================================

The first step of file I/O is getting the file attribute. 
It is an obvious procedure to acquire the file attribute before opening it. 
From the STON side of view, Kernel services the file attribute as below. 
(/cachefs is a mount path so Kernel drops it.)

.. figure:: img/conf_fs4.png
   :align: center
      
   Process of acquiring the file attribute

Linux does not distinguish files from directories, so acquiring a specific file attribute is more complex than it seems to be. 
As you can see from the above figure, as the number of subfolder increases, the performance gets decreased
because unnecessary virtual host search and file access occur. 
Especially, inaccessible directory requests like /one or /one/two occurs and causes origin server load. 
Once the file is cached, the origin server access will not happen while the TTL(Time To Live) is alive.
However, still it is not an ideal solution for the system.

A heuristic solution for this structural load ia adding a ``DotDir`` attribute.
``DotDir`` is a function that recognizes a path without a dot(.) as a directory(Dir). 
The previous figure illustrates when the ``DotDir`` is ``OFF``. 
The following figure illustrates when the ``DotDir`` is ``ON``.

.. figure:: img/conf_fs5.png
   :align: center
      
   Enabling( ``ON`` ) the global ``DotDir``

The calling process or frequency from the Kernel does not change. 
However, if a dot(.) is missing in the requested path, the request is considered as a directory.
Therefore, the virtual host does not respond, but the virtual host and file will be refered only for necessary requests. 
This feature is based on the observation that most programmers do not assign extensions to directories. 
In order to properly use this feature, you have to confirm the directory structure.

``DotDir`` is a global attribute of the ``<FileSystem>``.
In other words if none of virtual hosts use dots(.) for all directories, setting the ``DotDir`` to ``ON`` will make a very efficient system. 
Of course you can set the ``DotDir`` to ``OFF`` and configure each virtual host separately. 
This case, there will be a minor performance degradation as the below figure.

.. figure:: img/conf_fs6.png
   :align: center
      
   Activating( ``ON`` ) ``DotDir`` feature of virtual host

Virtual host search will occur, but refering file will only occur with a dot(.) in the request.
Since this function is frequently called, it is recommended for you to understand for better performance.


File Read
====================================

Acquiring a file attribute is complicated while reading a file is simple. 
First of all, open the file. 
All files are ReadOnly, therefore a file access with the Write permission will fail.
If a file is accessed for the first time, the file will be cached from the origin server just like the HTTP service. 
While downloading the requested file, file I/O is serviced concurrently so that the process would not wait.

.. figure:: img/conf_fs7.png
   :align: center
      
   File Open

After opening a file, procedures are identical to the HTTP service.
HTTP is advantageous in file transfer because sequential file access occurs from initially determined range.
On the other hand, file I/O might generate perpetual read accesses of about 1KB size regardless of the file size. 
STON implemented `Readahead <http://en.wikipedia.org/wiki/Readahead>`_ in the cache module
and this feature maximized file I/O performance. 

If a function to close a file(eg. fclose) is called or a process is terminated, file handle is turned in by the Kernel. 
This is same as closing an HTTP transaction.


File Delete
====================================
Cached file is managed by STON, but the process can request to delete the file.
STON provides several :ref:`api-cmd-purge` methods to respond with these requests 

For example, if ``<Unlink>`` is set to ``expire``, the corresponding file will be expired upon the file deletion request.
If the Kernel tries to access the file again, the file needs to be checked for modification because the file is expired.
Then, if the file is not modified, it can be serviced again.


File Expansion
====================================
HTTP can dynamically process original file with the following URL. :: 
    
    # Trim 0-60 second section of the /video.mp4 file via HTTP.
    http://www.example.com/video.mp4?start=0&end=60
    
QueryString method like this can be used for both HTTP and File System call methods. ::

    # Access the local file that has been trimmed from 0 to 60 second of /video.mp4.
    /cachefs/www.example.com/video.mp4?start=0&end=60
    
However, the process option followed by URL like MP4HLS or DIMS has problem in File I/O. ::

    /cachefs/image.winesoft.com/img.jpg/12AB/resize/500x500/
    /cachefs/www.winesoft.com/video.mp4/mp4hls/index.m3u8
    
As it is explained in "File Attribute", Linux inquries attribute for each path. 
STON does not know whether the additinoal path is existing at the end of requested path or not, so unprocessed files are serviced.

In order to resolve this issue, STON uses ``Separator (default: ^)`` attribute of the ``<FileSystem>`` as an identifier. ::

    /cachefs/image.winesoft.com/img.jpg^12AB^resize^500x500^
    /cachefs/www.winesoft.com/video.mp4^mp4hls^index.m3u8
    
.. figure:: img/conf_fs9.png
   :align: center
      
   MP4HLS access

Inside of the STON, ``Separator`` is switched to slash(/) so HTTP call standard can be 
STON 내부에서는 ``Separator`` 를 slash(/)로 변경하여 HTTP와 동일한 호출규격을 사용한다. 
이를 적극적으로 활용할 경우 다음과 같이 불필요 File I/O접근을 완전히 제거할 수 있다.

.. figure:: img/conf_fs7.png
   :align: center
      
   Extremely optimized access



Wowza 연동
====================================

File System을 이용해 손쉽게 Wowza를 연동할 수 있다. 
STON이 Mount된 경로를 Wowza의 파일경로로 설정하는 것으로 모든 설정이 완료된다.

**1. [STON - 전역설정] 파일시스템 설정 ON**

  전역설정(server.xml)에 다음과 같이 ``<FileSystem>`` 을 ``ON`` 으로 설정한다. 
  (예제에서는 Mount경로를 "/cachefs"로 설정한다.) ::
  
     # server.xml - <Server><Cache>
         
     <FileSystem Mount="/cachefs" DotDir="OFF" Separator="^">ON</FileSystem>     
     
  또는 WM의 전역설정 - 파일시스템에서 다음과 같이 파일 시스템을 "사용한다"로 설정한다.
  
  .. figure:: img/faq_wowza1.png
     :align: center

     설정 후 반드시 STON을 재시작해야 Mount된다.
     
**2. [STON - 가상호스트] 파일시스템 접근허가 & 응답코드 설정**

  가상호스트의 파일시스템 접근을 Active시킨다. 
  원본서버 응답코드에 따른 파일/디렉토리 결정도 설정한다. 
  여기서는 가상호스트 기본 설정(server.xml)을 예로 설명하지만 
  각각의 가상호스트(vhosts.xml)에서 개별적으로 설정할 수 있다. ::
  
     # server.xml - <Server><VHostDefault><Options>
     # vhosts.xml - <Vhosts><Vhost><Options>
     
     <FileSystem Status="Active" DotDir="OFF">
        <FileStatus>200</FileStatus>
        <DirStatus>301, 302, 400, 401, 403</DirStatus>
     </FileSystem>
     
  또는 WM의 가상호스트 - 파일시스템에서 다음과 같이 접근을 "허가한다"로 설정한다.
  
  .. figure:: img/faq_wowza2.png
     :align: center

     응답코드를 설정한다.
     
     
**3. [Wowza] Storage 경로 설정**

  Wowza설치경로 /Conf/Application.xml 파일을 다음과 같이 STON이 Mount된 경로를 바라보도록 편집한다. ::

     <Streams>
       <StreamType>default</StreamType>
       <StorageDir>/cachefs/example.com</StorageDir>
       <KeyDir>${com.wowza.wms.context.VHostConfigHome}/keys</KeyDir>
     </Streams>
     
**4. [Wowza] VOD 경로설정**

  Wowza설치경로 /Conf/vod/Application.xml 파일을 다음과 같이 STON이 Mount된 경로를 바라보도록 편집한다. ::
  
     <Streams>
       <StreamType>default</StreamType>
       <StorageDir>/cachefs/example.com</StorageDir>
       <KeyDir>${com.wowza.wms.context.VHostConfigHome}/keys</KeyDir>
     </Streams>
     
**5. 플레이어 테스트**

  Wowza 테스트 플레이어로 로컬에 존재하지 않는(=STON이 캐싱해야 하는) 영상을 RTMP로 재생한다.
  
  .. figure:: img/faq_wowza3.png
     :align: center

     테스트엔 적절한 영상이 필요합니다.
