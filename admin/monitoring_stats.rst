.. _monitoring_stats:

Chapter 10. Monitoring & Statistics
******************

This chapter explains monitoring and statistics.
While somewhat different in their usage, they are similar in the way that they use numbers to notify.

Real time updates are very important in monitoring service.
Waiting for five minutes is too long.
You have to see the change of service status in real time.
You must observe whether the policies are taking effect when they are applied.
All statistics are collected every second.

Statistics are collected on each virtual host and provided every second and an average is taken very five minutes. 
In order to help customers analyze and process stats more easily, JSON and XML format is used. ::

    http://127.0.0.1:10040/monitoring/realtime?type=[JSON or XML]
    http://127.0.0.1:10040/monitoring/average?type=[JSON or XML]
    
-  ``realtime`` 
   Provides the service status of one second prior.

-  ``average`` 
   Provides the average of five minutes of statistics.


.. toctree::
   :maxdepth: 2



Data Range
====================================

Configures the range of data to be collected. ::

   # server.xml - <Server><VHostDefault>
   # vhosts.xml - <Vhosts><Vhost>

   <Stats>
      <DirDepth>0</DirDepth>
      <DirDepthAccum>OFF</DirDepthAccum>
      <HttpsTraffic>OFF</HttpsTraffic>
      <ClientLocal>OFF</ClientLocal>
      <OriginLocal>OFF</OriginLocal>
   </Stats>
    
-  ``<DirDepth> (default: 0)`` 
   
   Collects statistics of each directory. 
   If this is set to 0, all statistics are collected in the root(/) directory. 
   If this is set to 1, statistics are collected in every first depth directory. 
   
   .. note:
      
      There is no restriction in this value, but memory problem could occur if you are trying to collect stats from tens of thousands of directories.
   
-  ``<DirDepthAccum>`` 

   Configures whether to accumulate statistics in the upper directory when collecting stats of each directory.
   This will be ignored if ``<DirDepth>`` is set to 0.
   
   - ``OFF (default)`` Does not accumulate stats in the upper directory.
   
   - ``ON`` Accumulates stats in the upper directory.
   
   Let's assume that ``<DirDepth>`` is set to 2 and all directories have 10 lines of traffic. 
   If ``<DirDepthAccum>`` is set to ``OFF``, the stats are collected in each directory where traffic is occurring, as shown in the figure on the left below.
   On the other hand, if it is set to ``ON``, all stats from subdirectories are accumulated in the upper directories, as shown in the figure on the right.
   
   .. figure:: img/stats_dirdepth.jpg
      :align: center
          
      Accumulated Statistics in the Upper Directory
   
   For instance, the summation of accumulated traffic from the subdirectories and traffic from the /img directory will be 30.
   This statistical value is accumulated in the upper directory.

-  ``<HttpsTraffic>`` 

   - ``OFF (default)`` Collects HTTPS traffic only with SSL stats.
   
   - ``ON`` Collects HTTPS traffic with both SSL and HTTP stats. 
   
   When traffic passes through the SSL layer, it will be collected in a separate SSL stats.
   HTTPS is processed as HTTP in the upper protocol; therefore, more detailed stats can be collected. 
   However, if there are redundancies in the SSL and HTTP stats, it is recommended to trust the HTTP stats.

-  ``<ClientLocal>`` 

   Takes statistical counts of the traffic between Loopback client and STON.

   - ``OFF (default)`` Does not count the traffic.
   
   - ``ON`` Counts the traffic.

-  ``<OriginLocal>`` 

   Takes statistical counts the traffic between STON and Loopback origin server.

   - ``OFF (default)`` Does not count the traffic.
   
   - ``ON`` Counts the traffic.
   

Host Comprehensive Statistics
====================================

Host stats are the aggregate stats of all virtual hosts in the service. 
Identical stats are provided in JSON and XML formats. ::

   {                                            <Host                                    
     "Host":                                      Version="2.0.0"                       
     {                                            Name="localhost"                       
       "Version":"2.0.0",                         State="Healthy"                        
       "Name":"localhost",                        Uptime="155986"                        
       "State":"Healthy",                         OriginSession="32"                     
       "Uptime":155996,                           OriginActiveSession="20"               
       "OriginSession":33,                        OriginInbound="1140741"                
       "OriginActiveSession":20,                  OriginOutbound="10059"                 
       "OriginInbound":688177,                    OriginReqCount="42"                    
       "OriginOutbound":14184,                    OriginResTotalCount="42"               
       "OriginReqCount":62,                       OriginResTotalTimeRes="5071"           
       "OriginResTotalCount":62,                  OriginResTotalTimeComplete="10288"     
       "OriginResTotalTimeRes":2375,              OriginRes2xxCount="19"                 
       "OriginResTotalTimeComplete":2509,         OriginRes2xxTimeRes="9989"             
       "OriginRes2xxCount":54,                    OriginRes2xxTimeComplete="21521"       
       "OriginRes2xxTimeRes":2327,                OriginRes3xxCount="23"                 
       "OriginRes2xxTimeComplete":2481,           OriginRes3xxTimeRes="1008"             
       "OriginRes3xxCount":8,                     OriginRes3xxTimeComplete="1008"        
       "OriginRes3xxTimeRes":2700,                OriginRes4xxCount="0"                  
       "OriginRes3xxTimeComplete":2700,           OriginRes4xxTimeRes="0"                
       "OriginRes4xxCount":0,                     OriginRes4xxTimeComplete="0"           
       "OriginRes4xxTimeRes":0,                   OriginRes5xxCount="0"                  
       "OriginRes4xxTimeComplete":0,              OriginRes5xxTimeRes="0"                
       "OriginRes5xxCount":0,                     OriginRes5xxTimeComplete="0"           
       "OriginRes5xxTimeRes":0,                   ClientSession="165"                    
       "OriginRes5xxTimeComplete":0,              ClientActiveSession="80"               
       "ClientSession":155,                       ClientInbound="14792"                  
       "ClientActiveSession":80                   ClientOutbound="1981700"               
       "ClientInbound":35748,                     ClientReqCount="64"                    
       "ClientOutbound":972906,                   ClientResTotalCount="64"               
       "ClientReqCount":152,                      ClientResTotalTimeRes="5535"           
       "ClientResTotalCount":152,                 ClientResTotalTimeComplete="6840"      
       "ClientResTotalTimeRes":1411,              ClientRes2xxCount="44"                 
       "ClientResTotalTimeComplete":1479,         ClientRes2xxTimeRes="8050"             
       "ClientRes2xxCount":93,                    ClientRes2xxTimeComplete="9943"        
       "ClientRes2xxTimeRes":2305,                ClientRes3xxCount="20"                 
       "ClientRes2xxTimeComplete":2409,           ClientRes3xxTimeRes="5"                
       "ClientRes3xxCount":59,                    ClientRes3xxTimeComplete="15"          
       "ClientRes3xxTimeRes":3,                   ClientRes4xxCount="0"                  
       "ClientRes3xxTimeComplete":13,             ClientRes4xxTimeRes="0"                
       "ClientRes4xxCount":0,                     ClientRes4xxTimeComplete="0"           
       "ClientRes4xxTimeRes":0,                   ClientRes5xxCount="0"                  
       "ClientRes4xxTimeComplete":0,              ClientRes5xxTimeRes="0"                
       "ClientRes5xxCount":0,                     ClientRes5xxTimeComplete="0"           
       "ClientRes5xxTimeRes":0,                   RequestHitRatio="6923"                 
       "ClientRes5xxTimeComplete":0,              ByteHitRatio="4243">                   
       "RequestHitRatio":6387,                    <HttpCountSum                          
       "ByteHitRatio":2926,                         OriginReqCount="0"                   
       "HttpCountSum" :                             OriginResTotalCount="0"              
       {                                            OriginRes2xxCount="0"                
         "OriginReqCount" : 0,                      OriginRes3xxCount="0"                
         "OriginResTotalCount" : 0,                 OriginRes4xxCount="0"                
         "OriginRes2xxCount" : 0,                   OriginRes5xxCount="0"                
         "OriginRes3xxCount" : 0,                   ClientReqCount="0"                   
         "OriginRes4xxCount" : 0,                   ClientResTotalCount="0"              
         "OriginRes5xxCount" : 0,                   ClientRes2xxCount="0"                
         "ClientReqCount" : 0,                      ClientRes3xxCount="0"                
         "ClientResTotalCount" : 0,                 ClientRes4xxCount="0"                
         "ClientRes2xxCount" : 0,                   ClientRes5xxCount="0"/>              
         "ClientRes3xxCount" : 0,                 <HttpRequestHitSum                     
         "ClientRes4xxCount" : 0,                   TCP_NONE="0"                         
         "ClientRes5xxCount" : 0                    TCP_HIT="0"                          
       },                                           TCP_IMS_HIT="0"                      
       "HttpRequestHitSum" :                        TCP_REFRESH_HIT="0"                  
       {                                            TCP_REF_FAIL_HIT="0"                 
         "TCP_NONE" : 0,                            TCP_NEGATIVE_HIT="0"                 
         "TCP_HIT" : 0,                             TCP_REDIRECT_HIT="0"                 
         "TCP_IMS_HIT" : 0,                         TCP_MISS="0"                         
         "TCP_REFRESH_HIT" : 0,                     TCP_REFRESH_MISS="0"                 
         "TCP_REF_FAIL_HIT" : 0,                    TCP_CLIENT_REFRESH_MISS="0"          
         "TCP_NEGATIVE_HIT" : 0,                    TCP_DENIED="0"                       
         "TCP_REDIRECT_HIT" : 0,                    TCP_ERROR="0"/>                      
         "TCP_MISS" : 0,                          <FileSystem>                           
         "TCP_REFRESH_MISS" : 0,                    <RequestHitRatio>0</RequestHitRatio> 
         "TCP_CLIENT_REFRESH_MISS" : 0,             <ByteHitRatio>0</ByteHitRatio>       
         "TCP_DENIED" : 0,                          <Outbound>0</Outbound>               
         "TCP_ERROR" : 0                            <Session>0</Session>                 
       },                                         </FileSystem>                          
       "FileSystem":                              <System> ... </System>                 
       {                                          <VirtualHost> ... </VirtualHost>       
         "RequestHitRatio":0,                     <VirtualHost> ... </VirtualHost>       
         "ByteHitRatio":0,                        <VirtualHost> ... </VirtualHost>       
         "Outbound":0,                            <View> ... </View>                     
         "Session":0                              <View> ... </View>                     
       },                                       </Host>                                  
       "System":{ ... },                                                                 
       "VirtualHost": [ ... ]
       "View": [ ... ]
     }
   }
   
