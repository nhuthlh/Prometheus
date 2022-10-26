# Grafana Dashboard for Prometheus
This lab build a Grafana Dashboard to visualize Prometheus metric data monitoring a Web server.

The Dashboard will show the Web server status panel, CPU Usage Graph panel, Memory Usage Graph panel.

The lab consists of 3 EC2s running on Ubuntu:
1. Web server
2. Prometheus server
3. Grafana server

# 1. Install the Prometheus server

Create the Prometheus system user and group
```
ubuntu@ip-172-31-82-218:~$ sudo useradd -M -r -s /bin/false prometheus
```
Create prometheus directories
```
ubuntu@ip-172-31-82-218:~$ sudo mkdir /etc/prometheus /var/lib/prometheus 
```

Download prometheus binaries
```
ubuntu@ip-172-31-82-218:~$ wget https://github.com/prometheus/prometheus/releases/download/v2.39.1/prometheus-2.39.1.linux-amd64.tar.gz
--2022-10-22 03:24:40--  https://github.com/prometheus/prometheus/releases/download/v2.39.1/prometheus-2.39.1.linux-amd64.tar.gz
Resolving github.com (github.com)... 140.82.114.3
Connecting to github.com (github.com)|140.82.114.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/6838921/81e7d819-6f49-4463-8ae6-caab757e9ced?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20221022%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20221022T032440Z&X-Amz-Expires=300&X-Amz-Signature=578b6c60efc78a2897f03012024523f85771b934a9bddf377f33f526d296446c&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=6838921&response-content-disposition=attachment%3B%20filename%3Dprometheus-2.39.1.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2022-10-22 03:24:40--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/6838921/81e7d819-6f49-4463-8ae6-caab757e9ced?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20221022%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20221022T032440Z&X-Amz-Expires=300&X-Amz-Signature=578b6c60efc78a2897f03012024523f85771b934a9bddf377f33f526d296446c&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=6838921&response-content-disposition=attachment%3B%20filename%3Dprometheus-2.39.1.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.111.133, 185.199.108.133, 185.199.109.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 86239200 (82M) [application/octet-stream]
Saving to: ‘prometheus-2.39.1.linux-amd64.tar.gz’

prometheus-2.39.1.linux-amd64.tar.gz                      100%[====================================================================================================================================>]  82.24M   193MB/s    in 0.4s    

2022-10-22 03:24:41 (193 MB/s) - ‘prometheus-2.39.1.linux-amd64.tar.gz’ saved [86239200/86239200]

ubuntu@ip-172-31-82-218:~$ ls
prometheus-2.39.1.linux-amd64.tar.gz
```

