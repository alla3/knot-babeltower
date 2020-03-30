# Babeltower Events API - v2.0.0

This document describes the events `babeltower` is able to receive and send. They are gruped based on the external clients point of view, i.e. publishing or subscribing to the topics. In each section, it is provided information about the header, payload and protocol binding details of the event.

## Content

- [Publish](#publish) (external clients can publish to):
  - [device.register](#device-register)
  - [device.unregister](#device-unregister)
  - [device.schema.sent](#device-schema-sent)
  - [data.sent](#data-sent)
  - [data.request](#data-request)
  - [data.update](#data-update)
  - [device.list](#device-list)
  - [device.auth](#device-auth)

- [Subscribe](#Subscribe) (external clients can subscribe to):
  - [device.registered](#device-registered)
  - [device.unregistered](#device-unregistered)
  - [device.schema.updated](#device-schema-updated)
  - [data.published](#data-published)
  - [device.[id].data.request](#device-<id>-data-request)
  - [device.[id].data.update](#device-<id>-data-update)

-----------------------------------------------------------------

## Publish

This section describes the events that this service can receive from the external applications.

### **device.register** <a name="device-register"></a>

Event-command to register a new thing on the things registry. The operation response is sent through [`device.registered`](#device-registered) event.

<details>
  <summary>Headers</summary>

  - `token` **String** user's token

</details>

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `name` **String** thing's name

  Example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "name": "KNoT Thing"
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - AMQP:
    - Exchange:
      - Type: direct
      - Name: device
      - Durable: `true`
      - Auto-delete: `false`
    - Routing key: device.register

</details>

### **device.unregister** <a name="device-unregister"></a>

Event-command to remove a thing from the things registry. The operation response is sent through [`device.unregistered`](#device-registered) event.

<details>
  <summary>Headers</summary>

  - `token` **String** user's token

</details>

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID

  Example:

  ```json
  {
    "id": "fbe64efa6c7f717e"
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing key: device.unregister

</details>

### **device.list** <a name="device-list"></a>

Event-command to list the registered things. It follows the request/reply pattern. After obtaining the things, `babeltower` will send a reply message by using the `correlation_id` property, which wass received in the request header, as reply message's `routing_key`. Because of that, considering the **requestor** has created and sent this `correlation_id` in the request, it can also subscribe to receive events that arrive with a `routing_key` equivalent to the `correlation_id`. Therefore, the reply is received by the application that has sent the request, in a **one-to-one** manner.

<details>
  <summary>Headers</summary>

  - `token` **String** user's token
  - `correlation_id` **String** ID to correlate request/reply
  - `reply_to` **String** queue that will process the reply

</details>


<details>
  <summary>Payload</summary>

  JSON in the following format:

  - Empty object

  Example:

  ```json
  {}
  ```

</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing key: `correlation_id`

</details>

### **device.auth** <a name="device-auth"></a>

Event-command to verify if a thing is authenticated based on its credentials. It follows the request/reply pattern. After authenticating the device, `babeltower` will send a reply message by using the `correlation_id` property, which is received in the request header, as reply message's `routing_key`. Because of that, considering the **requestor** has created and sent this `correlation_id` in the request, it can also subscribe to receive events that arrive with a `routing_key` equivalent to the `correlation_id`. Therefore, the reply is received by the application that has sent the request, in a **one-to-one** manner.

<details>
  <summary>Headers</summary>

  - `token` **String** user's token
  - `correlation_id` **String** ID to correlate reply/response
  - `reply_to` **String** queue that will process the reply

</details>

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** device's ID
  - `token` **String** device's token

  Example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "token": "0c20c12e2ac058d0513d81dc58e33b2f9ff8c83d"
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing key: `correlation_id`

</details>

### **device.schema.sent** <a name="device-schema-sent"></a>

Event that represents a device sending its schema to the services that are interested. After receiving this event, `babeltower` updates the thing's schema on the registry and send a [`device.schema.updated`](#device-schema-updated) event.

<details>
  <summary>Headers</summary>

  - `token` **String** user's token

</details>

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `schema` **Array** schema items, each one formed by:
    - `sensorId` **Number** sensor ID
    - `valueType` **Number** semantic value type (voltage, current, temperature, etc)
    - `unit` **Number** sensor unit (V, A, W, W, etc)
    - `typeId` **Number** data value type (boolean, integer, etc)
    - `name` **String** sensor name

  The semantic specification that defines `valueType`, `unit` and `typeId` properties can be find [here](https://knot-devel.cesar.org.br/doc/thing/unit-type-value.html?highlight=typeid).

  Example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "schema": [{
      "sensorId": 1,
      "valueType": 0xFFF1,
      "unit": 0,
      "typeId": 3,
      "name": "Door lock"
    }]
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing key: device.schema.sent

</details>

### **data.sent** <a name="data-sent"></a>

Event that represents a device sending the data gathered from its sensors to the services that are interested. After receiving this event, `babeltower` makes the necessary semantic validation and send a [`data.published`](#data-published) event.

<details>
  <details>
    <summary>Headers</summary>

    - `token` **String** user's token

  </details>

  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `data` **Array** data items to be published, each one formed by:
    - `sensorId` **Number** sensor ID
    - `value` **Number|Boolean|String** sensor value

  Example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "data": [
      {
        "sensorId": 1,
        "value": false
      },
      {
        "sensorId": 2,
        "value": 1000
      }
    ]
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: fanout
    - Name: data.sent
    - Durable: `true`
    - Auto-delete: `false`

</details>

### **data.request** <a name="data-request"></a>

Event-command to request data from a thing's sensor. After receiving this event, `babeltower` makes the necessary semantic validation and send a [`device.<id>.data.request`](#device-[id]-data-request) event to be routed to the service which control the thing.

<details>
  <summary>Headers</summary>

  - `token` **String** user's token

</details>

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `sensorIds` **Array (Number)** IDs of the sensor to send last value

  Example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "data": [1]
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing key: data.request

</details>

### **data.update** <a name="data-update"></a>

Event-command to update a thing's sensor data. After receiving this event, `babeltower` makes the necessary semantic validation and send a [`device.<id>.data.update`](#device-[id]-data-update) event to be routed to the service which control the thing.

<details>
  <summary>Headers</summary>

  - `token` **String** user's token

</details>

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `data` **Array (Object)** updates for sensors/actuators, each one formed by:
    - `sensorId` **Number** ID of the sensor to update
    - `value` **Number|Boolean|String** data to be written

  Example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "data": [{
        "sensorId": 1,
        "value": true
    }]
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing key: data.update

</details>

## Subscribe

The external consumer applications can subscribe to the events described in this section to receive them and take the appropriate action.

### **device.registered** <a name="device-registered"></a>

Event that represents a thing was registered.

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `token` **String** thing's token
  - `error` **String** described the occurred error

  Success example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "token": "5b67ce6bef21701331152d6297e1bd2b22f91787",
    "error": null
  }
  ```

  Error example:

  ```json
  {
    "id": "3aa21010cda96fe9",
    "token": "",
    "error": "device already exists"
  }
  ```

</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing key: device.registered

</details>

### **device.unregistered** <a name="device-unregistered"></a>

Event that represents a thing was removed.

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `error` **String** described the occurred error

  Success example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "error": null
  }
  ```

  Error example:

  ```json
  {
    "id": "3aa21010cda96fe9",
    "error": "forbidden",
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing key: device.unregistered

</details>

### **device.schema.updated** <a name="device-schema-updated"></a>

Event that represents a thing's schema was updated.

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `error` **String** a string with detailed error message

  Success example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "error": null
  }
  ```

  Error example:

  ```json
  {
    "id": "3aa21010cda96fe9",
    "error": "invalid schema"
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing key: device.schema.updated

</details>

### **data.published** <a name="data-published"></a>

Event that represents a data published from a thing's sensor.

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `data` **Array** data items to be published, each one formed by:
    - `sensorId` **Number** sensor ID
    - `value` **Number|Boolean|String** sensor value

  Example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "data": [
      {
        "sensorId": 1,
        "value": false
      },
      {
        "sensorId": 2,
        "value": 1000
      }
    ]
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: fanout
    - Name: data.published
    - Durable: `true`
    - Auto-delete: `false`

</details>

### **device.[id].data.request** <a name="device-<id>-data-request"></a>

Event-command to request a specific thing's sensor data after validating if the sensor exists in thing's schema and the `value` is in a valid format.

<details>
  <summary>Headers</summary>

  - `token` **String** user's token

</details>

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `data` **Array** data items to be published, each one formed by:
    - `sensorId` **Number** sensor ID
    - `value` **Number|Boolean|String** sensor value

  Example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "data": [
      {
        "sensorId": 1,
        "value": false
      },
      {
        "sensorId": 2,
        "value": 1000
      }
    ]
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing Key: `device.<id>.data.request`

</details>

### **device.[id].data.update** <a name="device-<id>-data-update"></a>

Event-command to update a specific thing's sensor data after validating if the `data` is compatible with thing's schema .

<details>
  <summary>Headers</summary>

  - `token` **String** user's token

</details>

<details>
  <summary>Payload</summary>

  JSON in the following format:

  - `id` **String** thing's ID
  - `data` **Array** data items to be published, each one formed by:
    - `sensorId` **Number** sensor ID
    - `value` **Number|Boolean|String** sensor value

  Example:

  ```json
  {
    "id": "fbe64efa6c7f717e",
    "data": [
      {
        "sensorId": 1,
        "value": false
      },
      {
        "sensorId": 2,
        "value": 1000
      }
    ]
  }
  ```
</details>

<details>
  <summary>AMQP Binding</summary>

  - Exchange:
    - Type: direct
    - Name: device
    - Durable: `true`
    - Auto-delete: `false`
  - Routing Key: `device.<id>.data.update`

</details>