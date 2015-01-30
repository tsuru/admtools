# Admtools plugin for tsuru

This project is just a bunch of hacks to avoid sysadmins freaking out when something wrong is not right =) 
It's intended to give some tools for fast debugging into a tsuru platform

## Dependencies
You will need to install tsuru and tsuru-admin commands, admin permission in tsuru and have your public key in all docker nodes and RPaaS VMs

## Installing

As easy as any other tsuru plugin
```bash
$ tsuru plugin-install admtools https://raw.githubusercontent.com/tsuru/admtools/master/admtools
$ tsuru admtools
Usage: tsuru admtools [ -l|--node-list <pool> ] | [ -x|--node-exec <pool> 'cmd' ] | [ --check-app|-c appname <path> ] | [ --check-app-router|-r appname <path> ] | [ -s|--rpaas-status-code-per-minute ip-rpaas <stringlog> ] | [ -u|--rpaas-url-equal-per-minute ip-rpaas <stringlog> ] | [ --help|-h ]
```
## Just trying out

Listing all pool names in your cloud

```bash
$ tsuru admtools -l
```

Listing all nodes of a specific pool

```bash
$ tsuru admtools -l pool-name
```

Connectivity test in all nodes of a specific pool 

```bash
$ tsuru admtools -x pool-name 'curl -s --connect-timeout 3 google.com:80 && echo OK || echo BAD'
$ tsuru admtools -x pool-name 'curl -x proxy:3128 -s --connect-timeout 3 google.com:80 && echo OK || echo BAD'
```

Check all unit containers of a specific application

```bash
$ #List of containers via redis(hipache)
$ REDIS_PARAMETERS="-h redis-hipache.yourcompany.com" tsuru admtools -r appname /healthcheck/
$ #List of containers via tsuru
$ tsuru admtools -c appname /healthcheck/
```

Group the nginx log of a VM created by RPaaS Service, giving group of status-code per minute 

```bash
$ #All minutes in the current hour
$ tsuru admtools -s ip-rpaas 
$ #All minutes of a specified time
$ tsuru admtools -s ip-rpaas 30/Jan/2015:15:41
```

Group the nginx log of a VM created by RPaaS Service, giving group of urls per minute(limited by 20) 

```bash
$ #All minutes in the current hour
$ tsuru admtools -u ip-rpaas 
$ #All minutes of a specified time
$ tsuru admtools -u ip-rpaas 30/Jan/2015:15:41
```


## Links

http://docs.tsuru.io/en/latest/using/cli/plugins.html