Extract the binaries
```
ubuntu@ip-172-31-82-218:~$ tar -xzf prometheus-2.39.1.linux-amd64.tar.gz 
ubuntu@ip-172-31-82-218:~$ ls
prometheus-2.39.1.linux-amd64  prometheus-2.39.1.linux-amd64.tar.gz
```
Copy the files/folders from the downloaded prometheus binaries to appropriate locations and change the ownership to prometheus user
```
ubuntu@ip-172-31-82-218:~$ cd prometheus-2.39.1.linux-amd64/
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ ls
LICENSE  NOTICE  console_libraries  consoles  prometheus  prometheus.yml  promtool
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo cp prometheus /usr/local/bin/
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo cp promtool /usr/local/bin/
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ 
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ 
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo cp -r consoles/ console_libraries/ /etc/prometheus/
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo cp prometheus.yml /etc/prometheus/prometheus.yml
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo chown -R prometheus:prometheus /etc/prometheus 
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo chown prometheus:prometheus /var/lib/prometheus 
```
Try running prometheus from the command line to check if everything is setup correctly
```
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ prometheus --config.file=/etc/prometheus/prometheus.yml 
ts=2022-10-22T03:39:25.784Z caller=main.go:499 level=info msg="No time or size retention was set so using the default time retention" duration=15d
ts=2022-10-22T03:39:25.784Z caller=main.go:543 level=info msg="Starting Prometheus Server" mode=server version="(version=2.39.1, branch=HEAD, revision=dcd6af9e0d56165c6f5c64ebbc1fae798d24933a)"
ts=2022-10-22T03:39:25.784Z caller=main.go:548 level=info build_context="(go=go1.19.2, user=root@273d60c69592, date=20221007-15:57:09)"
ts=2022-10-22T03:39:25.784Z caller=main.go:549 level=info host_details="(Linux 5.15.0-1019-aws #23-Ubuntu SMP Wed Aug 17 18:33:13 UTC 2022 x86_64 ip-172-31-82-218 (none))"
ts=2022-10-22T03:39:25.784Z caller=main.go:550 level=info fd_limits="(soft=1048576, hard=1048576)"
ts=2022-10-22T03:39:25.785Z caller=main.go:551 level=info vm_limits="(soft=unlimited, hard=unlimited)"
ts=2022-10-22T03:39:25.786Z caller=web.go:559 level=info component=web msg="Start listening for connections" address=0.0.0.0:9090
ts=2022-10-22T03:39:25.787Z caller=main.go:980 level=info msg="Starting TSDB ..."
ts=2022-10-22T03:39:25.791Z caller=head.go:551 level=info component=tsdb msg="Replaying on-disk memory mappable chunks if any"
ts=2022-10-22T03:39:25.791Z caller=head.go:595 level=info component=tsdb msg="On-disk memory mappable chunks replay completed" duration=3.946µs
ts=2022-10-22T03:39:25.791Z caller=head.go:601 level=info component=tsdb msg="Replaying WAL, this may take a while"
ts=2022-10-22T03:39:25.794Z caller=tls_config.go:195 level=info component=web msg="TLS is disabled." http2=false
ts=2022-10-22T03:39:25.794Z caller=head.go:672 level=info component=tsdb msg="WAL segment loaded" segment=0 maxSegment=0
ts=2022-10-22T03:39:25.794Z caller=head.go:709 level=info component=tsdb msg="WAL replay completed" checkpoint_replay_duration=41.436µs wal_replay_duration=3.005176ms wbl_replay_duration=729ns total_replay_duration=3.231059ms
ts=2022-10-22T03:39:25.797Z caller=main.go:1001 level=info fs_type=EXT4_SUPER_MAGIC
ts=2022-10-22T03:39:25.797Z caller=main.go:1004 level=info msg="TSDB started"
ts=2022-10-22T03:39:25.797Z caller=main.go:1184 level=info msg="Loading configuration file" filename=/etc/prometheus/prometheus.yml
ts=2022-10-22T03:39:25.804Z caller=main.go:1221 level=info msg="Completed loading of configuration file" filename=/etc/prometheus/prometheus.yml totalDuration=6.594918ms db_storage=1.6µs remote_storage=2.073µs web_handler=910ns query_engine=1.223µs scrape=6.100788ms scrape_sd=27.268µs notify=31.275µs notify_sd=11.404µs rules=1.883µs tracing=6.609µs
ts=2022-10-22T03:39:25.804Z caller=main.go:965 level=info msg="Server is ready to receive web requests."
ts=2022-10-22T03:39:25.804Z caller=manager.go:943 level=info component="rule manager" msg="Starting rule manager..."
```
Setup prometheus as a systemd service

