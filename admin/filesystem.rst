.. _filesystem:

Chapter 16. File System
******************

This chapter explains how to utilize STON like a local disk.
STON is based on `FUSE <http://fuse.sourceforge.net/>`_ and mounted on the Linux VFS(Virtual File System).
All files in the mounted path will be cached at the moment of access, but other processes will not notice.
You can view this system as **a ReadOnly disk with a Caching function**.

.. figure:: img/conf_fs1.png
   :align: center
      
   `Fuse <http://upload.wikimedia.org/wikipedia/commons/0/08/FUSE_structure.svg>`_ architecture

When the Linux Kernel forwards the file I/O function call directly to STON, 
no other elements(physical file I/O, socket communication, etc) will interfere with the process.
This architecture enables extremely high performance.
STON's memory caching capacity will enhance access performance more than the physical disk.


.. toctree::
   :maxdepth: 2

Mount
====================================

Mount is configured in the global setting(server.xml). ::

   # server.xml - <Server><Cache>

   <FileSystem Mount="/cachefs" DotDir="OFF" Separator="^">OFF</FileSystem>   
    
-  ``<FileSystem>``

   -  ``OFF (default)`` Doesn't do anything.
   
   -  ``ON`` Mounts STON to the path of the ``Mount`` property.
   
STON keeps the previous HTTP structure, but develops an expanded file system that accesses the cache module. 
Therefore, wherever the access is coming from, caching only happens once and is serviced with either HTTP or a file I/O. 
The File System is another way to access the cache module. 
   
.. figure:: img/conf_fs2.png
   :align: center
      
   HTTP and File I/O share a cache module.

Not only HTTP, but also File I/O can access content in the origin server.
By utilizing this method, you can increase the availability of solutions that are based on local files.

.. figure:: img/conf_fs3.png
   :align: center
      
   Any server is OK.
   
The following are supported functions by the STON File System.

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
In order to achieve the best performance, you should thoroughly understand each step.



Searching for a Virtual Host
====================================

The first step is searching for the virtual host to access. 
The Host header in the HTTP request helps finding a the virtual host. ::

    GET /ston.jpg HTTP/1.1
    host: example.com
    
File System uses the first path as the virtual host name, thus the host name can be used in the file system as well. 
For example, if STON is mounted to the /cachefs path, the below path can be used to access local files. ::

    /cachefs/example.com/ston.jpg
        
:ref:`env-vhost-find` works in the same way.
If *.example.com is configured as an ``<Aliss>`` of example.com, all of the following accesses refer to the identical file. ::

    /cachefs/example.com/ston.jpg
    /cachefs/img.example.com/ston.jpg
    /cachefs/example.example.com/ston.jpg
    
For instance, in order to link example.com to the Apache server, you have to set the DocumentRoot as /cachefs/example.com/.


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
   If the ``Status`` attribute is set to ``Inactive``, the File System cannot access the file/directory. 
   Set the ``status`` attribute to ``Active``.

-  ``<FileStatus>``
   Configures an HTTP response code for the origin server that will be identified as a file. 
   Usually 200 is used, but you can use other values for this.
   
-  ``<DirStatus>``
    Configures an HTTP response code for the origin server that will be identified as a directory. 
    Default values are 302, 400, 401, 403.
    
-  ``<Unlink>``
   Select how to deal with a file removal request from ``Purge``, ``Expire``, ``HardPurge``.

Each origin server can interpret HTTP response codes in multiple ways. 
Therefore, you must configure how to interpret each HTTP response code. 

If the file already exists in the origin server, it will reply with a **200 OK** response in most cases. 
In the case of directory access, the client will receive a **403 Forbidden** reply or will be redirected to another page with **302 Found**. 
If a comma(,) is used to identify each response code name, the Body of the corresponding HTTP response code is identified as a file or a directory. 
Any response code that is not configured will be considered as a non-existing code, and File I/O for the code will fail.


