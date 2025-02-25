# Generating an API Token for Application Development

Document Version: V1.0

##### Change Log

| Version | Date       | Content          |
| ------- | ---------- | ---------------- |
| 1.0     | 2025-01-10 | Document created |

##### Applicable Products
| Product | Version |
| ------- | ------- |
| AIG-302 | 1.0 |

##### Supported Environments

* Linux-based systems, ARM32 architecture

##### Prerequisites

* Access to the system's web interface with admin or Azure IoT Edge permissions


## Purpose
Due to security concerns, AIG-302 does not support an embedded token (such as the mx-api-token used in AIG-301/501) with administrator permissions for internal use. However, a service account can be created to generate a token for application development.

This guide explains two methods for creating and retrieving an API token: via **Web Interface** and via **REST API**.

## Using the Web Interface
1. Create a Service Account</br>
    Navigate to: Cloud Connectivity > Azure IoT Edge > Module Permission
    ![Azure IoT Edge > Module Permission](./images/AIE-Module-Permission.png)
    Create a new service account and assign the required permissions.
    ![Edit Service Accouont](./images/AIE-Module-Permission-Edit.png)

2. Copy or Download the Secret Access Key</br>
    After creating the service account, the secret access key will be provided. Make sure to copy or download it, as it will not be shown again.
    ![Secret Access Key](./images/AIE-Module-Permission-Secret-Access-Key.png)

3. Use the Token</br>
    Use the secret key as the token for API authentication. An example curl command is:
    ```bash
    curl -H "Authorization: Bearer <secret-access-key>" http://localhost:59000/api/v1/device/general
    ```

## Using REST API
Refer to https://tpe-tiger.github.io/AIG302/V1.0.0/#tag/azure-iotedge/paths/~1azure-iotedge~1services/post

1. Create a Service Account</br>
    Send a POST request to create a service account:
    ```json
    POST /api/v1/azure-iotedge/services
    Content-Type: application/json
    Authorization: Bearer <TOKEN_WITH_AZURE_IOTEDGE_PERMISSION>
    
    {
        "displayName": "My Test APP",
        "name": "testapp",
        "required": [
            "APP_AIE_RW",
            "SYS_MANAGEMENT_RW",
            "SYS_MAINTENANCE_RW"
        ],
        "source": "AzureIoTEdge"
    }
    ```

2. Retrieve the Secret Access Key</br>
    The response will include the secret access key ("token"). Save it securely, as it will not be retrievable again.
    ```json
    {
        "data": {
            "name": "testapp",
            "displayname": "My Test APP",
            "source": "AzureIoTEdge",
            "required": [
            "APP_AIE_RW",
            "SYS_MANAGEMENT_RW",
            "SYS_MAINTENANCE_RW"
            ],
            "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJVc2VySUQiOi0xLCJOYW1lIjoiQXp1cmVJb1RFZGdlL3Rlc3QiLCJTZXJpYWxOdW1iZXIiOiJUQkJHQjEwMjk0OTUiLCJTZXNzaW9uSUQiOjAsIlBlcm1pc3Npb25zIjpbIkFQUF9BSUVfUlciLCJTWVNfTUFOQUdFTUVOVF9SVyIsIlNZU19NQUlOVEVOQU5DRV9SVyJdLCJleHAiOjQ4ODg1NDA5Mjl9.Ec6BLGBKjet-xN6Zc39exhwogKJ2HAYJ8Z3Bch2wJS8"
        }
    }
    ```

3. Use the Token</br>
    Use the secret key as the token for API authentication. An example curl command is:
    ```bash
    curl -H "Authorization: Bearer <secret-access-key>" http://localhost:59000/api/v1/device/general
    ``

## Accessing APIs
Tokens can be used to access APIs via the following endpoints:

* Local Host:
    ```
    http://localhost:59000/api/v1<path>
    https://localhost:8443/api/v1<path>
    https://<LAN1-IP>:8443/api/v1<path>
    https://<LAN2-IP>:8443/api/v1<path>
    ```
* Container Access:
    ```
    http://172.31.0.1:59000/api/v1<path>
    ```