```
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo tee /etc/systemd/system/prometheus.service<<EOF
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```
Start the prometheus service
```
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo systemctl daemon-reload 
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo systemctl start prometheus 
ubuntu@ip-172-31-82-218:~/prometheus-2.39.1.linux-amd64$ sudo systemctl status prometheus.service
● prometheus.service - Prometheus
     Loaded: loaded (/etc/systemd/system/prometheus.service; disabled; vendor preset: enabled)
     Active: active (running) since Sat 2022-10-22 04:09:06 UTC; 5s ago
       Docs: https://prometheus.io/docs/introduction/overview/
   Main PID: 2265 (prometheus)
      Tasks: 6 (limit: 1143)
     Memory: 16.7M
        CPU: 65ms
     CGroup: /system.slice/prometheus.service
             └─2265 /usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_li>

Oct 22 04:09:06 ip-172-31-82-218 prometheus[2265]: ts=2022-10-22T04:09:06.828Z caller=head.go:601 level=info component=tsdb msg="Replaying WAL, this may take a while"
Oct 22 04:09:06 ip-172-31-82-218 prometheus[2265]: ts=2022-10-22T04:09:06.830Z caller=tls_config.go:195 level=info component=web msg="TLS is disabled." http2=false
Oct 22 04:09:06 ip-172-31-82-218 prometheus[2265]: ts=2022-10-22T04:09:06.831Z caller=head.go:672 level=info component=tsdb msg="WAL segment loaded" segment=0 maxSegment=0
Oct 22 04:09:06 ip-172-31-82-218 prometheus[2265]: ts=2022-10-22T04:09:06.831Z caller=head.go:709 level=info component=tsdb msg="WAL replay completed" checkpoint_replay_duration=34.848µs wal_replay_duration=3.130541ms wbl_replay_d>
Oct 22 04:09:06 ip-172-31-82-218 prometheus[2265]: ts=2022-10-22T04:09:06.833Z caller=main.go:1001 level=info fs_type=EXT4_SUPER_MAGIC
Oct 22 04:09:06 ip-172-31-82-218 prometheus[2265]: ts=2022-10-22T04:09:06.834Z caller=main.go:1004 level=info msg="TSDB started"
Oct 22 04:09:06 ip-172-31-82-218 prometheus[2265]: ts=2022-10-22T04:09:06.834Z caller=main.go:1184 level=info msg="Loading configuration file" filename=/etc/prometheus/prometheus.yml
Oct 22 04:09:06 ip-172-31-82-218 prometheus[2265]: ts=2022-10-22T04:09:06.839Z caller=main.go:1221 level=info msg="Completed loading of configuration file" filename=/etc/prometheus/prometheus.yml totalDuration=5.038594ms db_storag>
Oct 22 04:09:06 ip-172-31-82-218 prometheus[2265]: ts=2022-10-22T04:09:06.839Z caller=main.go:965 level=info msg="Server is ready to receive web requests."
Oct 22 04:09:06 ip-172-31-82-218 prometheus[2265]: ts=2022-10-22T04:09:06.839Z caller=manager.go:943 level=info component="rule manager" msg="Starting rule manager..."
 ^X
[1]+  Stopped                 sudo systemctl status prometheus.service
```

Check if prometheus is working properly
```
ubuntu@ip-172-31-82-218:~$ curl localhost:9090
<a href="/graph">Found</a>.

ubuntu@ip-172-31-82-218:~$ 

```

Test accessing Prometheus from web brower

