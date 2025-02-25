# AIG Series Application Development

For AIG-301/501, please refer to https://github.com/TPE-TIGER/AIG301-501-Application-Development.

## AIG Function vs. AIE Module vs. Docker Container vs. Native Application

There are multiple ways for customers to develop, run your own application on AIG device. All these ways allow your application to integrate and leverage ThingsPro Edge's capabilities and features by REST API and SDK.

The application could be edge computing program (to calculate, manipulate, analysis data on edge), data acquisition (to fetch data by proprietary protocols), scheduling task (to arrange routine operation tasks), or data publish program (to transfer data to specific IT system).

Below table gives you ideas how to select suitable application development way to fit your requirements.

|                        | AIG Function                                          | AIE Module / Docker Container                                | Native Application
| ---------------------- | ----------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Framework              | Python program managed by AIG Function                | Docker container managed by Azure IoT Edge                   | By your own                                                  |
| Language               | Python 3                                              | All programing language                                      | All programing language                                      |
| TagHub SDK             | Python SDK                                            | Python SDK                                                   | Python SDK |
| Access TPE API         | OK                                                    | OK                                                           | OK |
| LAN Access             | OK                                                    | OK                                                           | OK |
| WAN Access             | OK                                                    | OK                                                           | OK | 
| Storage Access         | X                                                     | OK                                                           | OK |
| Serial Port Access     | X                                                     | OK (*)                                                       | OK (*) |
| BLE Access             | X                                                     | OK (*)                                                       | OK |
| Expose REST API        | OK                                                    | OK                                                           | OK |
| Expose Web GUI         | X                                                     | OK                                                           | OK |
| Knowledge Require      | Python coding tpFunc deployment                       | - Docker container Application<br />design<br />- Linux Driver & Utility<br />- Azure IoT Edge (AIE Module) | - Debian System<br />- Linux Programming |
| Program Design Pattern | - Time Driven<br />- Data Driven<br />- Web API Style | By your own                                                  | By your own |
| Other Limitations      | - all source code must keep at 1 file                 |                                                              | |

(*1) Serial ports are bind by AIG Modbus Master if there's device enabled in RTU/ASCII, disable RTU/ASCII devices of disable the whole service if you need to use them.
> ```bash
> # use following command to disable Modbus Master service in AIG
> sudo systemctl disable moxa-modbus-master
> sudo systemctl stop moxa-modbus-master
> ```
(*2) Need to mount physical resources into Docker container


## Development Guide

### Generate an API Token

If you need to use REST API to interactive with AIG, an API token should be created first for authentication. Please follow the guideline below to generate an API token.

[Generating an API Token for Application Development](./documents/Generating-an-API-Token-for-Application-Development.md)


### AIG Function

1. [What is Funciton](https://github.com/TPE-TIGER/tpe-function-sdk)
2. [Development Notes and Limitations](./documents/function/Development-Notes-and-Limitations.md)
3. [(Example) Scheduled Operation Task 1 (by confg)](./documents/function/Scheduled-Operation-Task-1.md)
4. [(Example) Scheduled Operation Task 2 (by code)](./documents/function/Scheduled-Operation-Task-2.md)
5. [(Example) Enable onChange Feature on TagHub](./documents/function/Enable-onChange-Feature-on-TagHub.md)
6. [(Example) Import Modbus Configuration File by Azure IoT Hub Direct Method](documents/function/Import-Modbus-Config-by-Azure-IoT-Hub-Direct-Method.md)
7. [(Example) Virtual Tags by Hopping Window](./documents/function/Virtual-Tags-by-Hopping-Window.md)
8. [(Example) Customize MQTT topic for updating tag values](./documents/function/Customize-MQTT-topic-for-updating-tag-values.md)

### Docker Container

1. [Developing a Docker Container for AIG](./documents/container/Developing-a-Docker-Container-for-AIG.md)

### Azure IoT Edge Module
ToDo
