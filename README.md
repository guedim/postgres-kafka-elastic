# Postgres-Kafka-Elastic

This is a simple sample to get the data from a [Postgres](https://www.postgresql.org/) database,  read the data into [Kafka](https://kafka.apache.org/) and finally put in Elastick [Elastic Search](https://www.elastic.co/)

> This is a simple project using docker containers.

### Configuration

1) Open a web browser and go to [Play With Docker](play-with-docker.com) tool:
```sh
https://play-with-docker.com
```
2) Create one instache, however to avoid performance issues we recommend you to create a swarm cluster using the [PWD](play-with-docker.com) templates  (3 Managers and 2 Workers  or 5 Managers and no workers).

3) Download the [docker-compose](https://docs.docker.com/compose/) file in the new instance created in the above step:
```sh
wget https://raw.githubusercontent.com/guedim/postgres-kafka-elastic/master/docker-compose.yml
```

4) Start the services ([Postgres](https://www.postgresql.org/) - [Kafka](https://kafka.apache.org/) - Elastick [Elastic Search](https://www.elastic.co/)) in a [Swarm Mode](https://docs.docker.com/engine/swarm/):
```sh
docker stack deploy --compose-file docker-compose.yml postgres-kafka-es
```
5) Go to [Landoop](http://www.landoop.com/) portal (clic in 3030 port), for example:

```sh
http://pwd10-0-28-3-3030.host2.labs.play-with-docker.com/
```
6) In the [Landoop](http://www.landoop.com/) portal, create and set up the Postgres Kafka  using the [JDBC](http://docs.confluent.io/current/connect/connect-jdbc/docs/index.html) connector:
```sh
{
  "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
  "mode": "timestamp+incrementing",
  "timestamp.column.name": "updated_at",
  "incrementing.column.name": "id",
  "topic.prefix": "postgres_",
  "tasks.max": "1",
  "name": "source-postgres",
  "connection.url": "jdbc:postgresql://192.168.99.100:5432/postgres?user=postgres&password=postgres"
}
```
7) In the [Landoop](http://www.landoop.com/) portal, create and set up the [kafka elastic Sink](http://docs.confluent.io/current/connect/connect-elasticsearch/docs/elasticsearch_connector.html):
```sh
{
  "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
  "type.name": "kafka-connect",
  "topics": "postgres_users",
  "tasks.max": "1",
  "topic.key.ignore": "true",
  "name": "ElasticsearchSinkConnector",
  "connection.url": "http://192.168.99.100:9200",
  "key.ignore": "true",
  "value.converter": "org.apache.kafka.connect.json.JsonConverter",
  "key.converter": "org.apache.kafka.connect.json.JsonConverter",
  "topic.schema.ignore": "true"
}
```
8) Connect to postgres service and create the table **users** and insert some sample data:
```sql
-- Create the table users
CREATE TABLE users
(
    id          SERIAL NOT NULL,
    name        VARCHAR(100) NOT NULL,
    age         INTEGER,
    updated_at  TIMESTAMP NOT NULL DEFAULT NOW(),
    PRIMARY KEY(id)
);

-- Insert some data:
INSERT INTO users (name, age) VALUES ('john', 26);
INSERT INTO users (name, age) VALUES ('jane', 24);
INSERT INTO users (name, age) VALUES ('julia', 25);
INSERT INTO users (name, age) VALUES ('jamie', 22);
INSERT INTO users (name, age) VALUES ('jenny', 27);
```

9) Finally, you can query the Postgres data in the ElasticSearch tool, just click in the port 9200 and go to **_plugin/dejavu** (dont forget to use the postgres_users index):
```sh
http://192.168.99.100:9200/_plugin/dejavu
```

### Todos

 - How to create the sql data
 - Create all the data and configuration automatically

License
----

MIT
