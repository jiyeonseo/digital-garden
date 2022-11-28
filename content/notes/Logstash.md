---
title: "Logstash"
tags:
- logstash
- wip
---

## Configuration

```yaml
input {
  beats {
    port => 5044
  }
}

filter{
}

output {
  s3 {
    region => "us-east-1"
    bucket => "log-bucket"
    prefix => "%{+YYYY}/%{+MM}/%{+dd}"
    codec => line { format => "%{message}"}
  }
}

```

## Input
### filebeat

### file


## Filter

## Output
### S3
- ``

## References 
- [Using Logstash to Send Directly to an S3 Object Store](https://joshua-robinson.medium.com/using-logstash-to-send-directly-to-an-s3-object-store-34a4365a0960)