# Import Modbus Config by Azure IoT Hub Direct Method

Document Version: V1.0

##### Change Log

| Version | Date       | Content          |
| ------- | ---------- | ---------------- |
| 1.0     | 2025-01-09 | Document created |

##### Applicable Products
| Product | Version |
| ------- | ------- |
| AIG-302 | 1.0 |


### Purpose

This document demonstrates how to use **Function** to create and expose a HTTP REST endpoint to import Modbus Master configuration file. 

By this extension, you can invoke JSON based REST API via Azure IoT Hub/Direct Method or AWS IoT Core/Job or MQTT Topic to import Modbus Master configuration file.


------

### Download and Import

Download [modbus_import_csv.tar.gz](./samples/modbus_import_csv.tar.gz) and import it to AIG.


### How to use

1. ##### Invoke by general Restful API tools (such as postman)

   REST API End Point：https://{your device IP}:8443/api/v1/tpfunc/modbus/import

   Method：PUT

   Request Payload：

   ```json
   {
       "file": "https://xxxxxxxxxxxxxxxx/modbusmaster_221123090912.csv"
   }
   ```

2. ##### Invoke by Azure IoT Hub Direct Method

   Method Name：thingspro-api-v1

   Payload：

   ```json
   {
       "path":"/tpfunc/modbus/import",
       "method":"PUT",
       "headers":[],
       "requestBody": {"file": "https://xxxxxxxxxxxxxxxx/modbusmaster_221123090912.csv"}
   }
   ```

3. ##### Responses

   | Output Message                                               | Desc                                    |
   | ------------------------------------------------------------ | --------------------------------------- |
   | `{"status": "fail", "step": "1/3. download csv file from input URL: ", "content": "{}"}` | Fail on download CSV file, and reasons  |
   | `{"status": "fail", "step": "2/3. import csv content into Modbus Master", "content": "{}"}` | Fail on import CSV content, and reasons |
   | `{"status": "fail", "step": "3/3. apply Modbus Master configuration", "content": "{}"}` | Fail on apply configuration and reasons |
   | `{"status": "sucess", "step": "3/3. apply Modbus Master configuration", "content": "{}"}` | Success                                 |


