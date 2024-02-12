# Example: Postgres > Debezium > Kafka > Redshift

An example using Confluent Kafka connectors to stream data from a Postgres database to a Redshift database.

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
