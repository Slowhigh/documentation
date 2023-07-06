---
title: Node Quickstart
description: A quickstart on how to use the Node client library
---

## Installation

```sh
$ npm install memphis-dev
```

## Importing

For Javascript, you can choose to use the import or required keyword. This library exports a singleton instance of `memphis` with which you can consume and produce messages.

```js
const { memphis } = require('memphis-dev');
```

For Typescript, use the import keyword. You can import `Memphis` to aid for typechecking assistance.

```js
import { memphis, Memphis } from 'memphis-dev';
```

To leverage Nestjs dependency injection feature

```js
import { Module } from '@nestjs/common';
import { Memphis, MemphisModule, MemphisService } from 'memphis-dev';
```

### Connecting to Memphis

First, we need to connect with Memphis by using `memphis.connect`.

```js
/* Javascript and typescript project */
await memphis.connect({
            host: "<memphis-host>",
            port: <port>, // defaults to 6666
            username: "<username>", // (root/application type user)
            accountId: <accountId> //You can find it on the profile page in the Memphis UI. This field should be sent only on the cloud version of Memphis, otherwise it will be ignored
            connectionToken: "<broker-token>", // you will get it on application type user creation
            password: "<string>", // depends on how Memphis deployed - default is connection token-based authentication
            reconnect: true, // defaults to true
            maxReconnect: 3, // defaults to 3
            reconnectIntervalMs: 1500, // defaults to 1500
            timeoutMs: 1500, // defaults to 1500
            // for TLS connection:
            keyFile: '<key-client.pem>',
            certFile: '<cert-client.pem>',
            caFile: '<rootCA.pem>'
            suppressLogs: false // defaults to false - indicates whether to suppress logs or not
      });
```

Nest injection

```js
@Module({
    imports: [MemphisModule.register()],
})

class ConsumerModule {
    constructor(private memphis: MemphisService) {}

    startConnection() {
        (async function () {
            let memphisConnection: Memphis;

            try {
               memphisConnection = await this.memphis.connect({
                    host: "<memphis-host>",
                    username: "<application type username>",
                    connectionToken: "<broker-token>",
                });
            } catch (ex) {
                console.log(ex);
                memphisConnection.close();
            }
        })();
    }
}
```

Once connected, the entire functionalities offered by Memphis are available.

### Disconnecting from Memphis

To disconnect from Memphis, call `close()` on the memphis object.

```js
memphisConnection.close();
```

### Creating a Station
**Unexist stations will be created automatically through the SDK on the first producer/consumer connection with default values.**<br><br>
_If a station already exists nothing happens, the new configuration will not be applied_

```js
const station = await memphis.station({
    name: '<station-name>',
    schemaName: '<schema-name>',
    retentionType: memphis.retentionTypes.MAX_MESSAGE_AGE_SECONDS, // defaults to memphis.retentionTypes.MAX_MESSAGE_AGE_SECONDS
    retentionValue: 604800, // defaults to 604800
    storageType: memphis.storageTypes.DISK, // defaults to memphis.storageTypes.DISK
    replicas: 1, // defaults to 1
    idempotencyWindowMs: 0, // defaults to 120000
    sendPoisonMsgToDls: true, // defaults to true
    sendSchemaFailedMsgToDls: true, // defaults to true
    tieredStorageEnabled: false // defaults to false
});
```

Creating a station with Nestjs dependency injection

```js
@Module({
    imports: [MemphisModule.register()],
})

class stationModule {
    constructor(private memphis: MemphisService) { }

    createStation() {
        (async function () {
                  const station = await this.memphis.station({
                        name: "<station-name>",
                        schemaName: "<schema-name>",
                        retentionType: memphis.retentionTypes.MAX_MESSAGE_AGE_SECONDS, // defaults to memphis.retentionTypes.MAX_MESSAGE_AGE_SECONDS
                        retentionValue: 604800, // defaults to 604800
                        storageType: memphis.storageTypes.DISK, // defaults to memphis.storageTypes.DISK
                        replicas: 1, // defaults to 1
                        idempotencyWindowMs: 0, // defaults to 120000
                        sendPoisonMsgToDls: true, // defaults to true
                        sendSchemaFailedMsgToDls: true, // defaults to true
                        tieredStorageEnabled: false // defaults to false
                  });
        })();
    }
}
```

### Retention types

Memphis currently supports the following types of retention:

```js
memphis.retentionTypes.MAX_MESSAGE_AGE_SECONDS;
```

Means that every message persists for the value set in retention value field (in seconds)

