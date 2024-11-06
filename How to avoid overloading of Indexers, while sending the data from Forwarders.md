# Load Balancing in Splunk

Splunk's **load balancing** allows forwarders to distribute data across multiple receiving instances (indexers). With this method, each indexer receives a portion of the data, so when you need the complete data set, setting up **distributed searching** across all indexers is essential.

Load balancing enables **horizontal scaling** for improved performance and **automatic failover** for resilience. If an indexer becomes unavailable, the forwarder redirects data to the next available indexer, ensuring continued data flow. This feature is also particularly useful for managing high-velocity data sources, such as syslog data from routers. For instance, a universal forwarder can collect data on TCP port 514 and distribute it to several indexers.

> **Note**: Avoid using an external load balancer to handle load balancing between forwarders and receivers, as it can lead to unexpected results. Instead, leverage the built-in load balancing capabilities of the Splunk forwarder.

---

## Configuring Load Balancing in Splunk

Splunk offers options to configure load balancing based on time, volume, or a combination of both. 

### 1. **Choose a Load Balancing Method**

- **By Time**: The default load-balancing method is based on time intervals. The `autoLBFrequency` setting in `outputs.conf` specifies the interval (in seconds) after which the forwarder switches to a different indexer. The default value is 30 seconds, but you can customize it based on your needs.

  ```plaintext
  autoLBFrequency=30
  ```

- **By Volume**: You can also specify the volume of data (in bytes) a forwarder sends to an indexer before switching to another. This is controlled by the `autoLBVolume` setting. By default, it’s set to `0` (inactive). Setting a specific value enables switching based on the amount of data sent.

  ```plaintext
  autoLBVolume=1048576  # Example: 1 MB
  ```

- **Combined Logic**: If both settings are enabled, the forwarder’s load balancing operates as follows:
  - **Volume-based Switching**: If the forwarder has sent data exceeding `autoLBVolume` to an indexer, it will switch immediately to the next indexer.
  - **Time-based Switching**: If the specified volume is not met, the forwarder will switch based on the interval specified in `autoLBFrequency`.

---

### 2. **Configuring Static Target Lists**

You can specify a static list of target indexers in `outputs.conf`, allowing the forwarder to balance data across predefined IP addresses and ports.

#### Steps:
1. Open `$SPLUNK_HOME/etc/system/local/outputs.conf` on the forwarder.
2. Define the target receivers in the `[tcpout]` stanza.
3. Save the file and restart the forwarder.

#### Example Configurations

1. **Basic Load Balancing Across Indexers**
   - In this setup, the forwarder balances load between three indexers.
   
   ```plaintext
   [tcpout:my_LB_indexers]
   server=10.10.10.1:9997,10.10.10.2:9996,10.10.10.3:9995
   ```

2. **Volume-based Load Balancing**
   - Here, the forwarder switches after sending 1 MB of data to each indexer.
   
   ```plaintext
   [tcpout:my_LB_indexers]
   server=10.10.10.1:9997,10.10.10.2:9996,10.10.10.3:9995,10.10.10.4:9994
   autoLBVolume=1048576  # 1 MB
   ```

3. **Time-based Load Balancing with Hostnames and IPs**
   - The forwarder switches every 3 minutes (180 seconds) between the listed indexers.

   ```plaintext
   [tcpout:My_LB_Indexers]
   server=192.168.1.15:9997,192.168.1.179:9997,server1.mktg.example.com:9997
   autoLBFrequency=180
   ```

---

