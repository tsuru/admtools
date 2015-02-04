# Admtools plugin for tsuru

This project is just a bunch of hacks to avoid sysadmins freaking out when something wrong is not right =). 
It's intended to give some tools for fast debugging into a tsuru platform

## Dependencies
You will need to install tsuru and tsuru-admin commands, admin permission in tsuru and have your public key in all docker nodes and RPaaS VMs

## Installing

As easy as any other tsuru plugin (use the same command to upgrade)
```bash
$ tsuru plugin-install admtools https://raw.githubusercontent.com/tsuru/admtools/master/admtools
$ tsuru admtools
Usage: tsuru admtools [ -l|--node-list <pool> ] | [ -x|--node-exec <pool> 'cmd' ] | [ --check-app|-c appname <path> ] | [ --check-app-router|-r appname <path> ] | [ -m|--rpaas-per-minute <status-code>|url|bad-url|slow-url ip-rpaas <stringlog> ] | [ --help|-h ]
```
## Just trying out

Listing all pool names in your cloud

```bash
$ tsuru admtools -l
pool-name
```

Listing all nodes of a specific pool

```bash
$ tsuru admtools -l pool-name
http://10.10.10.10:4243
```

Connectivity test in all nodes of a specific pool 

```bash
$ tsuru admtools -x pool-name 'curl -s --connect-timeout 3 google.com:80 >/dev/null && echo OK || echo BAD'
Docker Node: 10.10.10.10
Warning: Permanently added '10.10.10.10' (RSA) to the list of known hosts.
BAD
$ tsuru admtools -x pool-name 'curl -x proxy:3128 -s --connect-timeout 3 google.com:80 >/dev/null && echo OK || echo BAD'
Docker Node: 10.10.10.10
Warning: Permanently added '10.10.10.10' (RSA) to the list of known hosts.
OK
```

Check all unit containers of a specific application with -c or -r

```bash
$ #List of containers via redis(of router hipache - you must use the entire application domain)
$ REDIS_PARAMETERS="-h redis-hipache.yourcompany.com" tsuru admtools.yourcompany.com -r appname /healthcheck/
unit: 10.10.10.10:49282/healthcheck/
real	0m0.094s
status: OK
$ #List of containers via tsuru - just use the application name here
$ tsuru admtools -c appname /healthcheck/
unit: c59bc17454 started 10.10.10.10:49282/healthcheck/
real	0m0.021s
status: OK
$ #Or without path
$ tsuru admtools -c appname 
unit: c59bc17454 started 10.10.10.10:49282
real	0m0.021s
status: OK
```

Group the nginx log of a VM created by RPaaS Service, giving group of status-code per minute 

```bash
$ #All minutes in the current hour(status-code is the default)
$ tsuru admtools -m status-code ip-rpaas 
    660 04/Feb/2015:15:00:
      1 302
      2 301
     13 304
     54 404
    590 200
    637 04/Feb/2015:15:01:
      1 301
     17 304
     50 404
    569 200
$ #Or just
$ tsuru admtools -m ip-rpaas 
    660 04/Feb/2015:15:00:
      1 302
      2 301
     13 304
     54 404
    590 200
    637 04/Feb/2015:15:01:
      1 301
     17 304
     50 404
    569 200
$ #All minutes of a specified time
$ tsuru admtools -m ip-rpaas $(date +%d/%b/%Y):13:4[0-1]
    660 04/Feb/2015:13:40:
      1 302
      2 301
     13 304
     54 404
    590 200
    637 04/Feb/2015:13:41:
      1 301
     17 304
     50 404
    569 200
$ #Or just
$ tsuru admtools -m ip-rpaas 2015:13:4[0-1]
    660 04/Feb/2015:13:40:
      1 302
      2 301
     13 304
     54 404
    590 200
    637 04/Feb/2015:13:41:
      1 301
     17 304
     50 404
    569 200
```

Group the nginx log of a VM created by RPaaS Service, giving group of urls per minute(limited by 20) 

