# Load Balancing in Splunk

With [**load balancing**](https://docs.splunk.com/Splexicon:Loadbalancing "Splexicon:Loadbalancing"), a forwarder distributes data across several receiving instances. Each receiver gets a portion of the total data, and together the receivers hold all the data. To access the full set of forwarded data, you need to set up distributed searching across all the receivers.

Load balancing enables horizontal scaling for improved performance. In addition, its automatic switchover capability ensures resiliency in the face of machine outages. If a host goes down, the forwarder sends data to the next available receiver.

Load balancing can also be of use when you get data from network devices like routers. To handle syslog and other data generated across TCP port 514, a single universal forwarder can monitor port 514 and distribute the incoming data across several indexers.

**Note:**  You should not use an external load balancer to implement load balancing between forwarders and receivers. This practice does not generate the results you would expect. Use the load balancing capability that comes with the forwarder.

## Options for configuring receiving targets for load balancing

### Choose a load balancing method

You can choose how you want the forwarders to load balance between the indexers in a load balancing list.

-   **By time**. The default method for load balancing is how frequently the forwarders change indexers in the load balanced list. The  `autoLBFrequency`  setting in outputs.conf controls how often forwarders switch between indexers. The default frequency is every 30 seconds, but you can set it higher or lower.
    

-   **By volume**. Another option is to set how much data a forwarder sends to an indexer before it switches between indexers in a load-balanced list. The  `autoLBVolume`  setting in  `outputs.conf`  controls the amount of data that a forwarder sends to a receiving indexer before it changes to another one. By default, this setting is not active (0 bytes.) If you set it to anything other than 0, then the forwarder will change indexers based on the amount of data that it has sent.
    

  
If you enable both settings, then the forwarder chooses an indexer based on the following logic:

-   If the forwarder has sent more than  `autoLBVolume`  bytes of data to an indexer, it changes indexers regardless of whether or not  `autoLBFrequency`  have passed since the last change to a receiving indexer.
    
-   If the forwarder has not sent more than  `autoLBVolume`  bytes of data before  `autoLBFrequency`  seconds have elapsed, then it changes indexers after that time has passed.

### Specify a static list target

1.  On a forwarder that you want to set a static list target, edit  `$SPLUNK_HOME/etc/system/local/outputs.conf`. You might have to create this file beforehand.
    
2.  In the  `outputs.conf`  file, specify each of the receivers in the target group  `[tcpout]`  stanza.
    
3.  Save the  `outputs.conf`  file.
    
4.  Restart the forwarder. The forwarder sends data to the static list targets.
    

### Examples of a static load-balancing configuration file

In the following example, the target group consists of three receivers, specified by IP address and receiving port number. The universal forwarder balances load between the three receivers. If one receiver goes down, the forwarder switches to another one on the list.

[tcpout: my_LB_indexers]
server=10.10.10.1:9997,10.10.10.2:9996,10.10.10.3:9995

  
In the following example, the target group consists of four receivers, specified by IP address and receiving port number. The universal forwarder has been configured to send a specific amount of data (in this case, 1MB) to a receiver before it switches to another receiver in the list.

[tcpout: my_LB_indexers]
server=10.10.10.1:9997,10.10.10.2:9996,10.10.10.3:9995,10.10.10.4:9994
autoLBVolume=1048576

  
In the following example, the target group consists of three receivers. one with a DNS hostname and two with IP addresses. The universal forwarder has been configured to send data to one indexer for 3 minutes (180 seconds) before switching to another receiver.

[tcpout:My_LB_Indexers]
server=192.168.1.15:9997,192.168.1.179:9997,server1.mktg.example.com:9997
autoLBFrequency=180

###How to check if Load Balancing Configuration is working

index="_internal" host="LAPTOP-OPDF715T" AutoLoadBalancedConnectionStrategy use this query in indexer
