# Nginx JSON Logging #

## Setting up the log format ##

__HTTP Server__

Add the following to /etc/nginx/nginx.conf or an auxillary configuration file.

```
log_format json_http_logs escape=json
    '{'
    '"time_local":"$time_local",'
    '"time_iso8601":"$time_iso8601",'
    '"host":"$host",'
    '"uri":"$uri",'
    '"scheme":"$scheme",'
    '"server_protocol":"$server_protocol",'
    '"remote_addr":"$remote_addr",'
    '"remote_user":"$remote_user",'
    '"request":"$request",'
    '"request_id":"$request_id",'
    '"request_uri":"$request_uri",'
    '"request_length":"$request_length",'
    '"request_method":"$request_method",'
    '"request_content_type":"$content_type",'
    '"status": $status,'
    '"body_bytes_sent":$body_bytes_sent,'
    '"bytes_sent":"$bytes_sent",'
    '"request_time":$request_time,'
    '"http_referrer":"$http_referer",'
    '"http_user_agent":"$http_user_agent"'
    '}';
```

__Stream Server__

WIP

## Using the log format

__HTTP Server__

To use this log format inside of a server or location, provide the following:

```
access_log /path/to/access_log.json json_http_logs;
```

Or for syslog:

```
access_log syslog:server=127.0.0.1:8514 json_http_logs;
```

Replace 127.0.0.1 with the syslog collector's IP and 8514 with the syslog collector's port.

__Stream Server__

WIP
