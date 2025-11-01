# Logstash JSON Parsing #

## Logstash Prereqs ##

Install the logstash-output-opensearch plugin.

This must be reinstalled every time Logstash is updated. Use caution when updating Logstash.

Find the logstash base directory if you do not know where it is.

```
find / -type f -wholename "*bin/logstash-plugin" 2>/dev/null | sed "s/\/bin\/logstash-plugin//g"
```

Then enter the base directory and execute bin/logstash-plugin install logstash-output-opensearch

```
cd /usr/share/logstash/
sudo bin/logstash-plugin install logstash-output-opensearch
```

An internet connection is required or you must look up how to manually install a Gem.

## Logstash Configuration ##

__For Nginx HTTP Server__

The following Logstash configuration is to receive the Nginx HTTP Server's JSON syslogs and parse them into Elastic Common Schema complaint logs.

```
input {
  syslog {
    port => 8514
    host => "0.0.0.0"
    codec => "plain"
  }
}

filter {
  json {
    source => "message"
    target => "nginx"
    remove_field => "message"
  }

  grok {
    match => {
      "[nginx][server_protocol]" => "%{WORD:[http][protocol]}/%{GREEDYDATA:[http][version]}"
    }
    remove_field => [ "[nginx][server_protocol]" ]
  }

  if ([@metadata][server_protocol_split][0] and [@metadata][server_protocol_split][1]) {
    mutate {
      rename => {
        "[@metadata][server_protocol_split][0]" => "[http][protocol]"
        "[@metadata][server_protocol_split][1]" => "[http][version]"
      }
    }
  }

  mutate {
    rename => {
      "[nginx][body_bytes_sent]" => "[http][response][body][bytes]"
      "[nginx][bytes_sent]" => "[http][response][bytes]"
      "[nginx][http_referrer]" => "[http][request][referrer]"
      "[nginx][status]" => "[http][response][status_code]"
      "[nginx][uri]" => "[url][path]"
      "[nginx][scheme]" => "[url][scheme]"
      "[nginx][remote_addr]" => "[source][ip]"
      "[nginx][http_user_agent]" => "[user_agent][original]"
      "[nginx][remote_user]" => "[user][name]"
      "[nginx][host]" => "[url][domain]"
      "[nginx][request_length]" => "[http][request][bytes]"
      "[nginx][request_method]" => "[http][request][method]"
      "[nginx][request_content_type]" => "[http][request][content_type]"
      "[nginx][request_uri]" => "[http][request][path]"
      "[nginx][request]" => "[http][request][original]"
      "[nginx][request_id]" => "[http][request][id]"
    }
  }

  ruby {
    code => "event.set('[http][response][duration_millis]', event.get('[nginx][request_time]') * 1000)"
    remove_field => [ "[nginx][request_time]" ]
  }

  if "_jsonparsefailure" not in [tags] {
    mutate {
      remove_field => [ "[event][original]" ]
    }
  }
}

output {
  opensearch {
    hosts       => ["https://changeme1:9200", "https://changeme2:9200", "https://changeme3:9200"]
    user        => "changeme"
    password    => "changeme"
    index       => "web-jsonlog-%{+YYYY.MM.dd}"
    ssl_certificate_verification => true
    manage_template => false
  }
}
```