-  ``Version`` STON version
-  ``Name`` The host name. If not defined, the system name will be stated.
-  ``State`` Service status. (Healthy=Normal service, Inactive=Inactive license, Emergency)
-  ``Uptime (unit: seconds)`` The running time of the service
-  ``OriginSession`` The number of origin sessions
-  ``OriginActiveSession`` The number of transmitting origin sessions
-  ``OriginInbound (unit: Bytes, average)`` The amount of data received from the origin server
-  ``OriginReqCount (average)`` The number of requests sent to the origin server
-  ``OriginOutbound (unit: Bytes, average)`` The amount of transmitted data to the origin server
-  ``OriginResTotalCount (average)`` The number of responses from the origin server
-  ``OriginResTotalTimeRes (unit: 0.01ms, average)`` The response time of the origin server (HTTP request ~ First HTTP response)
-  ``OriginResTotalTimeComplete (unit: 0.01ms, average)`` The HTTP transaction completion time of the origin server (transfer HTTP request ~ complete HTTP response)
-  ``OriginRes2xxCount (average)`` The number of 2xx responses of the origin server
-  ``OriginRes2xxTimeRes (unit: 0.01ms, average)`` The 2xx response time of the origin server
-  ``OriginRes2xxTimeComplete (unit: 0.01ms, average)`` The 2xx transaction completion time of the origin server
-  ``OriginRes3xxCount (average)`` The number of 3xx responses of the origin server
-  ``OriginRes3xxTimeRes (unit: 0.01ms, average)`` The 3xx response time of the origin server
-  ``OriginRes3xxTimeComplete (unit: 0.01ms, average)`` The 3xx transaction completion time of the origin server
-  ``OriginRes4xxCount (average)`` The number of 4xx responses of the origin server
-  ``OriginRes4xxTimeRes (unit: 0.01ms, average)`` The 4xx response time of the origin server
-  ``OriginRes4xxTimeComplete (unit: 0.01ms, average)`` The 4xx transaction completion time of the origin server
-  ``OriginRes5xxCount (average)`` The number of 5xx responses of the origin server
-  ``OriginRes5xxTimeRes (unit: 0.01ms, average)`` The 5xx response time of the origin server
-  ``OriginRes5xxTimeComplete (unit: 0.01ms, average)`` The 5xx transaction completion time of the origin server
-  ``ClientSession`` The number of client sessions
-  ``ClientActiveSession`` The number of transmitting client sessions
-  ``ClientInbound (unit: Bytes, average)`` The amount of inbound data from clients
-  ``ClientOutbound (unit: Bytes, average)`` The amount of outbound data to clients
-  ``ClientReqCount (average)`` The number of client requests
-  ``ClientResTotalCount (average)`` The number of client responses
-  ``ClientResTotalTimeRes (unit: 0.01ms, average)`` Client response times (HTTP request reception ~ transfer HTTP response)
-  ``ClientResTotalTimeComplete (unit: 0.01ms, average)`` Client HTTP transaction completion time (HTTP request reception ~ complete HTTP response)
-  ``ClientRes2xxCount (average)`` The number of client 2xx responses
-  ``ClientRes2xxTimeRes (unit: 0.01ms, average)`` The 2xx response time of the origin server
-  ``ClientRes2xxTimeComplete (unit: 0.01ms, average)`` The 2xx transaction completion time of the client
-  ``ClientRes3xxCount (average)`` The number of client 3xx responses
-  ``ClientRes3xxTimeRes (unit: 0.01ms, average)`` The 3xx response time of the origin server
-  ``ClientRes3xxTimeComplete (unit: 0.01ms, average)`` The 3xx transaction completion time of the client
-  ``ClientRes4xxCount (average)`` The number of client 4xx responses
-  ``ClientRes4xxTimeRes (unit: 0.01ms, average)`` The 4xx response time of the origin server
-  ``ClientRes4xxTimeComplete (unit: 0.01ms, average)`` The 4xx transaction completion time of the client
-  ``ClientRes5xxCount (average)`` The number of client 5xx responses
-  ``ClientRes5xxTimeRes (unit: 0.01ms, average)`` The 4xx response time of the origin server
-  ``ClientRes5xxTimeComplete (unit: 0.01ms, average)`` The 5xx transaction completion time of the client
-  ``RequestHitRatio (unit: 0.01%, average)`` Hit ratio. 
   If a caching object is created and the corresponding object is initialized, then it is considered as a Hit. 
   On the other hand, if the caching object is missing or the object is not initialized from the origin server, it does not count as a Hit. 
   The response code does not have to do with the hit ratio.
   
   .. figure:: img/stat_filesystem1.png
      :align: center
      
      HTTP and File I/O share the virtual host.
      
   The RequestHitRatio of the File I/O that is accessed via Apache becomes 0%.
   The HTTP server, on the other hand, has a 100% RequestHitRatio because it accesses cached files via File I/O. 
   ByteHitRatio is calculated by the ratio of the origin inbound to either Http outbound(when the HTTP server is used as the origin) or File I/O outbound(when the filesystem is applied).
   
-  ``ByteHitRatio (unit: 0.01%, average)`` The transfer ratio of the origin server to the client. ::

      (Client Outbound - Origin server Inbound) / Client Outbound
      
   Even if the origin server has a much faster transfer speed, the client session closes quickly, and the total ratio becomes a negative value.
   
