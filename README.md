## Confluent Schema Registry

### Summary

Many of the systems I have worked on have included Kafka and therefore 
Confluent Schema Registry. This Playground isolates the service so it can be 
explored locally. This is intended for learning purposes only. Note, it doesn't 
run Kafka at all, just the schema registry and a required broker. Also note, 
this does not run Zookeeper, instead runs in the new Kraft mode. 

The Confluent Schema Registry is a vital piece of the Kafka eco-system. It is 
used to persist the various schemas and their versions of the events that are 
published and consumed via Kafka. The registry service exposes an API over http.

Schemas can be defined in various formats:
 - JSON
 - AVRO
 - Protobuf

The point of this playground is to learn about creating Schemas and their 
Versions, along with exploring the various Compatibility modes.

Compatibility modes:
	- *Backward (recommended)* — receiver can read both current and previous versions.
	- *Backward All* — receiver can read current and all previous versions.
	- *Forward* — sender can write both current and previous versions.
	- *Forward All* — sender can write current and all previous versions.
	- *Full* — combination of Backward and Forward.
	- *Full All* — combination of Backward All and Forward All.
	- *None* — no compatibility checks performed.
	- *Disabled* — prevent any versioning for this schema.

### Installation

For this playground, we are going to run the registry via Docker.

Confluent provides a docker-compose file at: https://raw.githubusercontent.com/confluentinc/cp-all-in-one/7.5.3-post/cp-all-in-one-kraft/docker-compose.yml

In this repo is a slimmed down docker-compose for just running schema registry.

`docker compose up -d`

To confirm that schema registry is up, try requesting the subjects: `curl localhost:8081`

That should return an empty collection in JSON: `{}`

### View Schema types

`curl --silent -X GET http://localhost:8081/schemas/types`

### View Schema types currently registered:

`curl -X GET http://localhost:8081/schemas/types`

### Viewing Subjects

`curl --silent -X GET http://localhost:8081/subjects/ | jq .`

### Viewing Schemas

`curl --silent -X GET http://localhost:8081/schemas/ | jq .`

### Creating a JSON Schema

put the following in schema.json file:
`
{
	"schema" : {
		"type":"record",
		"name":"Payment",
		"namespace":"my.examples",
		"fields":[
			{
				"name":"id",
				"type":"string"
			},
			{
				"name":"amount",
				"type":"double"
			}
		]
	}
}
`

We have to specify the schemaType to have a value of JSON, hence the use of jq here for adding that attribute and serialization, 
note the subject is called 'payments-value' and will be used for other examples:

`jq '. | {schema: tojson, schemaType: "JSON"}' schema.json | curl -X POST "http://localhost:8081/subjects/payments-value/versions" -H "Content-Type:application/json" -d @-`

### Requesting a Schema

Substitute the ID number:

`curl -X GET http://localhost:8081/schemas/ids/2 | jq '.'`

### Creating a version of a schema

Make a change to the schema.json file, like adding a third field, and use the same POST command as when initially creating the schema, for the same subject:

`jq '. | {schema: tojson, schemaType: "JSON"}' schema.json | curl -X POST "http://localhost:8081/subjects/payments-value/versions" -H "Content-Type:application/json" -d @-`

### List all versions of a schema

`curl -X GET http://localhost:8081/subjects/payments-value/versions`

### Request a specific version of a schema

`curl -X GET http://localhost:8081/subjects/payments-value/versions/2 | jq '.'`

### Delete version 1 of a specific schema:

`curl -X DELETE http://localhost:8081/subjects/payments-value/versions/1`

### Delete all versions of a schema:

`curl -X DELETE http://localhost:8081/subjects/payments-value`

### Test compatibility of data against a schema

`curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  --data '{"schema": "{\"type\": \"string\"}"}' \
  http://localhost:8081/compatibility/subjects/payments-value/versions/latest`

### Get the top level config

`curl -X GET http://localhost:8081/config`

### Get the compatibility requirements for a subject

`curl -X GET http://localhost:8081/config/payments-value`

### Update the compatibility requirements for a subject

`curl -X PUT -H "Content-Type: application/vnd.schemaregistry.v1+json" --data '{"compatibility": "FULL"}' http://localhost:8081/configpayments-value`

