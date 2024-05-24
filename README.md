# How To Set Up a Central Logging Ubuntu Server with Rsyslog

## Step 1: Provision 2 EC2 Instances (Client and Server)

1. Set the Security Group of the Server to allow Inbound Traffic from **TCP** on **Port 514**. Note that, **Rsyslog** can also listen for **UDP** connections on the same port.

## Step 2: Install Rsyslog on the 2 Servers
1. Run the following command to check if **Rsyslog** is installed on your servers.

```sh
which rsyslogd
```
_If it returns an output, then rsyslog is already installed on your servers_

2. If no output was returned, run the following command to install rsyslog on your servers.

```sh
sudo apt update && sudo apt install rsyslog -y
```

3. Once rsyslog installed, you need to start the service for now, enable it to auto-start at boot and check it’s status with the systemctl command.

```sh
sudo systemctl start rsyslog
sudo systemctl enable rsyslog
sudo systemctl status rsyslog
```

## Step 3: Server-Side Setup
1. Create a directory where all the client logs will reside.

```sh
sudo mkdir /var/log/remote
sudo chmod 755 /var/log/remote
sudo chown -R syslog:syslog /var/log/remote
```

2. Configure the `rsyslog.conf` file.

```sh
sudo vi /etc/rsyslog.conf
```

3. Uncomment the `tcp` modules.

```sh
module(load="imtcp")
input(type="imtcp" port="514")
```

4. Add the following lines in the `rsyslog.conf` file to send the **client** logs to the directory you created.

```sh
$template RemoteLogs,"/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteLogs
& ~
```

The above ruleset is explained below:

* **`$template`** tells rsyslog daemon to gather and write all of the received remote messages to distinct logs under **/var/log/remote**, based on the hostname (client machine name) and remote client facility (program/application) that generated the messages as defined by the settings present in the template **RemoteLogs**.

* **`*.* ?RemoteLogs`** means record messages from all facilities at all severity levels using the **RemoteLogs** template configuration.

* **`& ~`** instructs rsyslog to stop processing the messages once it is written to a file. If you don’t include **`& ~`**, messages will instead be be written to the local files. This is **optional**.

5. Save and exit the file.

6. Restart the rsyslog to apply the changes.

```sh
sudo systemctl restart rsyslog
```

## Step 4: Client-Side Setup

1. Configure the `rsyslog.conf` file.

```sh
sudo vi /etc/rsyslog.conf
```

2. To force the rsyslog daemon to act as a log client and forward all locally generated log messages to the remote rsyslog server, add this forwarding rule, at the end of the file as shown below:

##Enable sending of logs over TCP add the following line:

```sh
##Enable sending of logs over TCP add the following line:
*.* @@<public_ip_server>:514

##Set disk queue when rsyslog server will be down:
$ActionQueueFileName queue
$ActionQueueMaxDiskSpace 1g
$ActionQueueSaveOnShutdown on
$ActionQueueType LinkedList
$ActionResumeRetryCount -1
```

_Note that @@ was used because it is the TCP port we are leveraging, @ is used when leveraging UDP._

3. Save and exit the file.

4. Run the following command to populate the ubuntu log:

```sh
logger "ikenna is king"
```

5. Restart the rsyslog to apply the changes.

```sh
sudo systemctl restart rsyslog
```

## Step 5: Monitor Remote Logging from the Server

1. Run the following command to check if the `/var/log/remote` directory was populated by rsyslog client.

```sh
ls /var/log/remote
```

![list hosts](./images/1.%20list%20the%20hosts.png)
_You will see the hostnames of your client and server._

2. Run the following command to list the content of the client logs directory:

```sh
ls /var/log/remote/<client_hostname>
```

![client logs](./images/2.%20list%20the%20log%20files.png)

3. Run the following command to check the log of the client **ubuntu** log file.

```sh
sudo tail -f /var/log/remote/<client_hostname>/ubuntu.log
```
![ubuntu log](./images/3.%20logs%20of%20ubuntu.png)
_You will see the log data you logged in from the client server._


4. Run the following command to check the log of the client **systemd** log file.

```sh
sudo tail -f /var/log/remote/<client_hostname>/systemd.log
```

![systemd log](./images/3.%20logs%20of%20systemd.png)