-  ``FileSystem`` A separate FileSystem of stats that do not accumulate with other stat values.

   - ``RequestHitRatio (unit: 0.01%, average)`` Requests the hit ratio via File I/O
   - ``ByteHitRatio (unit: 0.01%, average)`` Transfers the ratio of the origin server to File I/O
   - ``Outbound (unit: Bytes, average)`` Requests the outbound data size via File I/O
   - ``Session (average)`` The number of threads in the File I/O process
      
.. note::

   Items that are only provided statistics in five minute increments.
   
   -  ``HttpCountSum`` The total number of HTTP transactions   
   -  ``HttpRequestHitSum`` The cache HIT result
   
   
System Statistics
====================================

The stats of system and global resources are provided in JSON and XML formats. ::

    "System":                                   <System>                                          
    {                                             <CPU                                            
      "CPU":                                          Kernel="689"                                
      {                                               User="1316"                                 
        "Kernel":689,                                 Idle="7993"                                 
        "User":1316,                                  ProcKernel="570"                            
        "Idle":7993,                                  ProcUser="1216"                             
        "ProcKernel":570,                             Nice="0"                                    
        "ProcUser":1216,                              IOWait="52"                                 
        "Nice":0,                                     IRQ="10"                                    
        "IOWait":52,                                  SoftIRQ="12"                                
        "IRQ":10,                                     Steal="0" />                                
        "SoftIRQ":12,                             <Mem Free="5914644" STON="9785800"/>            
        "Steal":0                                 <Storage>                                       
      },                                            <Disk                                         
      "Mem":                                        	Path="/cache1"                            
      {                                             	Status="Normal"                           
        "Free":5914644,                             	Read="23"                                 
        "STON":9785800                              	ReadMerged="0"                            
      },                                            	ReadSectors="344"                         
      "Storage":                                    	ReadTime="117"                            
      {                                             	Write="24"                                
        "Disk":                                     	WriteMerged="93"                          
        [                                           	WriteSectors="936"                        
          {                                         	WriteTime="256"                           
            "Path":"/cache1",                       	IOProgress="0"                            
            "Status":"Normal",                      	IOTime="173"                              
            "Read":23,                              	IOWeightedTime="373"/>                    
            "ReadMerged":0,                         <Disk                                         
            "ReadSectors":344,                      	Path="/cache2"                            
            "ReadTime":117,                         	Status="Normal"                           
            "Write":24,                             	Read="27"                                 
            "WriteMerged":93,                       	ReadMerged="1"                            
            "WriteSectors":936,                     	ReadSectors="488"                         
            "WriteTime":256,                        	ReadTime="144"                            
            "IOProgress":0,                         	Write="24"                                
            "IOTime":173,                           	WriteMerged="86"                          
            "IOWeightedTime":373                    	WriteSectors="880"                        
          },                                        	WriteTime="254"                           
          {                                         	IOProgress="0"                            
            "Path":"/cache2",                       	IOTime="189"                              
            "Status":"Normal",                      	IOWeightedTime="380"/>                    
            "Read":27,                            </Storage>                                      
            "ReadMerged":1,                       <ServerSocket                                   
            "ReadSectors":488,                    	Total="42"                                    
            "ReadTime":144,                       	Established="2"                               
            "Write":24,                            	Accepted="1"                                  
            "WriteMerged":86,                      	Closed="0"/>                                  
            "WriteSectors":880,                   <ClientSocket                                   
            "WriteTime":254,                       	Total="1"                                     
            "IOProgress":0,                        	Established="0"                               
            "IOTime":189,                          	Connected="0"                                 
            "IOWeightedTime":380                   	Closed="0"/>                                  
          }                                       <TCPSocket                                      
        ]                                          	Established="30"                              
      },                                           	Timewait="2"                                  
      "ServerSocket":                              	Orphan="0"                                    
      {                                            	Alloc="0"                                     
        "Total":42,                                	Mem="20"/>                                    
        "Established":1,                          <EQ>0</EQ>                                      
        "Accepted":0,                             <RQ>1000000</RQ>                                
        "Closed":0                                <WaitingFiles2Write>0</WaitingFiles2Write>      
      },                                          <ServiceAccess Allow="60" Deny="2"/>            
      "ClientSocket":                             <SystemLoadAverage Min1="0" Min5="0" Min15="0"/>
      {                                           <URLRewrite>57</URLRewrite>                     
        "Total":1,                              </System>                                         
        "Established":0,
        "Connected":0,
        "Closed":0
      },
      "TCPSocket":
      {
        "Established":30,
        "Timewait":2,
        "Orphan":0,
        "Alloc":0,
        "Mem":20
      },
      "EQ":0,
      "RQ":1000000,
      "WaitingFiles2Write":0,
      "ServiceAccess":{"Allow":60, "Deny":2}
      "SystemLoadAverage":
      {
        "Min1":0,
        "Min5":0,
        "Min15":0
      },
      "URLRewrite":57
    }

-  ``CPU (unit: 0.01%)`` CPU usage. Total CPU usage can be calculated by Kernel + User.
   
   - ``Kernel`` CPU(Kernel) usage
   - ``User`` CPU(User) usage
   - ``Idle`` Idle CPU
   - ``ProcKernel`` STON's CPU(Kernel) usage
   - ``ProcUser`` STON's CPU(User) usage
   - ``Nice`` Processes executing in user mode
   - ``IOWait`` Waits for I/O to complete
   - ``IRQ`` Interrupts service
   - ``SoftIRQ`` Services softIRQs
   - ``Steal`` An involuntary wait

-  ``Mem (unit: Bytes)`` Memory usage.
   - ``Free`` The size of free Memory in the system.
   - ``STON`` STON's memory usage
   
-  ``Disk`` Disk performance stats

   - ``Path`` Disk path
   - ``Status`` Disk status (Normal: normal, Invalid: excluded due to failure, Unmounted: unmounted by the administrator)
   - ``Read`` The number of successful reads
   - ``ReadMerged`` The number of accumulated read
   - ``ReadSectors`` The number of read sectors
   - ``ReadTime (unit: ms)`` The elapsed time per read
   - ``Write`` The number of successful writes
   - ``WriteMerged`` The number of accumulated write
   - ``WriteSectors`` The number of written sectors
   - ``WriteTime (unit: ms)`` The elapsed time per write
   - ``IOProgress`` The number of progressing IOs
   - ``IOTime (unit: ms)`` The elapsed time per IO
   - ``IOWeightedTime (unit: ms)`` The elapsed time per IO(Weight applied)
      
-  ``ServerSocket`` Server socket(between clients and STON) information

   - ``Total`` The total number of server sockets
   - ``Established`` The number of connected server sockets
   - ``Accepted`` The number of newly connected server sockets
   - ``Closed`` The number of closed server sockets
   
-  ``ClientSocket`` Client socket(between STON and the origin server) information

   - ``Total`` The total number of client sockets
   - ``Established`` The number of connected client sockets
   - ``Connected`` The number of newly connected client sockets
   - ``Closed`` The number of closed client sockets
   
-  ``TCPSocket`` TCP status information provided by the system(OS)

   - ``Established`` The number of established status TCP connections
   - ``Timewait`` The number of TIME_WAIT status TCP connections
   - ``Orphan`` The number of TCP connections that have not been attached to the file handle
   - ``Alloc`` An allocated TCP connection
   - ``Mem`` An undocumented TCP connection
   
-  ``EQ`` The number of unprocessed events in the STON Framework
-  ``RQ`` The number of events that are saved in the recently serviced content reference queue
-  ``WaitingFiles2Write`` The number of disk write pending files
-  ``ServiceAccess`` The number of allowed(Allow) and denied(Deny) sockets by ServiceAccess
-  ``SystemLoadAverage`` The 1/5/15 minute average of the System Load Average
-  ``URLRewrite`` The number of successful conversions made by the URL preprocessor
    
    
Virtual Host Statistics
====================================

