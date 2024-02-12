# Example: Postgres > Debezium > Kafka > Redshift

An example using Confluent Kafka connectors to stream data from a Postgres database to a Redshift database.

[See comments in this PR](https://github.com/troyharvey/debezium-kafka-redshift-sink-example/pull/1) for more notes on this setup.

1. Start the services in Docker Compose.

        docker compose up

1. Connect to the Postgres database and create a test table.

        docker exec -it postgres psql -U postgres

        CREATE SEQUENCE IF NOT EXISTS test_table_id_seq;
        CREATE TABLE "public"."test_table" (
            "id" int4 NOT NULL DEFAULT nextval('test_table_id_seq'::regclass),
            "first_name" varchar NOT NULL,
            "last_name" varchar NOT NULL,
            PRIMARY KEY ("id")
        );
        INSERT INTO test_table(first_name, last_name) VALUES('herp', 'derperson');

1. Register the Postgres cdc Kafka connector.

        curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-postgres.json

1. Fill in your nonprod Redshift credentials in the `register-redshift.json` file.

        cp register-redshift.json.example register-redshift.json

1. Register the Redshift cdc Kafka connector.

        curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-redshift.json

1. Insert a record. It may take a few minutes for the record to propagate through to Redshift. Tail the logs in the `connect` container to see if there are any errors.

        INSERT INTO test_table(first_name, last_name) VALUES('derp', 'herperson');

1. Query the Redshift table to see if the record was inserted.

        select * from public.test_table;

## References

* [PostgreSQL Source (Debezium) Connector Configuration Properties](https://docs.confluent.io/kafka-connectors/debezium-postgres-source/current/postgres_source_connector_config.html#postgres-source-connector-config)
* [Debezium Topic Routing](https://debezium.io/documentation/reference/stable/transformations/topic-routing.html#topic-routing)
* [Amazon Redshift Sink Connector Configuration Properties](https://docs.confluent.io/kafka-connectors/aws-redshift/current/sink_config_options.html#redshift-sink-config-options)
* [Lessons Learned from Running Debezium with PostgreSQL on Amazon RDS](https://debezium.io/blog/2020/02/25/lessons-learned-running-debezium-with-postgresql-on-rds/)
* [Streaming data to a downstream database](https://debezium.io/blog/2017/09/25/streaming-to-another-database/)
* [Kafka Connect Debezium Mysql source/sink replication pipeline](https://medium.com/@alexander.murylev/kafka-connect-debezium-mysql-source-sink-replication-pipeline-fb4d7e9df790)
* [jyablonski/debezium-kafka-demo](https://github.com/jyablonski/debezium-kafka-demo)
* [From Zero to Hero with Kafka Connect](https://github.com/confluentinc/demo-scene/tree/master/kafka-connect-zero-to-hero)
* [Confluent Platform All-In-One](https://github.com/confluentinc/cp-all-in-one)
* [Run Debezium Kafka Connect using Docker | Kafka | Zookeeper | Kafdrop |Kafka Connect | Docker Compose](https://medium.com/@cloud_geek/run-debezium-kafka-connect-using-docker-kafka-zookeeper-kafdrop-kafka-connect-docker-2e67760ef85d)
* [Docker Compose For Your Next Debezium And Postgres Project](https://www.iamninad.com/posts/docker-compose-for-your-next-debezium-and-postgres-project/)
* [dariocazas/howto-debezium-to-snowflake](https://github.com/dariocazas/howto-debezium-to-snowflake)
* [Building an Enterprise CDC Solution](https://dzone.com/articles/howto_building-an-enterprise-cdc-solution)
* [Change Data Capture Magic: Streaming with Debezium, Kafka, and Docker](https://medium.com/yazilim-vip/change-data-capture-magic-streaming-with-debezium-kafka-and-docker-fa31328ef14e)
* [Practical Notes in Change Data Capture with Debezium and Postgres](https://medium.com/cermati-tech/practical-notes-in-change-data-capture-with-debezium-and-postgres-fe31bb11ab78)
* [Postgres WAL Files and Sequence Numbers](https://www.crunchydata.com/blog/postgres-wal-files-and-sequuence-numbers#)