```js
memphis.retentionTypes.MESSAGES;
```

Means that after max amount of saved messages (set in retention value), the oldest messages will be deleted

```js
memphis.retentionTypes.BYTES;
```

Means that after max amount of saved bytes (set in retention value), the oldest messages will be deleted


### Retention Values

The `retention values` are directly related to the `retention types` mentioned above, where the values vary according to the type of retention chosen.

All retention values are of type `int` but with different representations as follows:

`memphis.retentionTypes.MAX_MESSAGE_AGE_SECONDS` is represented **in seconds**, `memphis.retentionTypes.MESSAGES` in a **number of messages** and finally `memphis.retentionTypes.BYTES` in a **number of bytes**.

After these limits are reached oldest messages will be deleted.

### Storage types

Memphis currently supports the following types of messages storage:

```js
memphis.storageTypes.DISK;
```

Means that messages persist on disk

```js
memphis.storageTypes.MEMORY;
```

Means that messages persist on the main memory

### Destroying a Station

Destroying a station will remove all its resources (producers/consumers)

```js
await station.destroy();
```

### Creating a new schema 

```js
await memphisConnection.createSchema({schemaName: "<schema-name>", schemaType: "<schema-type>", schemaFilePath: "<schema-file-path>" });
```

### Enforcing a schema on an existing Station

```js
await memphisConnection.enforceSchema({ name: '<schema-name>', stationName: '<station-name>' });
```

### Deprecated - Use enforceSchema instead

```js
await memphisConnection.attachSchema({ name: '<schema-name>', stationName: '<station-name>' });
```

### Detaching a schema from Station

```js
await memphisConnection.detachSchema({ stationName: '<station-name>' });
```

### Produce and Consume messages

The most common client operations are `produce` to send messages and `consume` to
receive messages.

Messages are published to a station and consumed from it by creating a consumer.
Consumers are pull based and consume all the messages in a station unless you are using a consumers group, in this case messages are spread across all members in this group.

Memphis messages are payload agnostic. Payloads are `Uint8Arrays`.

In order to stop getting messages, you have to call `consumer.destroy()`. Destroy will terminate regardless
of whether there are messages in flight for the client.

### Creating a Producer

```js
const producer = await memphisConnection.producer({
    stationName: '<station-name>',
    producerName: '<producer-name>',
    genUniqueSuffix: false // defaults to false
});
```

Creating producers with nestjs dependecy injection

```js
@Module({
    imports: [MemphisModule.register()],
})

class ProducerModule {
    constructor(private memphis: MemphisService) { }

    createProducer() {
        (async function () {
                const producer = await memphisConnection.producer({
                    stationName: "<station-name>",
                    producerName: "<producer-name>"
                });
        })();
    }
}
```

### Producing a message

Without creating a producer.
In cases where extra performance is needed the recommended way is to create a producer first
and produce messages by using the produce function of it

```js
await memphisConnection.produce({
        stationName: '<station-name>',
        producerName: '<producer-name>',
        genUniqueSuffix: false, // defaults to false
        message: 'Uint8Arrays/object/string/DocumentNode graphql', // Uint8Arrays/object (schema validated station - protobuf) or Uint8Arrays/object (schema validated station - json schema) or Uint8Arrays/string/DocumentNode graphql (schema validated station - graphql schema)
        ackWaitSec: 15, // defaults to 15
        asyncProduce: true // defaults to false
        headers: headers, // defults to empty
        msgId: 'id' // defaults to null
});
```

Creating a producer first

```js
await producer.produce({
    message: 'Uint8Arrays/object/string/DocumentNode graphql', // Uint8Arrays/object (schema validated station - protobuf) or Uint8Arrays/object (schema validated station - json schema) or Uint8Arrays/string/DocumentNode graphql (schema validated station - graphql schema)
    ackWaitSec: 15 // defaults to 15
});
```

### Add Header

```js
const headers = memphis.headers();
headers.add('<key>', '<value>');
await producer.produce({
    message: 'Uint8Arrays/object/string/DocumentNode graphql', // Uint8Arrays/object (schema validated station - protobuf) or Uint8Arrays/object (schema validated station - json schema) or Uint8Arrays/string/DocumentNode graphql (schema validated station - graphql schema)
    headers: headers // defults to empty
});
```

or

```js
const headers = { key: 'value' };
await producer.produce({
    message: 'Uint8Arrays/object/string/DocumentNode graphql', // Uint8Arrays/object (schema validated station - protobuf) or Uint8Arrays/object (schema validated station - json schema) or Uint8Arrays/string/DocumentNode graphql (schema validated station - graphql schema)
    headers: headers
});
```

