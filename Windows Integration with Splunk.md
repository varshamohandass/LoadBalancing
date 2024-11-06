Usecase 1: Find the network ports used by each app

```bash
index=windows_idx  source="C:\\Program Files\\SplunkUniversalForwarder\\etc\\apps\\Splunk_TA_windows\\bin\\win_listening_ports.bat" sourcetype="Script:ListeningPorts" | table appname transport dest_ip dest_port 
```

Usecase 2: Plot the packets sent and received

```bash
index=windows_idx  source="PerfmonMk:Network" sourcetype="PerfmonMk:Network" | timechart span=10m sum(Packets_Received/sec) as Packets_Received sum(Packets_Sent/sec) as Packets_Sent
```

Usecase 3: List all the applications installed in your Windows Machine

```bash
index=windows_idx  source="C:\\Program Files\\SplunkUniversalForwarder\\etc\\apps\\Splunk_TA_windows\\bin\\win_installed_apps.bat" sourcetype="Script:InstalledApps"| table DisplayName
```
