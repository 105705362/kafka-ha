# BOSH release for kafka

This BOSH release and deployment manifest deploy a cluster of kafka.

## Usage

This repository includes base manifests and operator files. They can be used for initial deployments and subsequently used for updating your deployments:

```plain
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=kafka
bosh deploy kafka-boshrelease/manifests/kafka-src.yml 
bosh deploy kafka-boshrelease/manifests/kafka-dest.yml 

```

If you are deploying this in intranet (without internet access or blocked by GFW, please try to download required packages indicated in manifest.

You might also have to upload corresponding stemcell which is used in this release. Please find the version in manifest and try to download and upload appropriately.



### Topics

You can pre-define some simple topics using an operator script `./manifests/operators/simple-topics.sh`. Th

```plain
bosh deploy kafka-boshrelease/manifests/kafka.yml \
  -o <(kafka-boshrelease/manifests/operators/pick-from-cloud-config.sh kafka-boshrelease/manifests/kafka.yml) \
  -o <(kafka-boshrelease/manifests/operators/simple-topics.sh test1 test2)
```

But I prefer to do it manually, please try to download and install confluent toolkits (https://www.confluent.io/)
please find corresponding commands to create, consume and produce messages on kafka service. There are some examples as follows:

```
kafka-topics.sh --create \
--topic test.replica \
--zookeeper zk-0.zk-svc.default.svc.cluster.local:2181,zk-1.zk-svc.default.svc.cluster.local:2181,zk-2.zk-svc.default.svc.cluster.local:2181 \
--partitions 3 \
--replication-factor 2
```

```

kafka-console-consumer.sh --topic test --bootstrap-server localhost:9093
kafka-console-consumer.sh --topic test --bootstrap-server kafka-svc.default.svc.cluster.local:9093

cd nge/binary/confluent/bin
~/nge/binary/confluent/bin/kafka-mirror-maker --consumer.config ../conf/consumer-B.properties --producer.config ../conf/producer-B.properties --whitelist test-B
~/nge/binary/confluent/bin/kafka-mirror-maker --consumer.config ../conf/consumer-A.properties --producer.config ../conf/producer-A.properties --whitelist test-A
~/nge/binary/confluent/bin/kafka-console-consumer --topic test --bootstrap-server 10.34.54.176:9092
~/nge/binary/confluent/bin/kafka-console-consumer --topic test --bootstrap-server 10.34.54.168:9092
```

### Kafka Manager

![kafka-manager](https://github.com/cloudfoundry-community/kafka-boshrelease/raw/master/doc/kafka-manager.png)

The [Yahoo Kafka Manager](https://github.com/yahoo/kafka-manager) UI is installed on each Kafka node. You can access it via port 8080. To access via http://localhost:8080, open a tunnel:

```plain
bosh ssh kafka-manager/0 -- -L 8080:127.0.0.1:8080
```

Kafka Manager requires basic auth credentials. The default `username` is `admin`, and the `password` is the `((kafka-manager-password))` value from either Credhub/Config Server, or your `--vars-store creds.yml` file.

### Update

When new versions of `kafka-boshrelease` are released the `manifests/kafka.yml` file will be updated. This means you can easily `git pull` and `bosh deploy` to upgrade.

```plain
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=kafka
cd kafka-boshrelease
git pull
cd -
bosh deploy kafka-boshrelease/manifests/kafka.yml
```

### Development

To iterate on this BOSH release, use the `create.yml` manifest when you deploy:

```plain
bosh deploy manifests/kafka.yml -o manifests/operators/create.yml
```