Each virtual host provides statistics. 
There are four types of virtual host statistics: HTTP transfer(per directory), URL bypass, port bypass and SSL. ::

   "VirtualHost":                               <VirtualHost                                              
   [                                                Name="image.11st.co.kr"                               
     {                                              Uptime="155956"                                       
       "Name":"image.11st.co.kr",                   OriginSession="12"                                    
       "Uptime":155966,                             OriginActiveSession="6"                               
       "OriginSession":12,                          OriginInbound="106914"                                
       "OriginActiveSession":6,                     OriginOutbound="3238"                                 
       "OriginInbound":169,                         OriginReqCount="42"                                   
       "OriginOutbound":269,                        OriginResTotalCount="13"                              
       "OriginReqCount":62,                         OriginResTotalTimeRes="1553"                          
       "OriginResTotalCount":1,                     OriginResTotalTimeComplete="6630"                     
       "OriginResTotalTimeRes":3300,                OriginRes2xxCount="1"                                 
       "OriginResTotalTimeComplete":3300,           OriginRes2xxTimeRes="3300"                            
       "OriginRes2xxCount":0,                       OriginRes2xxTimeComplete="69300"                      
       "OriginRes2xxTimeRes":0,                     OriginRes3xxCount="12"                                
       "OriginRes2xxTimeComplete":0,                OriginRes3xxTimeRes="1408"                            
       "OriginRes3xxCount":1,                       OriginRes3xxTimeComplete="1408"                       
       "OriginRes3xxTimeRes":3300,                  OriginRes4xxCount="0"                                 
       "OriginRes3xxTimeComplete":3300,             OriginRes4xxTimeRes="0"                               
       "OriginRes4xxCount":0,                       OriginRes4xxTimeComplete="0"                          
       "OriginRes4xxTimeRes":0,                     OriginRes5xxCount="0"                                 
       "OriginRes4xxTimeComplete":0,                OriginRes5xxTimeRes="0"                               
       "OriginRes5xxCount":0,                       OriginRes5xxTimeComplete="0"                          
       "OriginRes5xxTimeRes":0,                     ClientSession="30"                                    
       "OriginRes5xxTimeComplete":0,                ClientActiveSession="12"                              
       "ClientSession":26,                          ClientInbound="4113"                                  
       "ClientActiveSession":16,                    ClientOutbound="895937"                               
       "ClientInbound":13968,                       ClientReqCount="64"                                   
       "ClientOutbound":110398,                     ClientResTotalCount="18"                              
       "ClientReqCount":152,                        ClientResTotalTimeRes="666"                           
       "ClientResTotalCount":52,                    ClientResTotalTimeComplete="4377"                     
       "ClientResTotalTimeRes":94,                  ClientRes2xxCount="10"                                
       "ClientResTotalTimeComplete":107,            ClientRes2xxTimeRes="1200"                            
       "ClientRes2xxCount":1,                       ClientRes2xxTimeComplete="7870"                       
       "ClientRes2xxTimeRes":4700,                  ClientRes3xxCount="8"                                 
       "ClientRes2xxTimeComplete":4800,             ClientRes3xxTimeRes="0"                               
       "ClientRes3xxCount":51,                      ClientRes3xxTimeComplete="12"                         
       "ClientRes3xxTimeRes":3,                     ClientRes4xxCount="0"                                 
       "ClientRes3xxTimeComplete":15,               ClientRes4xxTimeRes="0"                               
       "ClientRes4xxCount":0,                       ClientRes4xxTimeComplete="0"                          
       "ClientRes4xxTimeRes":0,                     ClientRes5xxCount="0"                                 
       "ClientRes4xxTimeComplete":0,                ClientRes5xxTimeRes="0"                               
       "ClientRes5xxCount":0,                       ClientRes5xxTimeComplete="0"                          
       "ClientRes5xxTimeRes":0,                     RequestHitRatio="10000"                               
       "ClientRes5xxTimeComplete":0,                ByteHitRatio="8806">                                  
       "RequestHitRatio":10000,                   <FileSystem>                                            
       "ByteHitRatio":9984,                         <RequestHitRatio>0</RequestHitRatio>                  
       "FileSystem":                                <ByteHitRatio>0</ByteHitRatio>                        
       {                                            <Outbound>0</Outbound>                                
         "RequestHitRatio":0,                       <Session>0</Session>                                  
         "ByteHitRatio":0,                        </FileSystem>                                           
         "Outbound":0,                            <Memory>784786700</Memory>                              
         "Session":0                              <SecuredMemory>0</SecuredMemory>                        
       },                                         <Disk> ... </Disk>                                      
       "Memory":785740769,                        <Session> ... </Session>                                
       "SecuredMemory":0,                         <File Total="458278" Opened="15" Instance="458292"/>    
       "Disk": { ... },                           <Cached> ... </Cached>                                  
       "Session": { ... },                        <CacheFileEvent> ... </CacheFileEvent>                  
       "FileTotal":458308,                        <WaitingFiles2Delete>1087593</WaitingFiles2Delete>      
       "FileOpened":15,                           <ClientHttpReqBypass Sum="8100">27</ClientHttpReqBypass>
       "FileInstance":458320,                     <ClientHttpReqDenied Sum="400">1</ClientHttpReqDenied>  
       "Cached": { ... },                         <OriginTraffic> ... </OriginTraffic>                    
       "CacheFileEvent": { ... },                 <PortBypass> ... </PortBypass>                          
       "WaitingFiles2Delete":1087595,             <ClientTraffic> ... </ClientTraffic>                    
       "ClientHttpReqBypassSum":8100,             <UrlBypass> ... </UrlBypass>                            
       "ClientHttpReqBypass":27,                </VirtualHost>                                            
       "ClientHttpReqDeniedSum":400,            <VirtualHost> ... </VirtualHost>                          
       "ClientHttpReqDenied":1,                 <VirtualHost> ... </VirtualHost>                          
       "OriginTraffic": { ... },                <VirtualHost> ... </VirtualHost>                          
       "PortBypass": { ... },
       "ClientTraffic": { ... },
       "UrlBypass": { ... },
     },
     ...
   ]
   
.. note:

   ※ Items from Name to FileSystem are identical to those from the host statistics.
   
-  ``Memory (unit: Bytes)`` The amount of content that is loaded into the memory
-  ``SecuredMemory (unit: Bytes)`` The amount of content that is discarded from memory
-  ``Disk`` Disk information
-  ``Session`` Session information
-  ``FileTotal`` The total number of files
-  ``FileOpened`` The number of opened local files
-  ``FileInstance`` The number of caching files
-  ``Cached`` Caching information
-  ``CacheFileEvent`` A caching file event
-  ``WaitingFiles2Delete`` The number of files pending deletion
-  ``ClientHttpReqBypass`` The number of bypassed client HTTP requests
-  ``ClientHttpReqDenied`` The number of denied HTTP requests
-  ``OriginTraffic`` Origin server traffic stats
-  ``PortBypass`` Port bypass traffic stats
-  ``ClientTraffic`` Client traffic stats
-  ``UrlBypass`` HTTP traffic stats that are bypassed to the origin server via URL matching or ``<BypassNoCacheRequest>``

.. note::

   Items that are only provided statistics in five minute increments.
   
   -  ``ClientHttpReqBypassSum`` The total number of bypassed HTTP requests   
   -  ``ClientHttpReqDeniedSum`` The total number of denied HTTP requests


Disk Statistics
------------------------------

