.. _filesystem:

Chapter 13. File System
******************

This chapter explains how to utilize STON like a local disk.
STON is based on `FUSE <http://fuse.sourceforge.net/>`_ and mounted on the Linux VFS(Virtual File System).
All files in the mounted path will be cached at the moment of access, but other processes do not notice this.
You can view this system as **a readonly disk with caching function**.

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


File Attributes
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

Kernel에서 호출되는 과정이나 회수에는 변함이 없다. 
하지만 요청된 경로에 dot(.)이 없으면 가상호스트까지 가지 않고 즉시 디렉토리로 응답하기 때문에 꼭 필요한 부분에서만 가상호스트와 파일이 참조된다. 
이 기능은 대부분의 프로그래머들이 파일에만 확장자를 부여하고 디렉토리에는 그렇지 않다는 것에 착안한 기능이다. 
그러므로 사용하기 전에 디렉토리 구조에 대해 반드시 확인이 필요하다.

``<FileSystem>`` 은 ``DotDir`` 속성은 전역이다.
쉽게 말해 모든 가상호스트가 디렉토리에 dot(.)을 사용하지 않는다면 전역 ``DotDir`` 을 ``ON`` 으로 설정하시는 것이 아주 효과적이다. 
물론 전역 ``DotDir`` 을 ``OFF`` 로 설정하고 가상호스트마다 별도로 설정할 수도 있다. 
이 경우 다음 그림처럼 약간의 성능부하가 발생한다.

.. figure:: img/conf_fs6.png
   :align: center
      
   가상호스트 ``DotDir`` 활성화( ``ON`` )

가상호스트 검색은 발생하지만 파일참조는 dot(.)이 있는 상태에서만 발생한다.
매우 빈번하게 호출되는 만큼 성능과 관련하여 반드시 이해할 것을 권장한다.


파일읽기
====================================

파일속성을 얻는 과정은 복잡하지만 정작 파일 읽기는 간단하다. 
먼저 파일을 Open한다. 
모든 파일은 당연히 ReadOnly이다.
Write권한의 파일 접근은 실패한다.
최초 파일이 접근되는 경우 HTTP와 마찬가지로 원본서버에서 파일을 Caching한다. 
파일을 요청한 프로세스가 기다리지 않도록 다운로드를 진행하면서 동시에 File I/O 서비스가 이루어진다.

.. figure:: img/conf_fs7.png
   :align: center
      
   파일 Open

이후 동작은 HTTP 서비스와 동일하다.
다만 HTTP의 경우 처음 결정된 Range에서 순차적(Sequential)인 파일접근이 발생하기 때문에 파일 전송에 유리한 면이 있다.
반면 File I/O의 경우 파일 크기와 상관없이 아주 작은 1KB단위의 read접근이 매우 많이 발생할 수 있다. 
성능의 극대화를 위해 STON은 Cache모듈에 `Readahead <http://en.wikipedia.org/wiki/Readahead>`_ 를 구현했으며, 이를 통해 File I/O 성능을 극대화시켰다. 

파일닫기(fclose등) 함수가 호출되거나 프로세스가 종료되는 경우 파일 handle은 Kernel에 의해 반납된다. 
이는 HTTP 트랜잭션이 종료되는 것과 같다.


파일삭제
====================================
Caching된 파일은 STON에 의해 관리되지만 프로세스가 삭제요청을 보낼 수 있다.
STON은 다양한 :ref:`api-cmd-purge` 방법을 제공하고 있으므로 이런 요청에 쉽게 대응할 수 있다. 

예를 들어 ``<Unlink>`` 가 ``expire`` 로 설정되어 있는 경우 파일삭제 요청에 대해 해당 파일을 expire하도록 동작한다.
Kernel에서 다시 해당 파일에 접근한다면 expire된 상태이므로 원본서버에서 변경여부를 확인한 뒤 변경되지 않았다면 해당 파일을 다시 서비스한다.


파일확장
====================================
HTTP의 경우 다음과 같이 URL을 이용하여 원본 파일을 동적으로 가공할 수 있다. :: 
    
    # HTTP를 통해 /video.mp4의 0~60초 구간을 Trimming한다.
    http://www.example.com/video.mp4?start=0&end=60
    
이와 같은 QueryString방식은 HTTP와 File System 모두 호출규격을 동일하게 사용할 수 있다. ::

    # "/video.mp4의 0~60초 구간을 Trimming한" 로컬파일에 접근한다.
    /cachefs/www.example.com/video.mp4?start=0&end=60
    
하지만 MP4HLS나 DIMS처럼 원본 URL뒤에 가공옵션을 디렉토리 형식으로 명시하는 방식은 File I/O에 문제가 있다. ::

    /cachefs/image.winesoft.com/img.jpg/12AB/resize/500x500/
    /cachefs/www.winesoft.com/video.mp4/mp4hls/index.m3u8
    
"파일속성 얻기" 에서 설명한 바와 같이 LINUX는 경로 각 부분의 속성을 매번 물어본다. 
STON관점에서는 현재 물어보는 경로 뒤에 추가 경로가 있는지 알 수 없기 때문에 가공되지 않은 파일을 서비스하게 된다.

이 문제를 극복하기 위해서 STON은 별도의 구분자로 ``<FileSystem>`` 의 ``Separator (기본: ^)`` 속성을 사용한다. ::

    /cachefs/image.winesoft.com/img.jpg^12AB^resize^500x500^
    /cachefs/www.winesoft.com/video.mp4^mp4hls^index.m3u8
    
.. figure:: img/conf_fs9.png
   :align: center
      
   MP4HLS 접근

STON 내부에서는 ``Separator`` 를 slash(/)로 변경하여 HTTP와 동일한 호출규격을 사용한다. 
이를 적극적으로 활용할 경우 다음과 같이 불필요 File I/O접근을 완전히 제거할 수 있다.

.. figure:: img/conf_fs7.png
   :align: center
      
   극도로 최적화된 접근



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
