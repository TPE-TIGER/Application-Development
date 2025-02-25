# Virtual Tags by Hopping Window 

Document Version: V1.0

##### Change Log

| Version | Date       | Content          |
| ------- | ---------- | ---------------- |
| 1.0     | 2025-01-20 | Document created |

##### Applicable Products
| Product | Version |
| ------- | ------- |
| AIG-302 | 1.0 |


### Purpose

This document demonstrates how to use **Function** to update IO values through customized MQTT topic.

------

### Download and Setup

1. Download [mqtt-update-io.tar.gz](./samples/mqtt-update-io.tar.gz) and unpack it.
    ```
    mqtt-update-io/
    - index.py
    - package.json
    - requirements.txt
    ```

    - `requirements.txt` describes the python libraries required by the python program.

2. Understand package.json
    ```json
    {
        "name": "mqtt-update-io",
        "enabled": true,
        "trigger": {
            "driven": "timeDriven",
            "dataDriven": {
                "tags": {},
                "events": {}
            },
            "timeDriven": {
                "mode": "boot",
                "intervalSec": 1,
                "cronJob": ""
            }
        },
        "expose": {
            "tags": []
        },
        "params": {
            "version": "1.0",
            "host": "test.mosquitto.org",
            "port": 1883,
            "subTopic": "/v1.1/devices/#",
            "pubTopic": "/resp/v1.1/devices",
            "provider": "IO"
        }
    }
    ```

   | Key      | Description                                                  |
   | -------- | ------------------------------------------------------------ |
   | host     | Host of MQTT broker |
   | port     | Port of MQTT broker |
   | subTopic | Subscribed topic of the MQTT broker |
   | pubTopic | The publish topic after value updated |
   | provider | The provider of the tags |


3. Understand index.py

    ```python
    # The callback for when a PUBLISH message is received from the server.
    def on_message(client, userdata, msg):
        print(msg.topic+" "+str(msg.payload))
        token = msg.topic.split("/")

        # topic format: /<version>/devices/<device>/<variable name>
        if token[0] != "" or len(token) < 4:
            print("invalid topic")
            return
        device = token[3]
        variable = ""
        if len(token) >= 5:
            variable = token[4]
        #print("device: {}, variable: {}".format(device, variable))

        values = json.loads(msg.payload.decode("ascii"))
        if variable == "":
            return set_variables(client, userdata, device, values)

        set_variable(client, userdata, device, variable, values)
    ```

    - The subscribed topic format is `/<version>/devices/<device>/<variable>`, and the payload is the value of the variable.
      - \<version\>: indicates the communication version
      - \<device\>: a device of the provider, same as the "source" from the tag definition
      - \<variable\>: a device's variable, same as the "tag name" from the tag definition
    - This callback update a IO device's value (IO defined by `provider` in `package.json`) when a message received. For example, a topic `/v1.1/devices/DI/DI-01` with payload `{"value": false}` will update the `IO/DI/DI-01` value to `false`.


### Install

1. Install required Python libraries by REST API.
    ```bash
    curl --location --insecure  --request PUT 'https://<device-ip>:8443/api/v1/function/requirements' \
    --header 'Content-Type: application/json' \
    --header 'Authorization: Bearer <token>' \
    --data '{"timeout":60,"pip":["paho-mqtt==1.6.1"]}'
    ```

2. Import [`mqtt-update-io.tar.gz`](./samples/mqtt-update-io.tar.gz) into AIG via Edge Computing > Function Management page.


### How to Use

1. Subscribe topic `/resp/#` for monitoring.
    ```bash
    mosquitto_sub -h test.mosquitto.org -t "/resp/#" -v
    ```
2. Update DO-01's value by publish topic `/v1.1/devices/DO` with payload `{"DO-01":true}`
    ```bash
    mosquitto_pub -t "/v1.1/devices/DO" -m '{"DO-01":true}' -h test.mosquitto.org
    ```
3.  The following output should be received from the subscription (step 1), and `IO/DO/DO-01`'s value should be changed from OFF to ON.
    ```
    /resp/v1.1/devices [{"tagName": "DO-01", "ts": 0, "dataType": "boolean", "dataValue": true, "prvdName": "IO", "srcName": "DO"}]
    ```