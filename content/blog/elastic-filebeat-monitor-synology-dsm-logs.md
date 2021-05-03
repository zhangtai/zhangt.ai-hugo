---
title: "Setup Elastic stack to monitor Synology NAS system logs"
date: 2018-03-14T22:16:48+08:00
---

> latest update at 2018-03-16

I have 2 physical server: logs receiving server running full elastic stack on Ubuntu by docker-compose, Synology NAS server which generates the logs.

## Setup full stack of elastic on destination server

Clone the official docker-compose file on [github](https://github.com/elastic/stack-docker), since the latest version of elastic is 6.2.2 (as of 14-Mar-2018, you can check the latest docker version by this [link](https://www.docker.elastic.co/)) you may need to change the version number in `.env` file to it.

I am going to send logs form another server(NAS) so I modified the config file of logstash in `/config/` folder of the repo to below.

``` js
input {
    beats {
    port => "5044"
  }
}

filter {
  if [source] == "/synolog/synorelayd.log" {
    grok {
      match => { "message" => "(?<time>%{SYSLOGTIMESTAMP} %{YEAR}) %{GREEDYDATA:logbody}" }
    }
    date {
      match => [
        "time",
        "MMM  d HH:mm:ss YYYY",
        "MMM dd HH:mm:ss YYYY"
        ]
      target => ["@timestamp"]
    }
  } else if [source] in [
    "/synolog/router.log",
    "/synolog/synoupdate.log",
    "/synolog/dpkg_upgrade.log",
    "/synolog/synopkg.log"
    ] {
    grok {
      match => { "message" =>
        "(?<time>%{YEAR}/%{MONTHNUM}/%{MONTHDAY} %{TIME})\s*%{GREEDYDATA:logbody}"
      }
    }
    date {
      match => [ "time", "YYYY/MM/dd HH:mm:ss" ]
      target => ["@timestamp"]
    }
  } else {
    grok {
      match => { "message" => "%{TIMESTAMP_ISO8601:time} %{GREEDYDATA:logbody}" }
    }
    date {
      match => [
        "time",
        "ISO8601",
        "YYYY-MM-dd HH:mm:ss"
        ]
      target => ["@timestamp"]
    }
  }
}

output {
  elasticsearch {
    hosts    => [ 'elasticsearch' ]
    user     => 'elastic'
    password => 'MyPassword'
  }
}
```

I have also midified the opening port and host in docker-compose.yml file to let others access within the same LAN network

``` yml
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch-platinum:${TAG}
    container_name: elasticsearch
    volumes:
      - ./esdata:/usr/share/elasticsearch/data
    environment: [
      'http.host=0.0.0.0',
      'transport.host=127.0.0.1',
      'ELASTIC_PASSWORD=${ELASTIC_PASSWORD}'
    ]
    ports: ['0.0.0.0.:9200:9200']
    networks: ['stack']

  kibana:
    image: docker.elastic.co/kibana/kibana:${TAG}
    container_name: kibana
    environment:
      - ELASTICSEARCH_USERNAME=kibana
      - ELASTICSEARCH_PASSWORD=${ELASTIC_PASSWORD}
    ports: ['0.0.0.0:5601:5601']
    networks: ['stack']
    depends_on: ['elasticsearch']

  logstash:
    image: docker.elastic.co/logstash/logstash:${TAG}
    container_name: logstash
    environment:
      - 'xpack.monitoring.elasticsearch.password=${ELASTIC_PASSWORD}'
    # Provide a simple pipeline configuration
    # for Logstash with a bind-mounted file.
    volumes:
      - ./config/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    networks: ['stack']
    ports: ['0.0.0.0:5044:5044']
    depends_on: ['elasticsearch', 'setup_logstash']
```

Start the full stack as a daemon by running `docker-compose up -d` in the repo folder. You can also run `docker ps` to check the containers running status.

## Setup filebeat on NAS server to send logs

I have docker installed on NAS server as well, so it's very easy to get filebeat running by just 1 command:

`docker run --rm --name filebeat --user root -d -v /volume1/codes/elk-docker/filebeat/config/filebeat.yml:/usr/share/filebeat/filebeat.yml -v /var/log:/synolog:ro docker.elastic.co/beats/filebeat:6.2.2`

But before running, create you own `filebeat.yml` file with below content.

``` yml
filebeat.prospectors:
- type: log
  paths:
    - /synolog/*.log
  exclude_files: [
    'synoindex\.log$',
    'synocrond-execute',
    'space_operation_error',
    'synocmsclient.log',
    'synopkg.log',
    'dpkg_upgrade.log',
    'calendar.log',
    'doc_export_word.log',
    'synochat.log'
  ]

- type: log
  paths:
    - /synolog/synoindex.log
    - /synolog/synochat.log
  multiline.pattern: '^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}'
  multiline.negate: true
  multiline.match: after

- type: log
  paths:
    - /synolog/synopkg.log
    - /synolog/dpkg_upgrade.log
  multiline.pattern: '^\d{4}\/\d{2}\/\d{2} \d{2}:\d{2}:\d{2}'
  multiline.negate: true
  multiline.match: after

output.logstash:
  hosts: ["ip_address_of_elastic_stac:5044"]
```

After started, run `docker logs filebeat` to see if any ERROR of the filebeat container, if no, go ahead to check the results.

`curl -X GET http://elastic:MyPassword@ip_address_of_elastic_stac:9200/logstash-2018.03.14/_search`

``` json
{
    "_index": "logstash-2018.03.14",
    "_type": "doc",
    "_id": "V8XgLGIBbRQAhdRMb3e4",
    "_score": 1.0,
    "_source": {
        "@version": "1",
        "tags": ["beats_input_codec_plain_applied"],
        "prospector": {
            "type": "log"
        },
        "time": "2018-03-14T17:36:00+08:00",
        "offset": 9603,
        "@timestamp": "2018-03-14T09:36:00.000Z",
        "source": "/synolog/fileindexd.log",
        "message": "2018-03-14T17:36:00+08:00 TNAS fileindexd: optree.cpp:194 Got event: \
          {\"event\":4,\"is_dir\":false,\"path\":\"/volume1/codes/notes/elk.md\"}",
        "host": "3fbd23495dab",
        "beat": {
            "version": "6.2.2",
            "name": "3fbd23495dab",
            "hostname": "3fbd23495dab"
        },
        "logbody": "TNAS fileindexd: optree.cpp:194 Got event: \
         {\"event\":4,\"is_dir\":false,\"path\":\"/volume1/codes/notes/elk.md\"}"
    }
}
```
