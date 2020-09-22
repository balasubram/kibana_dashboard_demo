# Kibana Dashboard Demo
This repository contains some sample dashboards created for Apache Web Logs collected from [secrepo.com](https://www.secrepo.com/self.logs/)

And the wonderful analytics that comes along with just plain web access logs as follows:
![Web Logs](https://github.com/nich07as/kibana_dashboard_demo/blob/master/kibana_dashboard_sample.png)

To use these dashboards, you will need to do the following:

1. Use the following index template mappings (you know to get the GeoIP etc formatted correctly)

```
PUT _index_template/apache-logs
{
  "index_patterns": ["apache-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    },
    "mappings": {
            "dynamic_templates": [
        {
          "strings_as_keywords": {
            "match_mapping_type": "string",
            "mapping": {
              "type": "keyword"
            }
          }
        }
      ],
      "properties": {
        "clientip": {
          "type": "ip"
        },
        "bytes": {
          "type": "long"
        },
        "geoip": {
          "properties": {
            "dma_code": {
              "type": "long"
            },
            "ip": {
              "type": "ip"
            },
            "latitude": {
              "type": "float"
            },
            "location": {
              "type": "geo_point"
            },
            "longitude": {
              "type": "float"
            }
          }
        },
        "timestamp": {
          "type": "date",
          "format": "dd/MMM/yyyy:HH:mm:ss Z"
        }
      }
    }
  }
}
```
2. Incase you're using logstash to ingest (which I did cuz I so happen to have it running on my laptop) here's the config snippet

```
input {
  file {
    path => ["path to your log location"]
    sincedb_path => "/tmp/logstash"
    start_position => "beginning"
    mode => "read"
    file_completed_log_path => "/tmp/logstash"
    file_completed_action => "log"
  }
}

filter {
  grok {
      match => { "message" => "%{COMMONAPACHELOG} %{QS:referrer} %{QS:browser_agent}" }
  }
  date {
    match => [ "timestamp", "dd/MMM/YYYY:HH:mm:ss Z" ]
    locale => en
  }
  geoip {
    source => "clientip"
  }
  useragent {
    source => "browser_agent"
    target => "useragent"
  }
}
output {
  stdout {
    codec => dots
  }
  elasticsearch {
    hosts => ["your elasticsearch cloud instance"]
    user => "username"
    password => "password"
    index => "apache-demo"
    # pipe => "apache-logs-pipeline" ## Optional to use for parsing
  }
}
```
3. Once ingested, create your kibana index pattern [more information here](https://www.elastic.co/guide/en/kibana/current/index-patterns.html#:~:text=An%20index%20pattern%20tells%20Kibana,clouds%2C%20and%20more%20in%20Visualize.)
4. Import the [Kibana objects](https://github.com/nich07as/kibana_dashboard_demo/blob/master/kibana_access_log_demo.ndjson) to your Kibana Instance via the steps [here](https://www.elastic.co/guide/en/kibana/current/managing-saved-objects.html#managing-saved-objects-export-objects)

Please let me know if you have any questions through issues in this repo. Thanks!
