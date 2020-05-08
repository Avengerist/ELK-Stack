# ELK-Stack
How to configure and install ELK Stack<br>

<h2>Server Configuration:</h2><br>

`systemctl stop firewalld && systemctl disable firewalld`<br>

`setenforce 0`<br>

`yum install nano -y`<br>

`yum install java-11-openjdk java-11-openjdk-devel -y`<br>

`yum install wget -y`<br>

<h2>ELASTICSEARCH:</h2><br>
  
`wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.2-x86_64.rpm`<br>

`rpm -ivh elasticsearch-7.6.2-x86_64.rpm`<br>

`systemctl daemon-reload`<br>

`systemctl enable elasticsearch`<br>

`nano /etc/elasticsearch/elasticsearch.yml`<br>

#uncomment network.host and add 127.0.0.1<br>
>network.host 127.0.0.1 <br>

`systemctl start elasticsearch`<br>

`ss -tulnp | grep 9200`<br>

`curl localhost:9200`<br>

<h2>KIBANA:</h2><br>

`wget https://artifacts.elastic.co/downloads/kibana/kibana-7.6.2-x86_64.rpm`<br>

`rpm -ivh kibana-7.6.2-x86_64.rpm`<br>

`systemctl daemon-reload`<br>

`systemctl enable kibana.service`<br>

`nano /etc/kibana/kibana.yml`<br>
Ucomment server.host and add ip<br>
example:<br>

>server.host: "192.168.2.106"

`systemctl start kibana.service`<br>

`ss -tulnp | grep 5601`<br>

Wait about 1-2 minute. It takes some time for start.<br>

You can check kibana in browser, just enter  `ip_address:5601`

<h2>LOGSTASH:</h2><br>

`wget https://artifacts.elastic.co/downloads/logstash/logstash-7.6.2.rpm`<br>

`rpm -ivh logstash-7.6.2.rpm`<br>

`systemctl enable logstash`<br>



`nano /etc/logstash/conf.d/01-logstash-simple.conf`
```
  if [type] == "syslog" {
    grok {
      match => {
         "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}"
      }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    syslog_pri { }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
     }
  }
}

output {
   elasticsearch {
     hosts => "localhost:9200"
     index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
   }
}
```



<h2>Client Configuration:</h2><br>

`systemctl stop firewalld`<br>

`setenforce 0`<br>

`yum install wget -y`<br>

<h2>FILEBEAT:</h2><br>

`wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.2-x86_64.rpm`<br>

`rpm -ivh filebeat-7.6.2-x86_64.rpm`<br>

`nano /etc/filebeat/filebeat.yml`
```
enabled: true
paths:
   - /var/log/*.log
   - /var/log/messages
```
Commit output.elasticsearch<br>
Uncommit output.logstash<br>
And setup IP address<br>
Example: 
> hosts: ["192.168.2.108:5044"]<br>

`systemctl start filebeat`<br>

`systemctl enable filebeat`<br>

Open browser and configure kibana which can be accessed for example:
>192.168.2.108:5601
