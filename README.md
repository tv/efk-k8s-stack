# EFK 7.0.0 OpenSource stack for Kubernetes

This stack collects all of the logs from the cluster to elasticsearch to be displayed in kibana. All of them are self hosted inside kubernetes.

Elasticsearch is clustered with 3 nodes with stateful sets and persistent volume claims inside kubernetes. All 3 nodes are masters, data and ingest nodes at the same time. This could be optimized, but for most logging needs it should be enough. Memory/CPU requests should be increased to at least m5.large sizes (2core/8GB) or so that all (or atleast most) of the logs from stored days fit in to the memory.

Kibana is managed by deployment and in this case has only one replica which could/should be increased to ensure high availability in real world. It also has it's own coordinator elasticsearch to ensure efficient loadbalancing of queries.

Filebeat is used as a log shipper. It's pretty lightweight compared to logstash, in bot memory footprint and in filter capabilities. But it supports kubernetes metadata and json parsing which are pretty much essential for kubernetes logging. For basic logging needs it looks to be enough. For more advanced use case (audit log parsing or so) I would switch that to fluentd or logstash. I also looked up fluent-bit, but it's also quite lightweight in the parsing side of things. Filebeat was chosen primarily instead of fluent-bit to try "full" elastic cloud experience, which wasn't really a thing without X-Pack additions.

Curator is used for log retentions. It's setup as cronjob, but unfortunately it does not yet support es 7.0. I left the manifest in there, but it won't work until next curator is released. ref: https://github.com/elastic/curator/issues/1380

While writing this, I noticed that basic X-Pack elastic cloud additions are now available without license. They are however not included in the open source image versions I use here. X-Pack would make it more viable for retention, authentication, SSL and so on. That should be investigated further.

## Running

For docker-for-mac kubernetes just run all of the yamls. Ensure that you have enough memory dedicated to the docker machine (>3GB, tested with 8GB). This will use hostpath storage provisioner and create localhost:9200/localhost:5601 loadbalancer services to access the es/kibana.

For real kubernetes, you would need to either change pvc storage class name for elasticsearch to one that is already provided and add means to access kibana service from outside world. Preferably with authentication front proxy and ingress. 
Of course all of the resource requests/limits and PVC sizes are way too low for real world usage, so these should be increased according the amount of logs, retention days, queries per second and so on. Dedicated kubernetes nodes would be advised for elasticsearch, for example m5.large or m5.xlarge in AWS.