### Async produce

Meaning your application won't wait for broker acknowledgement - use only in case you are tolerant for data loss

```js
await producer.produce({
    message: 'Uint8Arrays/object/string/DocumentNode graphql', // Uint8Arrays/object (schema validated station - protobuf) or Uint8Arrays/object (schema validated station - json schema) or Uint8Arrays/string/DocumentNode graphql (schema validated station - graphql schema)
    ackWaitSec: 15, // defaults to 15
    asyncProduce: true // defaults to false
});
```

### Message ID

Stations are idempotent by default for 2 minutes (can be configured), Idempotency achieved by adding a message id

```js
await producer.produce({
    message: 'Uint8Arrays/object/string/DocumentNode graphql', // Uint8Arrays/object (schema validated station - protobuf) or Uint8Arrays/object (schema validated station - json schema) or Uint8Arrays/string/DocumentNode graphql (schema validated station - graphql schema)
    ackWaitSec: 15, // defaults to 15
    msgId: 'id' // defaults to null
});
```

### Destroying a Producer

```js
await producer.destroy();
```

### Creating a Consumer

```js
const consumer = await memphisConnection.consumer({
    stationName: '<station-name>',
    consumerName: '<consumer-name>',
    consumerGroup: '<group-name>', // defaults to the consumer name.
    pullIntervalMs: 1000, // defaults to 1000
    batchSize: 10, // defaults to 10
    batchMaxTimeToWaitMs: 5000, // defaults to 5000
    maxAckTimeMs: 30000, // defaults to 30000
    maxMsgDeliveries: 10, // defaults to 10
    genUniqueSuffix: false, // defaults to false
    startConsumeFromSequence: 1, // start consuming from a specific sequence. defaults to 1
    lastMessages: -1 // consume the last N messages, defaults to -1 (all messages in the station)
});
```

### Passing context to message handlers

```js
consumer.setContext({ key: 'value' });
```

### Processing messages

```js
consumer.on('message', (message, context) => {
    // processing
    console.log(message.getData());
    message.ack();
});
```

### Fetch a single batch of messages

```js
const msgs = await memphis.fetchMessages({
    stationName: '<station-name>',
    consumerName: '<consumer-name>',
    consumerGroup: '<group-name>', // defaults to the consumer name.
    batchSize: 10, // defaults to 10
    batchMaxTimeToWaitMs: 5000, // defaults to 5000
    maxAckTimeMs: 30000, // defaults to 30000
    maxMsgDeliveries: 10, // defaults to 10
    genUniqueSuffix: false, // defaults to false
    startConsumeFromSequence: 1, // start consuming from a specific sequence. defaults to 1
    lastMessages: -1 // consume the last N messages, defaults to -1 (all messages in the station)
});
```

### Fetch a single batch of messages after creating a consumer

```js
const msgs = await consumer.fetch({
    batchSize: 10, // defaults to 10
});
```

To set Up connection in nestjs

```js
import { MemphisServer } from 'memphis-dev'

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      strategy: new MemphisServer({
        host: '<memphis-host>',
        username: '<application type username>',
        connectionToken: '<broker-token>'
      }),
    },
  );

  await app.listen();
}
bootstrap();
```

To consume messages in NestJS

```js
export class Controller {
    import { MemphisConsume, Message } from 'memphis-dev';

    @MemphisConsume({
        stationName: '<station-name>',
        consumerName: '<consumer-name>',
        consumerGroup: ''
    })
    async messageHandler(message: Message) {
        console.log(message.getData().toString());
        message.ack();
    }
}
```

### Acknowledge a message

Acknowledge a message indicates the Memphis server to not re-send the same message again to the same consumer / consumers group

```js
message.ack();
```

### Delay and resend the message after a given duration

Delay the message and tell Memphis server to re-send the same message again to the same consumer group. The message will be redelivered only in case `Consumer.maxMsgDeliveries` is not reached yet.

```js
message.delay(delayInMilliseconds);
```

### Get message payload

As Uint8Array

```js
msg = message.getData();
```

As Json

```js
msg = message.getDataAsJson();
```

### Get headers

Get headers per message

```js
headers = message.getHeaders();
```

### Get message sequence number

Get message sequence number

```js
sequenceNumber = message.getSequenceNumber();
```

### Catching async errors

```js
consumer.on('error', (error) => {
    // error handling
});
```

### Destroying a Consumer

```js
await consumer.destroy();
```

### Check if broker is connected

```js
memphisConnection.isConnected();
```