File Attribute
====================================

The first step of File I/O is getting the file attribute. 
Acquiring the file attribute before opening it is an obvious procedure. 
From the STON side of view, Kernel services the file attribute, as shown below. 
(/cachefs is a mount path so Kernel drops it.)

.. figure:: img/conf_fs4.png
   :align: center
      
   The process of acquiring the file attribute

Linux does not distinguish files from directories, so acquiring a specific file attribute is more complex than it seems to be. 
As you can see from the above figure, as the number of subfolders increases, performance decreases
because unnecessary virtual host searches and file accesses occur, 
especially inaccessible directory requests like /one or /one/two, which cause origin server load. 
Once the file is cached, the origin server will not be accessed while the TTL(Time To Live) is alive.
However, this is not an ideal solution for the system.

A heuristic solution for this structural load is adding a ``DotDir`` attribute.
``DotDir`` is a function that recognizes a path without a dot(.) as a directory(Dir). 
The previous figure illustrates when the ``DotDir`` is ``OFF``. 
The following figure illustrates when the ``DotDir`` is ``ON``.

.. figure:: img/conf_fs5.png
   :align: center
      
   Enabling( ``ON`` ) the global ``DotDir``.

The calling process or frequency from Kernel does not change. 
However, if a dot(.) is missing in the requested path, the request is considered as a directory.
Therefore, the virtual host will not respond, but the virtual host and file will be referred to only for necessary requests. 
This feature is based on the observation that most programmers do not assign extensions to directories. 
In order to properly use this feature, you have to confirm the directory structure.

``DotDir`` is a global attribute of the ``<FileSystem>``.
In other words, if none of the virtual hosts use dots(.) for directories, setting the ``DotDir`` to ``ON`` will make a very efficient system. 
Of course you can set the ``DotDir`` to ``OFF`` and configure each virtual host separately. 
In this case, there will be a minor performance degradation, as shown in the below figure.

.. figure:: img/conf_fs6.png
   :align: center
      
   Activating the ( ``ON`` ) ``DotDir`` feature of the virtual host

A virtual host search will occur, but the referring file will only occur with a dot(.) in the request.
For better performance, it is recommended for you to understand this function, as it is frequently called.


File Read
====================================

Acquiring a file attribute is complicated while reading a file is simple. 
First, open the file. 
All files are ReadOnly, so access to a Write permission file will fail.
If a file is accessed for the first time, the file will be cached from the origin server just like the HTTP service. 
While downloading the requested file, File I/O is serviced concurrently so that the process will not be delayed.

.. figure:: img/conf_fs7.png
   :align: center
      
   Opening a file.

Once a file is opened, procedures are identical to the HTTP service.
HTTP is advantageous in file transfer because sequential file access occurs from an initially determined range.
On the other hand, file I/O might generate perpetual read accesses of about 1KB regardless of the file size. 
STON has `Readahead <http://en.wikipedia.org/wiki/Readahead>`_ implemented in the cache module
and this feature maximizes file I/O performance. 

If a function to close a file(e.g. fclose) is called or a process is terminated, the file handle is turned in by Kernel, 
which is the same process as closing an HTTP transaction.


File Delete
====================================
A cached file is managed by STON, but the process can request to delete the file.
STON provides several :ref:`api-cmd-purge` methods to respond to these requests 

For example, if ``<Unlink>`` is set to ``expire``, the corresponding file will be expired upon a file deletion request.
If Kernel tries to access the file again, the file needs to be checked for modification because the file is expired.
Then, if the file has not been modified, it can be serviced again.


File Expansion
====================================
HTTP can dynamically process an original file with the following URL. :: 
    
    # Trim 0-60 second section of the /video.mp4 file via HTTP.
    http://www.example.com/video.mp4?start=0&end=60
    
