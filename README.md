# Telegraf influxdb and grafana aka TIG stack

I created this repository to learn TIG stack. Aim of this project is to monitor and compare Internet service provider quality of service by doing 

	* latency meassurements with DNS response times from well known DNS servers and ISP provided DNS server.
	* meassuring aage load time of well known websites.
	* meassure above seperate over IPv4 and IPv6 protocol stack. This will give opportunity to compare responses via both protocol stack.

Telegraf is used to collect the metrics Influxdb is used as time seriese database to write those metrics and Grafana to visualise them.

This project is suitable to run TIG stack as systemd service on a small computer like raspberry pi. I recommend using wired connection instead of wireless so get reliable meassurement result. 

Docker compose file is provided to run TIG stack as its own containers. Keep in mind that you would have to have Globally unique IP address for IPv6 to work which I was unable to assign to telegraf container at the time of writing. 

## Telegraf

`dns_query` plugin is used to perform DNS query responses for well known domains from public DNS servers. 
`ping` plugin is used to meassure response time of page load over a perticular IP stack. 

## Influxdb

Telegraf creates a database called `latency` if not exist which will have two meassurements `dns_query` and `ping`. 

```
$ influx
Connected to http://localhost:8086 version 1.8.9
InfluxDB shell version: 1.8.9
> show database
ERR: error parsing query: found DATABASE, expected CONTINUOUS, DATABASES, DIAGNOSTICS, FIELD, GRANTS, MEASUREMENT, MEASUREMENTS, QUERIES, RETENTION, SERIES, SHARD, SHARDS, STATS, SUBSCRIPTIONS, TAG, USERS at line 1, char 6
Warning: It is possible this error is due to not setting a database.
Please set a database with the command "use <database>".
> show databases
name: databases
name
----
telegraf
_internal
latency
> use latency
Using database latency
> show measurements
name: measurements
name
----
dns_query
ping
```

Data is written with following tags

```
show series
dns_query,domain=facebook.com,host=pi01,record_type=A,result=timeout,server=8.8.8.8
dns_query,domain=facebook.com,host=pi01,record_type=A,result=timeout,server=9.9.9.9
dns_query,domain=facebook.com,host=pi01,record_type=AAAA,result=error,server=2001:4860:4860::8888
dns_query,domain=facebook.com,host=pi01,record_type=AAAA,result=error,server=2606:4700:4700::1111
ping,host=pi01,url=facebook.com
ping,host=pi01,url=google.com
ping,host=pi01,url=linkedin.com
ping,host=pi01,url=twitter.com
ping,host=pi01,url=youtube.com
```

Telegraf would create following meassuement records on influxdb

```
> select * from ping limit 10
name: ping
time                average_response_ms host maximum_response_ms minimum_response_ms packets_received packets_transmitted percent_packet_loss result_code standard_deviation_ms ttl url
----                ------------------- ---- ------------------- ------------------- ---------------- ------------------- ------------------- ----------- --------------------- --- ---
1613668240000000000 28.370046           pi01 28.370046           28.370046           1                1                   0                   0           0                     56  facebook.com
1613668240000000000 41.703877           pi01 41.703877           41.703877           1                1                   0                   0           0                     112 youtube.com
1613668240000000000 39.708606           pi01 39.708606           39.708606           1                1                   0                   0           0                     112 google.com
1613668240000000000 51.394439           pi01 51.394439           51.394439           1                1                   0                   0           0                     57  twitter.com
1613668240000000000 19.081498           pi01 19.081498           19.081498           1                1                   0                   0           0                     119 linkedin.com
1613668250000000000 52.052894           pi01 52.052894           52.052894           1                1                   0                   0           0                     57  twitter.com
1613668250000000000 25.056108           pi01 25.056108           25.056108           1                1                   0                   0           0                     119 linkedin.com
1613668250000000000 45.357627           pi01 45.357627           45.357627           1                1                   0                   0           0                     112 youtube.com
1613668250000000000 40.042917           pi01 40.042917           40.042917           1                1                   0                   0           0                     56  facebook.com
1613668250000000000 45.277184           pi01 45.277184           45.277184           1                1                   0                   0           0                     112 google.com
> select * from dns_query limit 10
name: dns_query
time                domain       host query_time_ms rcode   rcode_value record_type result  result_code server
----                ------       ---- ------------- -----   ----------- ----------- ------  ----------- ------
1613668240000000000 facebook.com pi01 37.521301     NOERROR 0           A           success 0           1.1.1.1
1613668240000000000 facebook.com pi01 24.285766     NOERROR 0           A           success 0           208.67.222.123
1613668240000000000 facebook.com pi01 64.683141     NOERROR 0           A           success 0           9.9.9.9
1613668240000000000 facebook.com pi01 34.285546     NOERROR 0           AAAA        success 0           2620:fe::fe
1613668240000000000 facebook.com pi01 38.471029     NOERROR 0           AAAA        success 0           2001:4860:4860::8888
1613668240000000000 facebook.com pi01 26.28337      NOERROR 0           A           success 0           8.8.8.8
1613668240000000000 facebook.com pi01 34.153065     NOERROR 0           AAAA        success 0           2606:4700:4700::1111
1613668240000000000 facebook.com pi01 19.241404     NOERROR 0           AAAA        success 0           2620:119:53::53
1613668240000000000 google.com   pi01 19.185497     NOERROR 0           AAAA        success 0           2620:fe::fe
1613668240000000000 google.com   pi01 38.506325     NOERROR 0           A           success 0           1.1.1.1
```

## Grafana

I created few dashboards as per below. 

**TODO**

Following are InfuxQL Queries used for these dashboards. 
```
TODO
```

## TODO

* Address IPv6 challenges with `docker-compose`. Have an independent network stack and assign GUA IPv6 address to each container and ensure it can access each other with its IP address. 
