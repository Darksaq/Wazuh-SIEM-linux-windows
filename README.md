
    
# 1. Installing Wazuh Manager on Linux (Parrot OS)
First, we prepare the system, add the keys, and run the installation assistant, which will configure the entire stack (Manager, Indexer, Dashboard).
```sudo apt update && sudo apt full-upgrade -y```

```sudo apt update && sudo apt full-upgrade -y```

## - Adding the GPG key
```curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg```

## - Downloading and running the installation script
`curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i`

Once the installation is complete, the terminal will display the login and password.

# 2. Verifying service status
Sprawdzamy, czy wszystko działa:

`sudo systemctl status wazuh-manager`

`sudo systemctl status wazuh-indexer`

`sudo systemctl status wazuh-dashboard`

# 3. Accessing the Web Dashboard
To log in to the interface, check the machine's IP address and enter it into your browser.

## - Checking the local IP address
`ifconfig`

URL: https://<your_ip>

(Use the login and password provided in the terminal after installation).

<img width="1666" height="973" alt="1 dashboard" src="https://github.com/user-attachments/assets/f4cf6beb-9826-4c37-895f-160d9012055d" />

# 4. Registering an Agent on the Manager (Key Generation)
Before installing the agent on Windows, we must register it on the Manager system to obtain an authentication key.

`sudo /var/ossec/bin/manage_agents`

#### Steps in the menu:

1. Select **A** to add an agent.

2. Provide a name (e.g., WindowsHost).

3. Leave the IP address blank (unless you have a static assignment).

4. After creation, select **E** to extract the key.

5. Copy the generated character string.

# 5. Installing and Configuring the Agent on Windows
#### Generate the command in the Web GUI:

1. Go to **Wazuh > Agents > Deploy new agent**.

2. Fill in the fields according to your setup:
   * **Package selection:** Select the Windows icon.
   * **Wazuh server address:** Enter the local IP address of your wazuh-manager machine.
   * **Agent name:** Name it, e.g., Windows11.
   * **Group:** You can leave it as default.


<img width="1666" height="973" alt="instal 1" src="https://github.com/user-attachments/assets/c9479ed1-3922-47d3-8477-d92c3be6923f" />
<img width="1666" height="973" alt="instal 2" src="https://github.com/user-attachments/assets/f0b7b02d-f32b-4efc-ac29-3749274d083d" />



#### A PowerShell command will appear at the bottom of the page.

1. Log in to the agent machine (in this case, Win11).

2. Open **PowerShell as Administrator**.

3. Paste the command and press **Enter**.

#### The command will look something like this:

`Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.12.msi -OutFile ${env:tmp}\wazuh-agent.msi; msiexec.exe /i ${env:tmp}\wazuh-agent.msi /q WAZUH_MANAGER='1.1.1.1' WAZUH_AGENT_NAME='Windows11'`

<img width="956" height="265" alt="powershell" src="https://github.com/user-attachments/assets/c4f34280-5bd6-4129-9f1d-14ebea9bb3ef" />

## Starting the service:
Use one of the following commands (as administrator):

CMD:
`NET START WazuhSvc`

PowerShell:
`Start-Service wazuhsvc`

#### At this point, your Windows agent should appear as Active in the Dashboard.

## Configuration:
 #### 1. Adding a Custom Alert Rule for Testing

 `sudo nano /var/ossec/etc/rules/local_rules.xml`

 ##### Add a new rule that notifies about Brute-Force attack attempts on SSH services:

 `<group name="local,syslog,sshd,">
  <rule id="100001" level="12"> 
    <if_sid>18106, 5712, 5763</if_sid> 
    <description>ALARM: Wykryto atak na SSH/Windows!</description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>
</group>`

Level 12: Assigns high alarm priority.

if_sid: Links the rule to existing SSH and Windows log IDs.

MITRE T1110: Assigns the event to the "Brute Force" technique in the official attack classification.


#### 2. Global Server Configuration

sudo nano /var/ossec/etc/ossec.conf