Provides statistics of the disk that the virtual host uses. ::

   "Disk":                                      <Disk>                              
   {                                              <TotalSize>22003701435</TotalSize>
     "TotalSize":22004057982,                     <Create>1</Create>                
     "Create":0,                                  <Open>10</Open>                   
     "Open":1,                                    <Delete>0</Delete>                
     "Delete":0,                                  <ReadCount>9</ReadCount>          
     "ReadCount":1,                               <ReadSize>735726</ReadSize>       
     "ReadSize":104744,                           <WriteCount>1</WriteCount>        
     "WriteCount":0,                              <WriteSize>157145</WriteSize>     
     "WriteSize":0,                               <Distribution                     
     "Distribution":                                U1K="45725"                    
     {                                              U2K="192523"  
       "U1K="45725,                                 U4K="137055"                   
       "U2K="192523,                                U8K="39740"                   
       "U4K="137055,                                U16K="13408"                  
       "U8K="39740,                                 U32K="12303"                  
       "U16K="13408,                                U64K="11462"                  
       "U32K="12303,                                U128K="2560"                  
       "U64K="11462,                                U256K="22"                      
       "U128K="2560,                                U512K="0"                       
       "U256K="22,                                  U1M="45725"                    
       "U512K="0,                                   U2M="192523"                    
       "U1M="45725,                                 U4M="137055"                   
       "U2M="192523,                                U8M="39740"                   
       "U4M="137055,                                U16M="13408"                  
       "U8M="39740,                                 U32M="12303"                  
       "U16M="13408,                                U64M="11462"                  
       "U32M="12303,                                U128M="2560"                  
       "U64M="11462,                                U256M="22"                      
       "U128M="2560,                                U512M="0"                       
       "U256M="22,                                  U1G="0"                        
       "U512M="0,                                   U2G="0"                         
       "U1G="0,                                     U4G="0"                       
       "U2G="0,                                     U8G="0"                       
       "U4G="0,                                     U16G="0"                      
       "U8G="0,                                     O16G="0" />
       "U16G":0,                                  </Disk>
       "O16G":0                                 

     }
   }

-  ``TotalSize (unit: Bytes)`` The total size of local files
-  ``Create`` The number of created local files
-  ``Open`` The number of opened local files
-  ``Delete`` The number of deleted local files
-  ``ReadCount`` The number of times that a local file is read
-  ``ReadSize (unit: Bytes)`` The total size of read local files
-  ``WriteCount`` The number of times that a local file is written
-  ``WriteSize (unit: Bytes)`` The total size of written local files
-  ``Distribution`` Distribution of local files based on the size

   - ``U1K`` The number of files that are less than 1KB
   - ``U2K`` The number of files that are less than 2KB
   - ``U4K`` The number of files that are less than 4KB
   - ``U8K`` The number of files that are less than 8KB
   - ``U16K`` The number of files that are less than 16KB
   - ``U32K`` The number of files that are less than 32KB
   - ``U64K`` The number of files that are less than 64KB
   - ``U128K`` The number of files that are less than 128KB
   - ``U256K`` The number of files that are less than 256KB
   - ``U512K`` The number of files that are less than 512KB
   - ``U1M`` The number of files that are less than 1MB
   - ``U2M`` The number of files that are less than 2MB
   - ``U4M`` The number of files that are less than 4MB
   - ``U8M`` The number of files that are less than 8MB
   - ``U16M`` The number of files that are less than 16MB
   - ``U32M`` The number of files that are less than 32MB
   - ``U64M`` The number of files that are less than 64MB
   - ``U128M`` The number of files that are less than 128MB
   - ``U256M`` The number of files that are less than 256MB
   - ``U512M`` The number of files that are less than 512MB
   - ``U1G`` The number of files that are less than 1GB
   - ``U2G`` The number of files that are less than 2GB
   - ``U4G`` The number of files that are less than 4GB
   - ``U8G`` The number of files that are less than 8GB
   - ``U16G`` The number of files that are less than 16GB
   - ``O16G`` The number of files that are less than 16GB


Session Statistics
------------------------------

Provides statistics of the disk that the session stattistics uses. ::

   "Session":                                   <Session            
   {                                              Client="30"       
     "Client":30,                                 ActiveClient="20" 
     "ActiveClient":20,                           Origin="12"       
     "Origin":12,                                 ActiveOrigin="7" />
     "ActiveOrigin":7
   },
   
-  ``Client`` The number of total HTTP client sessions
-  ``ActiveClient`` The number of transmitting sessions among HTTP clients
-  ``Origin`` The number of total origin server sessions
-  ``ActiveOrigin`` The number of transmitting sessions among origin server sessions



Origin Server Statistics
------------------------------

Provides stats for traffic between STON and the origin server. ::

   "OriginTraffic":                             <OriginTraffic>                                  
   {                                              <HttpReqCount Sum="600">2</HttpReqCount>       
     "HttpReqCountSum":0,                         <HttpReqHeaderSize>3238</HttpReqHeaderSize>    
     "HttpReqCount":0,                            <HttpReqBodySize>0</HttpReqBodySize>           
     "HttpReqHeaderSize":269,                     <HttpResHeaderSize>2020</HttpResHeaderSize>    
     "HttpReqBodySize":0,                         <HttpResBodySize>104894</HttpResBodySize>      
     "HttpResHeaderSize":169,                     <Response>                                     
     "HttpResBodySize":0,                           <ResTotal>                                   
     "Response":                                      <Count Sum="8100">13</Count>               
     {                                                <Completed Sum="8100">12</Completed>       
       "ResTotal":                                    <TimeRes>1553</TimeRes>                    
       {                                              <TimeComplete>6630</TimeComplete>          
         "CountSum":0,                              </ResTotal>                                  
         "Count":1,                                 <Res2xx>                                     
         "CompletedSum":0,                            <Count Sum="8100">1</Count>                
         "Completed":1,                               <Completed Sum="8100">1</Completed>        
         "TimeRes":3300,                              <TimeRes>3300</TimeRes>                    
         "TimeComplete":3300                          <TimeComplete>69300</TimeComplete>         
       },                                           </Res2xx>                                    
       "Res2xx":                                    <Res3xx>                                     
       {                                              <Count Sum="8100">12</Count>               
         "CountSum":0,                                <Completed Sum="8100">11</Completed>       
         "Count":0,                                   <TimeRes>1408</TimeRes>                    
         "CompletedSum":0,                            <TimeComplete>1408</TimeComplete>          
         "Completed":0,                             </Res3xx>                                    
         "TimeRes":0,                               <Res4xx>                                     
         "TimeComplete":0                             <Count Sum="8100">0</Count>                
       },                                             <Completed Sum="8100">0</Completed>        
       "Res3xx":                                      <TimeRes>0</TimeRes>                       
       {                                              <TimeComplete>0</TimeComplete>             
         "CountSum":0,                              </Res4xx>                                    
         "Count":1,                                 <Res5xx>                                     
         "CompletedSum":0,                            <Count Sum="8100">0</Count>                
         "Completed":1,                               <Completed Sum="8100">0</Completed>        
         "TimeRes":3300,                              <TimeRes>0</TimeRes>                       
         "TimeComplete":3300                          <TimeComplete>0</TimeComplete>             
       },                                           </Res5xx>                                    
       "Res4xx":                                    <ConnectTimeout Sum="8100">0</ConnectTimeout>
       {                                            <ReceiveTimeout Sum="8100">0</ReceiveTimeout>
         "CountSum":0,                              <Close Sum="8100">0</Close>                  
         "Count":0,                               </Response>                                    
         "CompletedSum":0,                        <Connect>                                      
         "Completed":0,                             <Count>0</Count>                             
         "TimeRes":0,                               <AvgDNSQueryTime>0</AvgDNSQueryTime>         
         "TimeComplete":0                           <AvgConnTime>0</AvgConnTime>                 
       },                                         </Connect>                                     
       "Res5xx":                                </OriginTraffic>                                 
       {
         "CountSum":0,
         "Count":0,
         "CompletedSum":0,
         "Completed":0,
         "TimeRes":0,
         "TimeComplete":0
       },
       "ConnectTimeoutSum":0,
       "ConnectTimeout":0,
       "ReceiveTimeoutSum":0,
       "ReceiveTimeout":0,
       "CloseSum":0,
       "Close":0
     },
     "Connect":
     {
       "Count":0,
       "AvgDNSQueryTime":0,
       "AvgConnTime":0
     }
   },

-  ``HttpReqCount`` The number of HTTP requests sent to the origin server
-  ``HttpReqHeaderSize (unit: Bytes)`` The size of the HTTP header sent to the origin server
-  ``HttpReqBodySize (unit: Bytes)`` The size of the HTTP Body sent to the origin server
-  ``HttpResHeaderSize (unit: Bytes)`` The size of the HTTP header received by the origin server
-  ``HttpResBodySize (unit: Bytes)`` The size of the HTTP Body received by the origin server
-  ``Response`` The response from the origin server (ResXXX)

   -  ``Count`` The number of responses
   -  ``Completed`` The number of completely transferred HTTP transactions
   -  ``TimeRes`` The HTTP response time
   -  ``TimeComplete`` The completion time of the HTTP transaction

