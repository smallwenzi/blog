title: 查询rabbitmq 队列信息
author: wen
tags:
  - rabbitMq
  - spring cloud
categories:
  - spring cloud
date: 2019-05-29 16:48:00
---
最近需要做mq 消息预警，消息一旦堆积某个数量就报警。

RabbitMQ自己就提供了HTTP API手册，比如我本地的API手册地址为：http://localhost:15672/api

http://localhost:15672/api/queues/%2F/soaSyncPwdQueue
可以看到队列相关的所有信息都有记录。
```
{
  "consumer_details": [
    {
      "arguments": {
        
      },
      "channel_details": {
        "connection_name": "10.143.172.86:35833 -> 10.210.93.176:5672",
        "name": "10.143.172.86:35833 -> 10.210.93.176:5672 (1)",
        "node": "rabbit@CNQLS03247",
        "number": 1,
        "peer_host": "10.143.172.86",
        "peer_port": 35833,
        "user": "logincenter"
      },
      "ack_required": true,
      "consumer_tag": "amq.ctag-Gvgmd8yxscKvBkBiGp3J3g",
      "exclusive": false,
      "prefetch_count": 0,
      "queue": {
        "name": "soaSyncPwdQueue",
        "vhost": "/"
      }
    }
  ],
  "arguments": {
    "x-dead-letter-exchange": "",
    "x-dead-letter-routing-key": "DL_soaSyncPwdQueue"
  },
  "auto_delete": false,
  "backing_queue_status": {
    "avg_ack_egress_rate": 0.00021645676867156752,
    "avg_ack_ingress_rate": 0.00021645676867156752,
    "avg_egress_rate": 0.00021645676867156752,
    "avg_ingress_rate": 0.00021645676867156752,
    "delta": [
      "delta",
      "undefined",
      0,
      0,
      "undefined"
    ],
    "len": 0,
    "mode": "default",
    "next_seq_id": 2177,
    "q1": 0,
    "q2": 0,
    "q3": 0,
    "q4": 0,
    "target_ram_count": "infinity"
  },
  "consumer_utilisation": null,
  "consumers": 1,
  "deliveries": [
    
  ],
  "durable": true,
  "effective_policy_definition": [
    
  ],
  "exclusive": false,
  "exclusive_consumer_tag": null,
  "garbage_collection": {
    "fullsweep_after": 65535,
    "max_heap_size": 0,
    "min_bin_vheap_size": 46422,
    "min_heap_size": 233,
    "minor_gcs": 294
  },
  "head_message_timestamp": null,
  "idle_since": "2019-05-29 7:34:01",
  "incoming": [
    
  ],
  "memory": 18808,
  "message_bytes": 0,
  "message_bytes_paged_out": 0,
  "message_bytes_persistent": 0,
  "message_bytes_ram": 0,
  "message_bytes_ready": 0,
  "message_bytes_unacknowledged": 0,
  "message_stats": {
    "ack": 2007,
    "ack_details": {
      "rate": 0.0
    },
    "deliver": 2181,
    "deliver_details": {
      "rate": 0.0
    },
    "deliver_get": 2181,
    "deliver_get_details": {
      "rate": 0.0
    },
    "deliver_no_ack": 0,
    "deliver_no_ack_details": {
      "rate": 0.0
    },
    "get": 0,
    "get_details": {
      "rate": 0.0
    },
    "get_no_ack": 0,
    "get_no_ack_details": {
      "rate": 0.0
    },
    "publish": 2177,
    "publish_details": {
      "rate": 0.0
    },
    "redeliver": 4,
    "redeliver_details": {
      "rate": 0.0
    }
  },
  "messages": 0,
  "messages_details": {
    "rate": 0.0
  },
  "messages_paged_out": 0,
  "messages_persistent": 0,
  "messages_ram": 0,
  "messages_ready": 0, //消息未消费
  "messages_ready_details": {
    "rate": 0.0
  },
  "messages_ready_ram": 0,
  "messages_unacknowledged": 0, //正在消费
  "messages_unacknowledged_details": {
    "rate": 0.0
  },
  "messages_unacknowledged_ram": 0,
  "name": "soaSyncPwdQueue",
  "node": "rabbit@CNQLS03247",
  "operator_policy": null,
  "policy": null,
  "recoverable_slaves": null,
  "reductions": 23652972,
  "reductions_details": {
    "rate": 0.0
  },
  "state": "running",
  "vhost": "/"
}
```

注意：
虚拟主机名Virtual host在设置的时候不要带/，不然会访问不到
```
{"error":"Object Not Found","reason":"\"Not Found\"\n"}
```
之前就是被这个坑了好久，明明按照API写的格式来的，就是访问不到。
java代码：
```
jdk 1.8
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.Base64;

public class Test {
    public static void main(String[] args) throws Exception {
        String host = "";
        String port = "";
        String virtualHost = "";
        String queueName = "";
        // 发送一个GET请求
        HttpURLConnection httpConn = null;
        BufferedReader in = null;

        String urlString = "http://" + host + ":" + port + "/api/queues/" + virtualHost + "/" + queueName;
        // urlString = "http://" + host + ":" + port + "/api/queues/";
        URL url = new URL(urlString);
        httpConn = (HttpURLConnection) url.openConnection();
        // 设置用户名密码
        String user="";
        String password="";
        String auth = user + ":" + password;

        String encoding = new String(Base64.getEncoder().encode(auth.getBytes()));
        httpConn.setDoOutput(true);
        httpConn.setRequestProperty("Authorization", "Basic " + encoding);
        // 建立实际的连接
        httpConn.connect();
        // 读取响应
        if (httpConn.getResponseCode() == HttpURLConnection.HTTP_OK) {
            StringBuilder content = new StringBuilder();
            String tempStr = "";
            in = new BufferedReader(new InputStreamReader(httpConn.getInputStream()));
            while ((tempStr = in.readLine()) != null) {
                content.append(tempStr);
            }
            in.close();
            httpConn.disconnect();
            System.out.println(content.toString());
        } else {
            System.out.println(httpConn.getResponseCode());
            httpConn.disconnect();
        }
    }
}

```