```
<ossec_config>
  <global>
    <jsonout_output>yes</jsonout_output>
    <alerts_log>yes</alerts_log>
    <logall>yes</logall>
    <logall_json>yes</logall_json>
    <email_notification>no</email_notification>
    <agents_disconnection_time>10m</agents_disconnection_time>
    <agents_disconnection_alert_time>0</agents_disconnection_alert_time>
    <white_list>127.0.0.1</white_list>
    <white_list>8.8.8.8</white_list>
  </global>

  <alerts>
    <log_alert_level>3</log_alert_level>
    <email_alert_level>12</email_alert_level>
  </alerts>

  <logging>
    <log_format>plain</log_format>
  </logging>

  <remote>
    <connection>secure</connection>
    <port>1514</port>
    <protocol>tcp</protocol>
    <queue_size>131072</queue_size>
  </remote>

  <rootcheck>
    <disabled>no</disabled>
    <check_files>yes</check_files>
    <check_trojans>yes</check_trojans>
    <check_dev>yes</check_dev>
    <check_sys>yes</check_sys>
    <check_pids>yes</check_pids>
    <check_ports>yes</check_ports>
    <check_if>yes</check_if>
    <frequency>3600</frequency>
    <rootkit_files>etc/rootcheck/rootkit_files.txt</rootkit_files>
    <rootkit_trojans>etc/rootcheck/rootkit_trojans.txt</rootkit_trojans>
  </rootcheck>

  <wodle name="syscollector">
    <disabled>no</disabled>
    <interval>1h</interval>
    <scan_on_start>yes</scan_on_start>
    <hardware>yes</hardware>
    <os>yes</os>
    <network>yes</network>
    <packages>yes</packages>
    <ports all="yes">yes</ports>
    <processes>yes</processes>
  </wodle>

  <vulnerability-detection>
    <enabled>yes</enabled>
    <interval>5m</interval>
    <run_on_start>yes</run_on_start>
    <provider name="msu">
      <enabled>yes</enabled>
      <update_interval>1h</update_interval>
    </provider>
  </vulnerability-detection>

  <syscheck>
    <disabled>no</disabled>
    <frequency>43200</frequency>
    <scan_on_start>yes</scan_on_start>
    <alert_new_files>yes</alert_new_files>
    <auto_ignore>no</auto_ignore>
    <directories realtime="yes" check_all="yes">/etc,/usr/bin,/usr/sbin,/bin,/sbin</directories>
    <directories realtime="yes" check_all="yes">/root,/home/*/.ssh</directories>
  </syscheck>

  <active-response>
    <disabled>yes</disabled>
  </active-response>

  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/syslog</location>
  </localfile>

  <localfile>
    <log_format>apache</log_format>
    <location>/var/log/nginx/*.log</location>
  </localfile>

  <localfile>
    <log_format>apache</log_format>
    <location>/var/log/apache2/*.log</location>
  </localfile>

  <ruleset>
    <decoder_dir>ruleset/decoders</decoder_dir>
    <rule_dir>ruleset/rules</rule_dir>
    <decoder_dir>etc/decoders</decoder_dir>
    <rule_dir>etc/rules</rule_dir>
  </ruleset>

  <auth>
    <disabled>no</disabled>
    <port>1515</port>
    <ssl_verify_host>no</ssl_verify_host>
    <ssl_manager_cert>etc/sslmanager.cert</ssl_manager_cert>
    <ssl_manager_key>etc/sslmanager.key</ssl_manager_key>
  </auth>

</ossec_config>

<ossec_config>

  <indexer>
    <enabled>yes</enabled>
    <hosts>
      <host>https://0.0.0.0:9200</host>
    </hosts>
    <ssl>
      <certificate_authorities>
        <ca>/etc/filebeat/certs/root-ca.pem</ca>
      </certificate_authorities>
      <certificate>/etc/filebeat/certs/filebeat.pem</certificate>
      <key>/etc/filebeat/certs/filebeat-key.pem</key>
    </ssl>
  </indexer>

</ossec_config>
```

## Testing

#### Test if Wazuh detects SSH logins

##### You can do this manually with the command:

``ssh <nazwa_użytkownika>@<adres_agenta>``

##### Or you can use Hydra:

``hydra -l h -P /usr/share/wordlists/rockyou.txt.gz -t 64 -V -f <ip_agenta> ssh``

-t 64: To create many simultaneous connections and increase detection.

-V: To see every attempt in real-time (optional).

-f: Stops the attack after finding a valid password (optional).

<img width="1320" height="766" alt="image" src="https://github.com/user-attachments/assets/346ba497-67a3-4d01-a44f-adfa7a762c45" />

<img width="1320" height="766" alt="image" src="https://github.com/user-attachments/assets/69f2f554-d48b-4a63-9422-bbc81341fe71" />





## Lab Summary
Wazuh-manager: Parrot OS

Wazuh-agent: Windows 11