-  ``Response`` Origin server connection error
   
   -  ``ConnectTimeout`` A connection failure
   -  ``ReceiveTimeout`` A transmission delay
   -  ``Close`` The origin server closed socket during the transfer
   
-  ``Connect`` Origin server connection stats

   -  ``Count`` The number of connections
   -  ``AvgDNSQueryTime (unit: 0.01ms)`` The average DNS query time
   -  ``AvgConnTime (unit: 0.01ms)`` The average connection time (TCP SYN transmission ~ TCP SYN ACK reception)
   
.. note::

   Items that are only provided statistics in five minute increments.
   
   -  ``HttpReqCountSum`` The total number of HTTP requests
   -  ``CountSum`` The total number of HTTP responses
   -  ``CompletedSum`` The total number of completed HTTP transactions
   -  ``ConnectTimeoutSum`` The total number of origin server connection failures
   -  ``ReceiveTimeoutSum`` The total number of origin server transmission delays
   -  ``CloseSum`` The total number of connections closed by the origin server
      
   

Port Bypass Statistics
------------------------------

Provides traffic stats via ``<PortBypass>``. ::

   "PortBypass":                                            <PortBypass SrcPort="1935" DestPost="1935">
   [                                                          <Session>0</Session>                     
     {                                                        <Hit Established="0"                     
       "SrcPort":1935, "DestPort":1935, "Session":0,               ClientClosed="0"                    
       "Hit":                                                      OriginClosed="0"                    
       {                                                           OriginCT="0" />                     
         "Established":0, "ClientClosed":0,                   <ClientTraffic In="0" Out="0"/>          
         "OriginClosed":0, "OriginCT":0                       <OriginTraffic In="0" Out="0"/>          
       },                                                   </PortBypass>                              
       "ClientTraffic": { "In":0, "Out":0 },                <PortBypass SrcPort="1936" DestPost="1936">
       "OriginTraffic": { "In":0, "Out":0 }                   <Session>17</Session>                    
     },                                                       ...                                      
     {                                                      </PortBypass>                              
       "SrcPort":1936, "DestPort":1936, "Session":17,
       ...
     }
   ],
   
   
-  ``SrcPort/DestPort`` Bypassed the STON port/origin server port
-  ``Session`` The total number of connected sessions
-  ``Hit`` Bypass connection stats

   -  ``Established`` The total number of established connections
   -  ``ClientClosed`` The total number of connections closed by clients
   -  ``OriginClosed`` The total number of connections closed by the origin server
   -  ``OriginCT`` The total number of origin server connection failures

-  ``ClientTraffic (unit: Bytes)`` Client traffic (In=Inbound, Out=Outbound)
-  ``OriginTraffic (unit: Bytes)`` Origin server traffic (In=Inbound, Out=Outbound)



Client Statistics
------------------------------

Client traffic is stated as multiple "Traffic" based on the stats configuration for each directory.
If stats for each directory have not been set, all traffic is counted in the root(/). 
If directory stats have been set, only the root(/) and directories involved with the traffic will be counted. ::

   "ClientTraffic":                             <ClientTraffics Depth="0" Accum="OFF" HttpsTraffic="OFF">
   {                                              <TrafficCount>1</TrafficCount>                         
     "Depth":0,                                   <Traffic RequestHitRatio="0">                          
     "Accum":"OFF",                                 <Path>/</Path>                                       
     "HttpsTraffic":"OFF",                          <HttpReqCount Sum="0">0</HttpReqCount>               
     "TrafficCount":1,                              <HttpReqHeaderSize>4113</HttpReqHeaderSize>          
     "Traffic":                                     <HttpReqBodySize>0</HttpReqBodySize>                 
     [                                              <HttpResHeaderSize>3066</HttpResHeaderSize>          
       {                                            <HttpResBodySize>892871</HttpResBodySize>            
         "RequestHitRatio" : 9984,                  <Response>                                           
         "Path":"/",                                  <ResTotal>                                         
         "HttpReqCountSum":0,                           <Count Sum="0">18</Count>                        
         "HttpReqCount":100,                            <Completed Sum="0">18</Completed>                
         "HttpReqHeaderSize":13968,                     <TimeRes>666</TimeRes>                           
         "HttpReqBodySize":0,                           <TimeComplete>4377</TimeComplete>                
         "HttpResHeaderSize":5654,                    </ResTotal>                                        
         "HttpResBodySize":104744,                    <Res2xx>                                           
         "Response":                                    <Count Sum="0">10</Count>                        
         {                                              <Completed Sum="0">10</Completed>                
           "ResTotal":                                  <TimeRes>1200</TimeRes>                          
           {                                            <TimeComplete>7870</TimeComplete>                
             "CountSum":0,                            </Res2xx>                                          
             "Count":52,                              <Res3xx>                                           
             "CompletedSum":0,                          <Count Sum="0">8</Count>                         
             "Completed":52,                            <Completed Sum="0">8</Completed>                 
             "TimeRes":94,                              <TimeRes>0</TimeRes>                             
             "TimeComplete":107                         <TimeComplete>12</TimeComplete>                  
           },                                         </Res3xx>                                          
           "Res2xx":                                  <Res4xx>                                           
           {                                            <Count Sum="0">0</Count>                         
             "CountSum":0,                              <Completed Sum="0">0</Completed>                 
             "Count":1,                                 <TimeRes>0</TimeRes>                             
             "CompletedSum":0,                          <TimeComplete>0</TimeComplete>                   
             "Completed":1,                           </Res4xx>                                          
             "TimeRes":4700,                          <Res5xx>                                           
             "TimeComplete":4800                        <Count Sum="0">0</Count>                         
           },                                           <Completed Sum="0">0</Completed>                 
           "Res3xx":                                    <TimeRes>0</TimeRes>                             
           {                                            <TimeComplete>0</TimeComplete>                   
             "CountSum":0,                            </Res5xx>                                          
             "Count":51,                            </Response>                                          
             "CompletedSum":0,                      <SSL RecvSize="0" SendSize="0"/>                     
             "Completed":51,                        <RequestHit                                          
             "TimeRes":3,                             TCP_NONE="0"                                       
             "TimeComplete":15                        TCP_HIT="0"                                        
           },                                         TCP_IMS_HIT="0"                                    
           "Res4xx":                                  TCP_REFRESH_HIT="0"                                
           {                                          TCP_REF_FAIL_HIT="0"                               
             "CountSum":0,                            TCP_NEGATIVE_HIT="0"                               
             "Count":0,                               TCP_REDIRECT_HIT="0"                               
             "CompletedSum":0,                        TCP_MISS="0"                                       
             "Completed":0,                           TCP_REFRESH_MISS="0"                               
             "TimeRes":0,                             TCP_CLIENT_REFRESH_MISS="0"                        
             "TimeComplete":0                         TCP_DENIED="0"                                     
           },                                         TCP_ERROR="0"/>                                    
           "Res5xx":                                <RequestHitSum                                       
           {                                          TCP_NONE="0"                                       
             "CountSum":0,                            TCP_HIT="0"                                        
             "Count":0,                               TCP_IMS_HIT="0"                                    
             "CompletedSum":0,                        TCP_REFRESH_HIT="0"                                
             "Completed":0,                           TCP_REF_FAIL_HIT="0"                               
             "TimeRes":0,                             TCP_NEGATIVE_HIT="0"                               
             "TimeComplete":0                         TCP_REDIRECT_HIT="0"                               
           }                                          TCP_MISS="0"                                       
         },                                           TCP_REFRESH_MISS="0"                               
         "SSL":                                       TCP_CLIENT_REFRESH_MISS="0"                        
         {                                            TCP_DENIED="0"                                     
           "RecvSize":0,                              TCP_ERROR="0"/>                                    
           "SendSize":0                           </Traffic>                                             
         },                                       <FileSystem>                                           
         "RequestHit":                              <GetAttr                                             
         {                                            TimeRes="0"                                        
           "TCP_NONE":0,                              FileCount="0"                                      
           "TCP_HIT":0,                               DirCount="0"                                       
           "TCP_IMS_HIT":0,                           FailCount="0">0</GetAttr>                          
           "TCP_REFRESH_HIT":0,                     <Open TimeRes="0">0</Open>                           
           "TCP_REF_FAIL_HIT":0,                    <Read                                                
           "TCP_NEGATIVE_HIT":0,                      TimeRes="0"                                        
           "TCP_REDIRECT_HIT":0,                      BufferSize="0"                                     
           "TCP_MISS":0,                              BufferFilled="0">0</Read>                          
           "TCP_REFRESH_MISS":0,                    <RequestHit                                          
           "TCP_CLIENT_REFRESH_MISS":0,               TCP_NONE="0"                                       
           "TCP_DENIED":0,                            TCP_HIT="0"                                        
           "TCP_ERROR":0                              TCP_IMS_HIT="0"                                    
         },                                           TCP_REFRESH_HIT="0"                                
         "RequestHitSum":                             TCP_REF_FAIL_HIT="0"                               
         {                                            TCP_NEGATIVE_HIT="0"                               
           "TCP_NONE":0,                              TCP_REDIRECT_HIT="0"                               
           "TCP_HIT":0,                               TCP_MISS="0"                                       
           "TCP_IMS_HIT":0,                           TCP_REFRESH_MISS="0"                               
           "TCP_REFRESH_HIT":0,                       TCP_CLIENT_REFRESH_MISS="0"                        
           "TCP_REF_FAIL_HIT":0,                      TCP_DENIED="0"                                     
           "TCP_NEGATIVE_HIT":0,                      TCP_ERROR="0"/>                                    
           "TCP_REDIRECT_HIT":0,                    <RequestHitSum                                       
           "TCP_MISS":0,                              TCP_NONE="0"                                       
           "TCP_REFRESH_MISS":0,                      TCP_HIT="0"                                        
           "TCP_CLIENT_REFRESH_MISS":0,               TCP_IMS_HIT="0"                                    
           "TCP_DENIED":0,                            TCP_REFRESH_HIT="0"                                
           "TCP_ERROR":0                              TCP_REF_FAIL_HIT="0"                               
         },                                           TCP_NEGATIVE_HIT="0"                               
         "FileSystem":                                TCP_REDIRECT_HIT="0"                               
         {                                            TCP_MISS="0"                                       
           "GetAttr" :                                TCP_REFRESH_MISS="0"                               
           {                                          TCP_CLIENT_REFRESH_MISS="0"                        
             "TimeRes" : 0,                           TCP_DENIED="0"                                     
             "FileCount" : 0,                         TCP_ERROR="0"/>                                    
             "DirCount" : 0,                      </FileSystem>                                          
             "FailCount" : 0,                   </ClientTraffics>                                        
             "TotalCount" : 0
           },
           "Open" :
           {
             "TimeRes" : 0,
             "Count" : 0
           },
           "Read" :
           {
             "TimeRes" : 0,
             "BufferSize" : 0,
             "BufferFilled" : 0,
             "Count" : 0
           },
           "RequestHit":
           {
             "TCP_NONE":0,
             "TCP_HIT":0,
             "TCP_IMS_HIT":0,
             "TCP_REFRESH_HIT":0,
             "TCP_REF_FAIL_HIT":0,
             "TCP_NEGATIVE_HIT":0,
             "TCP_REDIRECT_HIT":0,
             "TCP_MISS":0,
             "TCP_REFRESH_MISS":0,
             "TCP_CLIENT_REFRESH_MISS":0,
             "TCP_DENIED":0,
             "TCP_ERROR":0
           },
           "RequestHitSum":
           {
             "TCP_NONE":0,
             "TCP_HIT":0,
             "TCP_IMS_HIT":0,
             "TCP_REFRESH_HIT":0,
             "TCP_REF_FAIL_HIT":0,
             "TCP_NEGATIVE_HIT":0,
             "TCP_REDIRECT_HIT":0,
             "TCP_MISS":0,
             "TCP_REFRESH_MISS":0,
             "TCP_CLIENT_REFRESH_MISS":0,
             "TCP_DENIED":0,
             "TCP_ERROR":0
           }
         }
       }
     ]
   }
   
