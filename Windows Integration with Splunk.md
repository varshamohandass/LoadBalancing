### **1. Set Up Splunk on EC2 Instance**

#### **Step 1: Launch EC2 Instance**
1. Log in to your AWS Management Console.
2. Launch a new EC2 instance with the desired configuration, such as `t2.medium`.
3. Select an Amazon Linux 2 or Ubuntu instance based on your preference.
4. Configure your instance security group to allow inbound traffic on port **Custom TCP Port from 8000-9999** - `8000` (for Splunk web UI), `8089` (for Splunk management), and `9997` (for the Universal Forwarder to send data to the Splunk indexer). 
5. Once the EC2 instance is running, SSH into it.

#### **Step 2: Install Splunk on EC2 Instance**
1. Splunk Installation
```bash 
https://github.com/varshamohandass/Splunk-System-Admin-Training/blob/main/Splunk_enterprise_instalation_linux_9.2.2.md
```
2. Open the Splunk UI in a browser using your EC2 instance's public IP address:
   ```
   http://<EC2_PUBLIC_IP>:8000
   ```

3. Log in with the credentials you set up earlier.
4. Goto Settings--> Forwarding and Receiving --> Configure Receiving --> Enter **9997** port number --> Click ok

#### **Step 4: Create an Index in Splunk**
1. Once logged into the Splunk UI, navigate to **Settings** > **Indexes**.
2. Click **New Index** and create a new index called `windows_idx` (or any name you prefer).
3. Ensure to configure the retention and storage settings according to your environment's needs.

---

### **2. Install and Configure the Splunk Universal Forwarder on Windows Machine**

#### **Step 1: Download and Install the Universal Forwarder**
1. Download the **Splunk Universal Forwarder** for Windows from the [Splunk Downloads](https://www.splunk.com/en_us/download/universal-forwarder.html).
2. Install the Universal Forwarder on the Windows machine by following the installation wizard.
   - Ensure to select the option to install the Universal Forwarder as a service.
   - Enter Splunk Indexer's Public IP when it prompts

#### **Step 2: Configure Forwarding to the Splunk Indexer**
1. Open the **Splunk Universal Forwarder** and navigate to the configuration directory (usually under `C:\Program Files\SplunkUniversalForwarder\etc\system\local`).
2. Open (or create if missing) the `outputs.conf` file and configure it to forward data to your Splunk indexer (EC2 instance):

   Example configuration:
   ```ini
   [tcpout]
   defaultGroup = default-autolb-group

   [tcpout:default-autolb-group]
   server = <EC2_PUBLIC_IP>:9997

   ```

   **Replace `<EC2_PUBLIC_IP>`** with the public IP address of your Splunk indexer EC2 instance.

#### **Step 3: Install and Configure Splunk Add-on for Windows**
1. Download the **Splunk Add-on for Windows** from the [Splunkbase](https://splunkbase.splunk.com/app/742/).
2. Extract the add-on files to your Universal Forwarder installation directory under:
   ```text
   C:\Program Files\SplunkUniversalForwarder\etc\apps\
   ```

3. Navigate to the **Splunk Add-on for Windows** directory (`Splunk-TA-Windows`), then go to `local` and copy the `inputs.conf` file from the `default` folder to the `local` folder.

#### **Step 4: Configure the inputs.conf File**
1. Open `inputs.conf` under `local` for editing.
2. Add `index=windows_idx` under each stanza you want to enable, and ensure `disabled = 0` for the ones you want to enable.

   Example configuration:
   ```ini
   [WinEventLog://Application]
   index = windows_idx
   disabled = 0

   [WinEventLog://Security]
   index = windows_idx
   disabled = 0

   [WinEventLog://System]
   index = windows_idx
   disabled = 0

   [perfmon://CPU]
   index = windows_idx
   disabled = 0
   ```

   For any stanzas you do not need, you can set `disabled = 1` to prevent the data from being collected.

3. Save and close the file.

#### **Step 5: Restart the Universal Forwarder Service**
1. Open the **Services** app in Windows and locate **Splunk Universal Forwarder**.
2. Right-click and select **Restart** to apply the changes.

---

### **3. Verify the Data in Splunk Indexer**

#### **Step 1: Search for Data**
1. Go to the **Splunk UI** on your EC2 instance.
2. In the **Search & Reporting** app, use the following search query to check if the data is being received:
   ```spl
   index=windows_idx
   ```
3. You should now start seeing the Windows logs being forwarded from the Universal Forwarder.

#### **Step 2: Troubleshooting (If Necessary)**
- Ensure the Universal Forwarder’s **forwarding configuration** (in `outputs.conf`) points to the correct IP and port.
- Ensure that **firewall** or security group settings are open on the necessary ports (`9997` for Splunk forwarding) between the Windows machine and the EC2 instance.
- Check the **Splunk Universal Forwarder logs** on the Windows machine (`$SPLUNK_HOME\var\log\splunk\splunkd.log`) for any errors related to forwarding.
- In the Splunk UI, use the **Forwarding status** page to verify if data is being forwarded correctly.

---

### **4. Final Steps and Best Practices**

- **Monitor and Optimize**: Monitor the performance of the EC2 instance and Universal Forwarder. You might need to adjust the configuration based on log volume and data retention policies.
- **Data Retention**: Consider configuring retention settings for the `windows_idx` to prevent the index from growing too large. This can be done under **Settings** > **Indexes** in the Splunk UI.
- **Security Best Practices**: Ensure that your EC2 instance and Universal Forwarder are properly secured with appropriate firewall settings and that access to Splunk’s web interface is restricted to authorized users.

By following these detailed steps, you will have a fully functional Splunk deployment on AWS EC2, collecting and indexing logs from your Windows machine using the Universal Forwarder.


**Usecase 1:** Find the network ports used by each app

```bash
index=windows_idx  source="C:\\Program Files\\SplunkUniversalForwarder\\etc\\apps\\Splunk_TA_windows\\bin\\win_listening_ports.bat" sourcetype="Script:ListeningPorts" | table appname transport dest_ip dest_port 
```

**Usecase 2:** Plot the packets sent and received

```bash
index=windows_idx  source="PerfmonMk:Network" sourcetype="PerfmonMk:Network" | timechart span=10m sum(Packets_Received/sec) as Packets_Received sum(Packets_Sent/sec) as Packets_Sent
```

**Usecase 3:** List all the applications installed in your Windows Machine

```bash
index=windows_idx  source="C:\\Program Files\\SplunkUniversalForwarder\\etc\\apps\\Splunk_TA_windows\\bin\\win_installed_apps.bat" sourcetype="Script:InstalledApps"| table DisplayName
```
