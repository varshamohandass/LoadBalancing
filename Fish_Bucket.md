# Fish Bucket

## What is Fish Bucket?
In Splunk, the fishbucket is an internal index (also a Sub directory) which monitors/tracks internally how far the  content of your file is indexed in splunk(specifically inputs that use the monitor input type) , from where to resume indexing.
It records checkpoint data, including the position of the last read event from a file or directory, allowing Splunk to resume reading data where it left off, even after a restart. Those two check points are known as Seek Pointers and CRCs (Cyclic Redundancy Check)
To see the contents of fishbucket, search “index=_thefishbucket” in your splunk GUI (contents can only be seen in the older versions of splunk). 

## Why Ddo we need Fish Bucket?
 This tracking prevents duplication and ensures consistent data ingestion, crucial for managing log files and other continuously updated sources. You can access the fishbucket through commands to troubleshoot or reset data inputs if needed.
 
## How does Fish Bucket Works?
Before understanding How Fish Bucket works, we need understand two terms: 
1. Cyclic Redundancy Check(CRC):
   The splunk monitoring processor selects and reads the first 256 bytes of a new file and hashes this data into a begin and end cyclic redundancy check (CRC), this functions as a fingerprint representing the file content. Splunk uses this CRC to look up an entry in a database that contains all the beginning CRCs of files it has seen before.

2. Seek Pointer/Address:
    This Adress keeps track of where Splunk left off in the file, so if Splunk is stopped and restarted, it can continue reading from the exact point it previously processed, rather than re-reading the entire file.

**Working of a Fish Bucket:**
1. When a splunk monitoring processor is instructed to read a file, it selects and reads the first 256 bytes of a new file and hashes this data which we call Cyclic Redundancy Check (CRC)
2. Then the Splunk monitoring processor is going to take this CRC value to comapare it against a database maintained by the Fish bucket (this DB contains the CRC's of all files that has been read)
3. If the CRC is not found in the Fish Bucket Database that means this file has never been read and then it is going to instruct the monitoring processor to go ahead and read into the file.
4. In addition to CRC it has Seek address (how far the monitoring processor has read into the file) and SeekCRC (Fingerprint of the location of the last byte that has been read into the file). CRC, Seek Address and SeekCRC are maintained in the Finsh bucket.
5.  Suppose if the CRC is available in the Fish Bucket Database (ie. the file has be read). The Splunk Monitoring processsor compares whether the value of Seek Address is smaller than the size of the file being monitored, that means new data came in. The Fish bucket instructs the Splunk Monitoring processsor to start monitor the file from the SeekCRC
6.  Also if the CRC is not same as the CRC for this file available in Fish Bucket Database, then the Fish bucket instructs the Splunk Monitoring processsor to start monitor the file from the beginning thinking that some changes has happened in the first 256 bytes of the file

   
## How and Where is this Fish Bucket is Stored?
```bash
$SPLUNK_HOME/var/lib/splunk/fishbucket
```
This location contains the _thefishbucket index, where Splunk records information about each monitored file, including its position, file pointer, and a unique file signature (called CRC). Splunk uses this index to track file read positions so it can resume reading from where it left off. 


References
1. https://www.youtube.com/watch?v=Zcd77x8S0Zk
   