-  ``Depth`` The directory depth to collect stats
-  ``Accum`` If directory stats are set, then all subdirectory stats will be accumulated in the upper directory
-  ``HttpsTraffic`` Allows redundant accumulation of HTTPS traffic as HTTP traffic
-  ``TrafficCount`` The aggregated traffic count
-  ``Traffic`` Stats per each directory. Roots(/) always have traffic.

   -  ``Path`` The service directory
   -  ``HttpReqCount(unit: Bytes)`` The total number of HTTP requests sent by clients
   -  ``HttpReqHeaderSize(unit: Bytes)`` The size of HTTP request headers sent by clients
   -  ``HttpReqBodySize(unit: Bytes)`` The size of HTTP request Body sent by clients
   -  ``HttpResHeaderSize(unit: Bytes)`` The size of HTTP response headers sent by STON
   -  ``HttpResBodySize(unit: Bytes)`` The size of HTTP response Body sent by STON
   -  ``Response`` Responses sent by STON
   
      -  ``Count`` Response counts
      -  ``Completed`` The total number of completed transferred HTTP transactions
      -  ``TimeRes`` The HTTP response time
      -  ``TimeComplete`` The completion time of the HTTP transaction
        
-  ``SSL(unit: Bytes)`` HTTPS traffic (RecvSize=received size, SendSize=transmitted size)
-  ``RequestHit`` Cache HIT result
-  ``FileSystem`` FileSystem access

   -  ``GetAttr`` The getattr function call count and response time. (FileCount: File response, DirCount: Dir response, FailCount: failure response)
   -  ``Open`` Opens the function call count and response time
   -  ``Read`` Reads the function call count and response time, requested size(BufferSize) and response size(BufferFilled)
   -  ``RequestHit`` (File I/O access) The cache HIT result


.. note::

   Items that are only provided statistics in five minute increments.
   
   -  ``HttpReqCountSum`` The total number of HTTP requests
   -  ``CountSum`` The total number of HTTP responses
   -  ``CompletedSum`` The total number of completed HTTP transactions
   -  ``RequestHitSum`` The cache HIT result
   

View
====================================

Extracts statistics from multiple virtual hosts. 
The concept came from the View that deals with multiple Tables in a Database. 
The structure is simple, as shown below. ::

   # vhosts.xml

   <Vhosts>
     <Vhost> ... </Vhost>
     <Vhost> ... </Vhost>
     ... (skip) ... 
     <View Name="SK">
       <Vhost>...</Vhost>
       <Vhost>...</Vhost>
     </View>
     <View Name="KT">
       <Vhost>...</Vhost>
       <Vhost>...</Vhost>
       <Vhost>...</Vhost>
     </View>
     <View Name="LG">
       <Vhost>...</Vhost>
       <Vhost>...</Vhost>
     </View>
   </Vhosts>
   
The View can even include a virtual host that does not exist. 
The following shows the stats provided by View. ::

-  Realtime XML/JSON
-  SNMP - cache(1.3.6.1.4.1.40001.1.4).10 ~ 12

Let's explore a scenario where View could be adopted. 
There are three administrators who are running their favorite sport communities(baseball.com, basketball.com, and football.com). ::

   # vhosts.xml

   <Vhosts>
     <Vhost Name="baseball.com"> ... </Vhost>
     <Vhost Name="basketball.com"> ... </Vhost>
     <Vhost Name="football.com"> ... </Vhost>
   </Vhosts>
   
Now, they decide to run a comprehensive sports community service. 
The sports.com domain is considered because it can include three sports in the same category. 
The development/management team has to resolve the following issues:

- Provide the service through sports.com
- Keep previous domains and services the same for previous users
- Merge development teams. Merge management teams
- Build only the first page and provide all previous services via links
- Manage without enough budget, manpower, or time
- Complete all purchase procedures