![image](https://user-images.githubusercontent.com/67490369/197318525-f930eae7-549d-4b15-8187-e055a1fa4b51.png)


# 2. Install Node Exporter on Web server and collect Web server metrics with prometheus

Create the node_exporter system user and group
```
ubuntu@ip-172-31-89-170:~$ sudo useradd -M -r -s /bin/false node_exporter 
```

Download the node_exporter binaries

```
ubuntu@ip-172-31-89-170:~$ wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
--2022-10-22 04:54:08--  https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
Resolving github.com (github.com)... 140.82.114.4
Connecting to github.com (github.com)|140.82.114.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/9524057/5fdacb9b-a17a-4b4a-b2f0-f2946771d4ca?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20221022%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20221022T045408Z&X-Amz-Expires=300&X-Amz-Signature=e4f010ed710c10f48b76126980fa260b9792d495e90d38f694c227e41c8daf85&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=9524057&response-content-disposition=attachment%3B%20filename%3Dnode_exporter-1.4.0.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2022-10-22 04:54:08--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/9524057/5fdacb9b-a17a-4b4a-b2f0-f2946771d4ca?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20221022%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20221022T045408Z&X-Amz-Expires=300&X-Amz-Signature=e4f010ed710c10f48b76126980fa260b9792d495e90d38f694c227e41c8daf85&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=9524057&response-content-disposition=attachment%3B%20filename%3Dnode_exporter-1.4.0.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.110.133, 185.199.111.133, 185.199.108.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10111972 (9.6M) [application/octet-stream]
Saving to: ‘node_exporter-1.4.0.linux-amd64.tar.gz’

node_exporter-1.4.0.linux-amd64.tar.gz                    100%[====================================================================================================================================>]   9.64M  --.-KB/s    in 0.05s   

2022-10-22 04:54:08 (202 MB/s) - ‘node_exporter-1.4.0.linux-amd64.tar.gz’ saved [10111972/10111972]
```

Extract the binaries

```
ubuntu@ip-172-31-89-170:~$ tar xzf node_exporter-1.4.0.linux-amd64.tar.gz 
```

Copy the file to the correct location and change the ownership

```
ubuntu@ip-172-31-89-170:~$ sudo cp node_exporter-1.4.0.linux-amd64/node_exporter /usr/local/bin/
ubuntu@ip-172-31-89-170:~$ 
ubuntu@ip-172-31-89-170:~$ sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter 
```

Set up node_exporter as a systemd service

```
ubuntu@ip-172-31-89-170:~$ sudo tee /etc/systemd/system/node_exporter.service <<EOF
> [Unit] 
Description=Prometheus Node Exporter 
Wants=network-online.target 
After=network-online.target 

[Service] 
User=node_exporter 
Group=node_exporter 
Type=simple 
ExecStart=/usr/local/bin/node_exporter 

[Install] 
WantedBy=multi-user.target
EOF
[Unit] 
Description=Prometheus Node Exporter 
Wants=network-online.target 
After=network-online.target 

[Service] 
User=node_exporter 
Group=node_exporter 
Type=simple 
ExecStart=/usr/local/bin/node_exporter 

[Install] 
WantedBy=multi-user.target
ubuntu@ip-172-31-89-170:~$ 
```

Start the service

```
ubuntu@ip-172-31-89-170:~$ sudo systemctl daemon-reload
ubuntu@ip-172-31-89-170:~$ sudo systemctl start node_exporter
ubuntu@ip-172-31-89-170:~$ sudo systemctl status node_exporter
● node_exporter.service - Prometheus Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: enabled)
     Active: active (running) since Sat 2022-10-22 04:56:38 UTC; 9s ago
   Main PID: 1758 (node_exporter)
      Tasks: 3 (limit: 1143)
     Memory: 2.1M
        CPU: 9ms
     CGroup: /system.slice/node_exporter.service
             └─1758 /usr/local/bin/node_exporter

Oct 22 04:56:38 ip-172-31-89-170 node_exporter[1758]: ts=2022-10-22T04:56:38.200Z caller=node_exporter.go:115 level=info collector=thermal_zone
Oct 22 04:56:38 ip-172-31-89-170 node_exporter[1758]: ts=2022-10-22T04:56:38.200Z caller=node_exporter.go:115 level=info collector=time
Oct 22 04:56:38 ip-172-31-89-170 node_exporter[1758]: ts=2022-10-22T04:56:38.200Z caller=node_exporter.go:115 level=info collector=timex
Oct 22 04:56:38 ip-172-31-89-170 node_exporter[1758]: ts=2022-10-22T04:56:38.200Z caller=node_exporter.go:115 level=info collector=udp_queues
Oct 22 04:56:38 ip-172-31-89-170 node_exporter[1758]: ts=2022-10-22T04:56:38.200Z caller=node_exporter.go:115 level=info collector=uname
Oct 22 04:56:38 ip-172-31-89-170 node_exporter[1758]: ts=2022-10-22T04:56:38.201Z caller=node_exporter.go:115 level=info collector=vmstat
Oct 22 04:56:38 ip-172-31-89-170 node_exporter[1758]: ts=2022-10-22T04:56:38.201Z caller=node_exporter.go:115 level=info collector=xfs
Oct 22 04:56:38 ip-172-31-89-170 node_exporter[1758]: ts=2022-10-22T04:56:38.201Z caller=node_exporter.go:115 level=info collector=zfs
Oct 22 04:56:38 ip-172-31-89-170 node_exporter[1758]: ts=2022-10-22T04:56:38.201Z caller=node_exporter.go:199 level=info msg="Listening on" address=:9100
Oct 22 04:56:38 ip-172-31-89-170 node_exporter[1758]: ts=2022-10-22T04:56:38.201Z caller=tls_config.go:195 level=info msg="TLS is disabled." http2=false
ubuntu@ip-172-31-89-170:~$ 
  ```

Test if node_exporter is working properly

```
ubuntu@ip-172-31-89-170:~$ curl localhost:9100/metrics
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 6
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.19.1"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 849960
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 849960
```

Go to the Prometheus server and change the configuration file to add a job to scrape the metrics from the Web server
  
```
ubuntu@ip-172-31-82-218:~$ sudo vi /etc/prometheus/prometheus.yml 
ubuntu@ip-172-31-82-218:~$ 
```

```
  # my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "My Web Server"
    
    static_configs:
      - targets: ["172.31.89.170:9100"]
~                                                                                                                                                                                                                                      
~                                           
```
Restart the prometheus service
```
ubuntu@ip-172-31-82-218:~$ sudo systemctl restart prometheus
ubuntu@ip-172-31-82-218:~$ 
  ```
  
Go to the web brower to verify the Prometheus can get metrics from the Web server by querring: node_filesystem_avail_bytes{job="LimeDrop Web Server"} 
  
  ![image](https://user-images.githubusercontent.com/67490369/197321022-cf282506-6d1a-4844-83d7-77feab993cdf.png)




# Installing Grafana server

```
ubuntu@ip-172-31-91-177:~$ sudo apt-get update -y
Hit:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates InRelease [114 kB]
Get:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports InRelease [99.8 kB]
Get:4 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
Get:5 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 Packages [14.1 MB]
Get:6 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy/universe Translation-en [5652 kB]
Get:7 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy/universe amd64 c-n-f Metadata [286 kB]
Get:8 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy/multiverse amd64 Packages [217 kB]
Get:9 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy/multiverse Translation-en [112 kB]
Get:10 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy/multiverse amd64 c-n-f Metadata [8372 B]
Get:11 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages [662 kB]
Get:12 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/main Translation-en [152 kB]
Get:13 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/main amd64 c-n-f Metadata [9404 B]
Get:14 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/restricted amd64 Packages [399 kB]
Get:15 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/restricted Translation-en [61.3 kB]
Get:16 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/restricted amd64 c-n-f Metadata [532 B]
Get:17 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/universe amd64 Packages [435 kB]
Get:18 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/universe Translation-en [110 kB]
Get:19 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/universe amd64 c-n-f Metadata [4404 B]
Get:20 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/multiverse amd64 Packages [7220 B]
Get:21 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/multiverse Translation-en [2360 B]
Get:22 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/multiverse amd64 c-n-f Metadata [420 B]
Get:23 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports/main amd64 Packages [3008 B]
Get:24 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports/main Translation-en [1432 B]
Get:25 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports/main amd64 c-n-f Metadata [272 B]
Get:26 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports/restricted amd64 c-n-f Metadata [116 B]
Get:27 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports/universe amd64 Packages [6752 B]
Get:28 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports/universe Translation-en [9240 B]
Get:29 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports/universe amd64 c-n-f Metadata [352 B]
Get:30 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports/multiverse amd64 c-n-f Metadata [116 B]
Get:31 http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages [435 kB]
Get:32 http://security.ubuntu.com/ubuntu jammy-security/main Translation-en [96.5 kB]
Get:33 http://security.ubuntu.com/ubuntu jammy-security/restricted amd64 Packages [363 kB]
Get:34 http://security.ubuntu.com/ubuntu jammy-security/restricted Translation-en [55.8 kB]
Get:35 http://security.ubuntu.com/ubuntu jammy-security/universe amd64 Packages [307 kB]
Get:36 http://security.ubuntu.com/ubuntu jammy-security/universe Translation-en [68.7 kB]
Get:37 http://security.ubuntu.com/ubuntu jammy-security/universe amd64 c-n-f Metadata [2408 B]
Get:38 http://security.ubuntu.com/ubuntu jammy-security/multiverse amd64 Packages [4192 B]
Get:39 http://security.ubuntu.com/ubuntu jammy-security/multiverse Translation-en [900 B]
Get:40 http://security.ubuntu.com/ubuntu jammy-security/multiverse amd64 c-n-f Metadata [228 B]
Fetched 23.9 MB in 4s (6330 kB/s)                         
Reading package lists... Done
```

Install the necessary packages

```
ubuntu@ip-172-31-91-177:~$ sudo apt-get install -y apt-transport-https software-properties-common wget 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
wget is already the newest version (1.21.2-2ubuntu1).
wget set to manually installed.
software-properties-common is already the newest version (0.99.22.3).
software-properties-common set to manually installed.
The following NEW packages will be installed:
  apt-transport-https
0 upgraded, 1 newly installed, 0 to remove and 55 not upgraded.
Need to get 1506 B of archives.
After this operation, 169 kB of additional disk space will be used.
Get:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates/universe amd64 apt-transport-https all 2.4.8 [1506 B]
Fetched 1506 B in 0s (102 kB/s)         
Selecting previously unselected package apt-transport-https.
(Reading database ... 63663 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_2.4.8_all.deb ...
Unpacking apt-transport-https (2.4.8) ...
Setting up apt-transport-https (2.4.8) ...
Scanning processes...                                                                                                                                                                                                                  
Scanning linux images...                                                                                                                                                                                                               

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
ubuntu@ip-172-31-91-177:~$ 
```

Download and add the Grafana GPG key
```
ubuntu@ip-172-31-91-177:~$ wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add - 
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
OK
ubuntu@ip-172-31-91-177:~$ 
```

Add the Grafana repo to APT

```
ubuntu@ip-172-31-91-177:~$ sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main" 
Repository: 'deb https://packages.grafana.com/oss/deb stable main'
Description:
Archive for codename: stable components: main
More info: https://packages.grafana.com/oss/deb
Adding repository.
Press [ENTER] to continue or Ctrl-c to cancel.
Adding deb entry to /etc/apt/sources.list.d/archive_uri-https_packages_grafana_com_oss_deb-jammy.list
Adding disabled deb-src entry to /etc/apt/sources.list.d/archive_uri-https_packages_grafana_com_oss_deb-jammy.list
Hit:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates InRelease [114 kB]                      
Get:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports InRelease [99.8 kB]                   
Get:4 https://packages.grafana.com/oss/deb stable InRelease [12.1 kB]                                      
Hit:5 http://security.ubuntu.com/ubuntu jammy-security InRelease                                           
Get:6 https://packages.grafana.com/oss/deb stable/main amd64 Packages [35.6 kB]
Fetched 262 kB in 0s (549 kB/s)   
Reading package lists... Done
W: https://packages.grafana.com/oss/deb/dists/stable/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
```

```
ubuntu@ip-172-31-91-177:~$ sudo apt-get update 
Hit:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-updates InRelease [114 kB]                                            
Get:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy-backports InRelease [99.8 kB]                                         
Hit:4 https://packages.grafana.com/oss/deb stable InRelease                                                                      
Hit:5 http://security.ubuntu.com/ubuntu jammy-security InRelease                                                                 
Fetched 214 kB in 0s (487 kB/s)
Reading package lists... Done
W: https://packages.grafana.com/oss/deb/dists/stable/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
```

Install Grafana

```
ubuntu@ip-172-31-91-177:~$ sudo apt-get install grafana -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  fontconfig-config fonts-dejavu-core libfontconfig1
The following NEW packages will be installed:
  fontconfig-config fonts-dejavu-core grafana libfontconfig1
0 upgraded, 4 newly installed, 0 to remove and 55 not upgraded.
Need to get 97.6 MB of archives.
After this operation, 326 MB of additional disk space will be used.
Get:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 fonts-dejavu-core all 2.37-2build1 [1041 kB]
Get:2 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 fontconfig-config all 2.13.1-4.2ubuntu5 [29.1 kB]
Get:3 http://us-east-1.ec2.archive.ubuntu.com/ubuntu jammy/main amd64 libfontconfig1 amd64 2.13.1-4.2ubuntu5 [131 kB]
Get:4 https://packages.grafana.com/oss/deb stable/main amd64 grafana amd64 9.2.1 [96.4 MB]
Fetched 97.6 MB in 2s (60.0 MB/s)   
Selecting previously unselected package fonts-dejavu-core.
(Reading database ... 63667 files and directories currently installed.)
Preparing to unpack .../fonts-dejavu-core_2.37-2build1_all.deb ...
Unpacking fonts-dejavu-core (2.37-2build1) ...
Selecting previously unselected package fontconfig-config.
Preparing to unpack .../fontconfig-config_2.13.1-4.2ubuntu5_all.deb ...
Unpacking fontconfig-config (2.13.1-4.2ubuntu5) ...
Selecting previously unselected package libfontconfig1:amd64.
Preparing to unpack .../libfontconfig1_2.13.1-4.2ubuntu5_amd64.deb ...
Unpacking libfontconfig1:amd64 (2.13.1-4.2ubuntu5) ...
Selecting previously unselected package grafana.
Preparing to unpack .../grafana_9.2.1_amd64.deb ...
Unpacking grafana (9.2.1) ...
Setting up fonts-dejavu-core (2.37-2build1) ...
Setting up fontconfig-config (2.13.1-4.2ubuntu5) ...
Setting up libfontconfig1:amd64 (2.13.1-4.2ubuntu5) ...
Setting up grafana (9.2.1) ...
Adding system user `grafana' (UID 114) ...
Adding new user `grafana' (UID 114) with group `grafana' ...
Not creating home directory `/usr/share/grafana'.
### NOT starting on installation, please execute the following statements to configure grafana to start automatically using systemd
 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable grafana-server
### You can start grafana-server by executing
 sudo /bin/systemctl start grafana-server
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.1) ...
Scanning processes...                                                                                                                                                                                                                  
Scanning linux images...                                                                                                                                                                                                               

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

Starting Grafana

```
ubuntu@ip-172-31-91-177:~$ sudo systemctl enable grafana-server 
Synchronizing state of grafana-server.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable grafana-server
Created symlink /etc/systemd/system/multi-user.target.wants/grafana-server.service → /lib/systemd/system/grafana-server.service.
ubuntu@ip-172-31-91-177:~$ sudo systemctl start grafana-server 
ubuntu@ip-172-31-91-177:~$ sudo systemctl status grafana-server 
● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2022-10-22 05:55:52 UTC; 23s ago
       Docs: http://docs.grafana.org
   Main PID: 3640 (grafana-server)
      Tasks: 6 (limit: 1143)
     Memory: 108.7M
        CPU: 1.018s
     CGroup: /system.slice/grafana-server.service
             └─3640 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini --pidfile=/run/grafana/grafana-server.pid --packaging=deb cfg:default.paths.logs=/var/log/grafana cfg:default.paths.data=/var/lib/grafana cfg:default.p>

Oct 22 05:55:56 ip-172-31-91-177 grafana-server[3640]: logger=live.push_http t=2022-10-22T05:55:56.879257127Z level=info msg="Live Push Gateway initialization"
Oct 22 05:55:57 ip-172-31-91-177 grafana-server[3640]: logger=infra.usagestats.collector t=2022-10-22T05:55:57.049446505Z level=info msg="registering usage stat providers" usageStatsProvidersLen=2
Oct 22 05:55:57 ip-172-31-91-177 grafana-server[3640]: logger=server t=2022-10-22T05:55:57.049811581Z level=info msg="Writing PID file" path=/run/grafana/grafana-server.pid pid=3640
Oct 22 05:55:57 ip-172-31-91-177 grafana-server[3640]: logger=provisioning.alerting t=2022-10-22T05:55:57.050549198Z level=info msg="starting to provision alerting"
Oct 22 05:55:57 ip-172-31-91-177 grafana-server[3640]: logger=provisioning.alerting t=2022-10-22T05:55:57.050707273Z level=info msg="finished to provision alerting"
Oct 22 05:55:57 ip-172-31-91-177 grafana-server[3640]: logger=http.server t=2022-10-22T05:55:57.069797873Z level=info msg="HTTP Server Listen" address=[::]:3000 protocol=http subUrl= socket=
Oct 22 05:55:57 ip-172-31-91-177 grafana-server[3640]: logger=ngalert t=2022-10-22T05:55:57.070788182Z level=info msg="warming cache for startup"
Oct 22 05:55:57 ip-172-31-91-177 grafana-server[3640]: logger=ticker t=2022-10-22T05:55:57.073425926Z level=info msg=starting first_tick=2022-10-22T05:56:00Z
Oct 22 05:55:57 ip-172-31-91-177 grafana-server[3640]: logger=grafanaStorageLogger t=2022-10-22T05:55:57.096280383Z level=info msg="storage starting"
Oct 22 05:55:57 ip-172-31-91-177 grafana-server[3640]: logger=ngalert.multiorg.alertmanager t=2022-10-22T05:55:57.097472747Z level=info msg="starting MultiOrg Alertmanager"
lines 1-21/21 (END)
[1]+  Stopped                 sudo systemctl status grafana-server
ubuntu@ip-172-31-91-177:~$ 
ubuntu@ip-172-31-91-177:~$ 
ubuntu@ip-172-31-91-177:~$ 
```
Access Grafana from a web browser

![image](https://user-images.githubusercontent.com/67490369/197322991-e9f9bca4-8ae0-471a-843c-5ba7d2f2e344.png)

Log in to Grafana with the username admin and password admin.

Reset the password when prompted.

![image](https://user-images.githubusercontent.com/67490369/197323032-e7a885f4-926a-44b8-abb7-eff0e4c5f56b.png)

Add the Prometheus data source
1.	Click Add data source.
2.	Select Prometheus.
3.	For the URL, enter http://x.x.x.x:9090. Note that x.x.x.x is the IP address of the Prometheus server.
4.	Click Save & Test. You should see a banner that says, Data source is working.
5.	Click the Explore icon on the left.
6.	In the PromQL Query input, enter a simple query, such as up.
7.	Click Run Query. Some data should then appear. If so, congratulations! This data comes from the Prometheus server.

![image](https://user-images.githubusercontent.com/67490369/197323092-59c4abca-b94d-494e-a694-6cf99d5d2e4b.png)

![image](https://user-images.githubusercontent.com/67490369/197323323-02895ad6-11ae-40b0-a65f-c4e160850e76.png)

**Create the Dashboard and Add a Web Server Status Panel**

1.	Click the Create button on the left, and then select Dashboard.
2.	Click the Save Dashboard button near the top right.
3.	For the dashboard name, enter "My Web Server".
4.	Click Save.
5.	Click the Add panel button near the top right.
6.	Click Add Query.
7.	For the PromQL query, enter: up{instance="x.x.x.x:9100"} 
8.	Click the Visualization icon.
9.	Click the visualization type dropdown (which currently says Graph) and change it to Stat.
10.	Under Value Mappings, enter two value to text mappings:
    1 -> Up
    0 -> Down
11.	Click the General icon.
12.	Change the panel title to "Server Status".
13.	Click the back button in the top left (next to LimeDrop Web Server). You should see your dashboard, and the Server Status panel should say Up.
14.	Click the Save Dashboard button near the top right, and then Save to save your changes.

**Create a CPU Usage Graph Panel**

1.	Click the Add panel button near the top right.
2.	Click Add Query.
3.	For the PromQL query, enter: sum(rate(node_cpu_seconds_total{instance='x.x.x.x:9100',mode!='idle'}[5m])) * 100 
4.	Click the General icon.
5.	Change the panel title to "CPU Usage".
6.	Click the back button in the top left. You should see your dashboard, and there should be a graph showing CPU utilization.
7.	Click the Save Dashboard button near the top right, and then Save to save your changes.

**Create a Memory Usage Graph Panel**

1.	Click the Add Panel button near the top right.
2.	Click Add Query.
3.	For the PromQL query, enter: 100 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 
4.	Click the General icon.
5.	Change the panel title to "Memory Usage".
6.	Click the back button in the top left. You should see your dashboard, and there should be a graph showing memory utilization.
7.	Rearrange your panels by dragging and dropping them if desired.
8.	Click the Save Dashboard button near the top right, and then Save to save your changes.



![image](https://user-images.githubusercontent.com/67490369/197323867-b41340bc-4e82-4343-a62c-698a1a4d2a59.png)