```bash
$ #All minutes in the current hour
$ tsuru admtools -m url ip-rpaas 
    660 04/Feb/2015:15:00:
      3 app.company.com GET /questions/tag/whatsapp.json 200
     21 0.0.0.0 GET /healthcheck 200
     24 _tsuru_nginx_app GET /_nginx_healthcheck/ 200
     63 app.company.com GET /upfiles/logo_forum2_1.jpg 200
     76 app.company.com GET /m/default/media/images/favicon.ico 200
     85 app.company.com GET /cstyle.css 200
    637 04/Feb/2015:15:01:
      8 app.company.com GET /tags/jogos/?type=rss 200
     12 app.company.com GET /questions/tag/jogos.json 200
     19 0.0.0.0 GET /healthcheck 200
     24 _tsuru_nginx_app GET /_nginx_healthcheck/ 200
     68 app.company.com GET /m/default/media/images/favicon.ico 200
     73 app.company.com GET /cstyle.css 200
$ #All minutes of a specified time
$ tsuru admtools -m url ip-rpaas 04/Feb/2015:13:4
    660 04/Feb/2015:13:40:
      3 app.company.com GET /questions/tag/whatsapp.json 200
     21 0.0.0.0 GET /healthcheck 200
     24 _tsuru_nginx_app GET /_nginx_healthcheck/ 200
     63 app.company.com GET /upfiles/logo_forum2_1.jpg 200
     76 app.company.com GET /m/default/media/images/favicon.ico 200
     85 app.company.com GET /cstyle.css 200
    637 04/Feb/2015:13:41:
      8 app.company.com GET /tags/jogos/?type=rss 200
     12 app.company.com GET /questions/tag/jogos.json 200
     19 0.0.0.0 GET /healthcheck 200
     24 _tsuru_nginx_app GET /_nginx_healthcheck/ 200
     68 app.company.com GET /m/default/media/images/favicon.ico 200
     73 app.company.com GET /cstyle.css 200
```

Group the nginx log of a VM created by RPaaS Service, giving group of "BAD" urls per minute(limited by 20) discarding 20x, 30x and 404 status-code

```bash
$ #All minutes in the current hour
$ tsuru admtools -m bad-url ip-rpaas 
    635 04/Feb/2015:16:00:
    769 04/Feb/2015:16:01:
      1 04/Feb/2015:16:01:48 -0200 _tsuru_nginx_app GET /tags/internet/&=&2=&e=&i=&i=&L=&O=&0=&s=&r=&r=&u=&u=&t=&%=&z=&z=&type=rss HTTP/1.1 400
    738 04/Feb/2015:16:02:
$ #All minutes of a specified time
$ tsuru admtools -m bad-url ip-rpaas 04/Feb/2015:07:4[2-7]
    313 04/Feb/2015:07:42:
    304 04/Feb/2015:07:43:
      1 04/Feb/2015:07:43:04 -0200 _tsuru_nginx_app ---41184676334-- 400
    262 04/Feb/2015:07:44:
    287 04/Feb/2015:07:45:
    349 04/Feb/2015:07:46:
      1 04/Feb/2015:07:46:17 -0200 0.0.0.0 GET /healthcheck HTTP/1.1 499
    289 04/Feb/2015:07:47:
```

Group the nginx log of a VM created by RPaaS Service, giving the last 20 slowest urls

```bash
$ #All minutes in the current hour
$ tsuru admtools -m slow-url ip-rpaas 
    585 04/Feb/2015:18:00:
04/Feb/2015:18:00:42 -0200 app.company.com GET /tags/easports/ HTTP/1.1 200 0.595
04/Feb/2015:18:00:33 -0200 app.company.com GET /tags/erro/ HTTP/1.1 200 0.885
04/Feb/2015:18:00:16 -0200 app.company.com GET /tags/jogos/ HTTP/1.1 200 1.013
04/Feb/2015:18:00:14 -0200 app.company.com GET /cstyle.css HTTP/1.1 200 2.448
    520 04/Feb/2015:18:01:
04/Feb/2015:18:01:20 -0200 app.company.com GET /perguntas/8022 HTTP/1.1 200 0.607
04/Feb/2015:18:01:19 -0200 app.company.com GET /perguntas/23067 HTTP/1.1 200 0.740
04/Feb/2015:18:01:36 -0200 app.company.com GET /tags/linux/ HTTP/1.1 200 0.770
04/Feb/2015:18:01:00 -0200 app.company.com GET /perguntas/ HTTP/1.1 200 3.024
04/Feb/2015:18:01:34 -0200 app.company.com GET /usuarios/8212/ HTTP/1.0 200 3.351
```

Links:

- Full Plugin documentation: http://docs.tsuru.io/en/latest/using/cli/plugins.html
- RPaaS Service documentation: https://github.com/tsuru/rpaas

FAQ:

1 - Log Format: This plugin only supports the log format bellow:

```bash
    log_format main
      '$remote_addr\t$time_local\t$host\t$request\t$http_referer\t$http_x_mobile_group\t'
      'Local:\t$status\t$body_bytes_sent\t$request_time\t'
      'Proxy:\t$upstream_cache_status\t$upstream_status\t$upstream_response_length\t$upstream_response_time\t'
      'Agent:\t$http_user_agent\t'
      'Fwd:\t$http_x_forwarded_for';
```