In order to reasonably meet all of the above requirements, the development team decides to specify previous domains in the first directory, as shown below. ::

   # Previous services
   http://baseball.com/standing/list.html
   http://basketball.com/stats/2014/view.html
   http://football.com/player/messi.php

   # Integrated services
   http://sports.com/baseball/standing/list.html
   http://sports.com/basketball/stats/2014/view.html
   http://sports.com/football/player/messi.php
   
This can be configured easily with the URL preprocessor. ::

   # vhosts.xml

   <Vhosts>
     <Vhost Name="baseball.com"> ... </Vhost>
     <Vhost Name="basketball.com"> ... </Vhost>
     <Vhost Name="football.com"> ... </Vhost>
     <URLRewrite>
       <Pattern>sports.com/(.*)/(.*)</Pattern>
       <Replace>#1.com/#2</Replace>
     </URLRewrite>  
   </Vhosts>
   
The newly merged management team now has to monitor not only each service but also integrated services such as traffic, sessions, and response codes. 
Most administrators are familiar with SNMP. In order to acquire an integrated index, the View can be configured as shown below.

.. figure:: img/view1.png
   :align: center

::

   # vhosts.xml

   <Vhosts>
      <Vhost Name="baseball.com"> ... </Vhost>
      <Vhost Name="basketball.com"> ... </Vhost>
      <Vhost Name="football.com"> ... </Vhost>
      <URLRewrite>
         <Pattern>sports.com/(.*)/(.*)</Pattern>
         <Replace>#1.com/#2</Replace>
      </URLRewrite>
      <View Name="sports.com">
         <Vhost>baseball.com</Vhost>
         <Vhost>basketball.com</Vhost>
         <Vhost>football.com</Vhost>
      </View>  
   </Vhosts>
   
As you can see from the above example, the combination of URL Rewrite and View is effective when merging multiple sites into one.


View Statistics
----------------------------

Provides identical stats for the virtual host, except the following different tag names. ::

   "View":                                  <View ...>        
   [                                           ...            
     { ... },                               </View>           
     { ... },                               <View> ... </View>
   ]                                        <View> ... </View>
   
   
.. _api-monitoring-vhostlist:
   
Virtual Host List Inquiry
====================================

This requests the list of virtual hosts. ::

    http://127.0.0.1:10040/monitoring/vhostslist
    
The result is returned in JSON format. ::

    {
        "version": "2.0.0",
        "method": "vhostslist",
        "status": "OK",
        "result": [ "www.example.com","www.winesoft.com", "site1.com" ] 
    }






.. _api-monitoring-fileinfo:
   
Caching Information
====================================

Monitors the status of files being cached. 
Generally, files are distinguished by URLs, but there could be multiple files when a URL has other options(e.g. Accept-Encoding). ::

    http://127.0.0.1:10040/monitoring/fileinfo?url=example.com/sample.dat
    
The result is returned in JSON format.
The following is the result of a /sample.dat file inquiry. ::

    {
        "version": "2.0.0",
        "method": "fileinfo",
        "status": "OK",
        "result":
        [ 
            {
                "URI": "/sample.dat",
                "Accept-Encoding": "N",
                "RefCount": 0,
                "Disk-Index": 0,
                "Size": 2100267,
                "FID": 24267,
                "LocalPath": "/cache1/example.com/000i/q3.bin",
                "File-Opened ": "N",
                "File-Updating": "-",
                "Downloader-Count": "0",
                "LastAccess": "[ 2012.09.03 14:29:50, -2 ]",
                "UpdateTime": "[ 2012.09.03 13:53:43, -2169 ]",
                "TTL-Left": "[ 2012.10.03 13:53:43, 2589831 ]",
                "ResponseCode": 200,
                "ContentType": "text/plain",
                "LastModifiedTime": "[ 2010.11.22 20:31:47, -56224685 ]",
                "ExpireTime": "[ 0, 0 ]",
                "CacheControl": "not-specified",
                "ETag": "502dd614:200c2b",
                "CustomTTL": 0,
                "NoMoreExist": "N",
                "LocalFileExist": "Y",
                "SmallFile": "N",
                "State": "Cached",
                "Deleted": "N",
                "AddedSize": "Y",
                "TransferEncoding": "N",
                "Compression": "-",
                "Purge": "N",
                "Ignore-IMS ": "N",
                "Redirect-Location ": "-",
                "Content-Disposition ": "-",
                "NoCache": "N"
            }
        ]
    }
    
-  ``URI`` The file URI
-  ``Accept-Encoding`` ("Y" or "N") If Accept-Encoding is supported, then "Y"
-  ``RefCount`` The file reference count
-  ``Size`` (Bytes) The file size
-  ``Disk-Index`` (starting from 0) The saved disk index
-  ``FID`` The file ID
-  ``LocalPath`` The local path
-  ``File-Opened`` ("Y" or "N") If a local file is opened, then "Y"
-  ``File-Updating`` If a file is being updated, File-Updating specifies the pointer of the updated object
-  ``Downloader-Count`` The total number of sessions that are downloading a corresponding file from the origin server
-  ``LastAccess`` (Last accessed time, last accessed time - current time) [ 2012.09.03 14:29:50, -2 ] Means that the file was accessed two seconds prior to 2012.09.03 14:29:50
-  ``UpdateTime`` (Modified time, modified time - current time) The time when the file is modified. 304 Not Modified will also update the time.
-  ``TTL-Left`` (Expiration time, expiration time - current time) Expiration time of the content. If there is TTL left, the value is positive, otherwise a negative value is returned.
-  ``ResponseCode`` The origin server response code
-  ``ContentType`` MIME Type
-  ``LastModifiedTime`` (Last Modified Time, Last Modified Time - current time) The last Modified Time sent from the origin server. If the origin server didn't send this value, 0 will be returned.
-  ``ExpireTime`` (Expire Time, Expire Time - current time) The expire Time sent from the origin server. If the origin server didn't send this value, 0 will be returned.
-  ``CacheControl`` ("no-cache" or "not-specified" or (Integer)) The cache-Control value sent from the origin server.
-  ``ETag`` An ETag created by STON
-  ``CustomTTL`` A custom TTL. If not configured, 0 will be returned.
-  ``NoMoreExist`` ("Y" or "N") If the file is pending for discard, then "Y"
-  ``LocalFileExist`` ("Y" or "N") If the file exist locally, then "Y" (Files that are not 200 OK will always return "Y")
-  ``SmallFile`` ("Y" or "N") If the file is considered as a small file, then "Y" (development purpose)
-  ``State`` ("Not Init" or "Cached" or "Error") The file status
-  ``Deleted`` ("Y" or "N") If the file is deleted, then "Y" (development purpose)
-  ``AddedSize`` ("Y" or "N") If the file size is reflected in the stats, then "Y" (development purpose)
-  ``TransferEncoding`` ("Y" or "N") If Transfer-Encoding is supported, then "Y"
-  ``Compression`` The compression method
-  ``Purge`` ("Y" or "N") If the file is purged, then "Y"
-  ``Ignore-IMS`` ("Y" or "N") When the file is being updated, if the setting does not send an If-Modified-Since header, then "Y"
-  ``Redirect-Location`` The location header value
-  ``Content-Disposition`` The Content-Disposition header value
-  ``NoCache`` ("Y" or "N") If the origin server replied no-cache, then "Y"



.. _api-monitoring-logtrace:
   
Log Trace
====================================

Subscribes log in real time. 
Access, Origin, and Monitoring logs have to specify a virtual host(vhost). ::

    http://127.0.0.1:10040/monitoring/logtrace/info
    http://127.0.0.1:10040/monitoring/logtrace/deny
    http://127.0.0.1:10040/monitoring/logtrace/sys
    http://127.0.0.1:10040/monitoring/logtrace/originerror
    http://127.0.0.1:10040/monitoring/logtrace/access?vhost=www.site1.com
    http://127.0.0.1:10040/monitoring/logtrace/origin?vhost=www.site1.com
    http://127.0.0.1:10040/monitoring/logtrace/monitoring?vhost=www.site1.com