The QueryString method like this can be used for both HTTP and File System call methods. ::

    # Access the local file that has been trimmed from 0 to 60 second of /video.mp4.
    /cachefs/www.example.com/video.mp4?start=0&end=60
    
However, the process option followed by URLs like MP4HLS or DIMS has problems in File I/O. ::

    /cachefs/image.winesoft.com/img.jpg/12AB/resize/500x500/
    /cachefs/www.winesoft.com/video.mp4/mp4hls/index.m3u8
    
As it is explained in "File Attribute", Linux inquries property of each path. 
STON does not know whether the additional path exists at the end of the requested path or not, so unprocessed files are serviced.

In order to resolve this issue, STON uses the ``Separator (default: ^)`` attribute of the ``<FileSystem>`` as an identifier. ::

    /cachefs/image.winesoft.com/img.jpg^12AB^resize^500x500^
    /cachefs/www.winesoft.com/video.mp4^mp4hls^index.m3u8
    
.. figure:: img/conf_fs9.png
   :align: center
      
   MP4HLS access

Inside STON, ``Separator``s are switched to slashes(/) so that the HTTP call standard can be used identically. 
Using this separator eliminates unnecessary File I/O access, as shown below.

.. figure:: img/conf_fs7.png
   :align: center
      
   Extremely optimized access.



Wowza Interworking
====================================

File System can easily interwork with Wowza. 
All you have to do is configure a STON mounted path as a file path for Wowza.

**1. [STON - Global setting] Turn on the file system configuration**

  Set the ``<FileSystem>`` to ``ON`` in the global setting(server.xml). 
  (In the example below, the mount path is configured as "/cachefs".) ::
  
     # server.xml - <Server><Cache>
         
     <FileSystem Mount="/cachefs" DotDir="OFF" Separator="^">ON</FileSystem>     
     
  Or, in the WM, go to the global setting - File System and configure the file system to ``enable``.
  
  .. figure:: img/faq_wowza1.png
     :align: center

     For a successful mount, STON must be restarted after the configuration.
     
**2. [STON - Virtual host] Create file system access permission & the response code configuration**

  This section explains how to activate the file system access of the virtual host. 
  You can also configure whether to identify the URL as a file or a directory based on the response code from the origin server.
  The following uses the virtual host default setting(server.xml) as an example, but each virtual hosts(vhosts.xml) can have independent configurations. ::
  
     # server.xml - <Server><VHostDefault><Options>
     # vhosts.xml - <Vhosts><Vhost><Options>
     
     <FileSystem Status="Active" DotDir="OFF">
        <FileStatus>200</FileStatus>
        <DirStatus>301, 302, 400, 401, 403</DirStatus>
     </FileSystem>
     
  Or, in the WM, go to the global setting - File system and configure the following access to ``allow``.
  
  .. figure:: img/faq_wowza2.png
     :align: center

     Configure the response code.
     
     
**3. [Wowza] Storage Path Configuration**

  In the Wowza installed path, the /Conf/Application.xml file should be edited to refer the STON mounted path, as shown below. ::

     <Streams>
       <StreamType>default</StreamType>
       <StorageDir>/cachefs/example.com</StorageDir>
       <KeyDir>${com.wowza.wms.context.VHostConfigHome}/keys</KeyDir>
     </Streams>
     
**4. [Wowza] VOD Path Configuration**

  In the Wowza installed path, the /Conf/vod/Application.xml file should be edited to refer the STON mounted path, as shown below. ::
  
     <Streams>
       <StreamType>default</StreamType>
       <StorageDir>/cachefs/example.com</StorageDir>
       <KeyDir>${com.wowza.wms.context.VHostConfigHome}/keys</KeyDir>
     </Streams>
     
**5. Player Test**

  Use the Wowza test player to play the video that is not saved in the local storage(the video that STON has to cache) via RTMP.
  
  .. figure:: img/faq_wowza3.png
     :align: center

     The test needs an appropriate video clip.
