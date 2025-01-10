# Scheduled Operation Task 1

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

This document demonstrates how to use **Function** to perform a hourly based task by invoking AIG REST API.


------

### Download and Setup

1. Download [schedule1.tar.gz](./samples/scheduler1.tar.gz) and unpack it.
   ```
   scheduler1/
   - index.py
   - package.json
   ```

2. Understand package.json

   ```json
   {
     "name": "scheduler1",
     "enabled": true,
     "trigger": {
       "driven": "timeDriven",
       "timeDriven": {
         "mode": "cronJob",
         "intervalSec": 1,
         "cronJob": "0 * * * *"
       }
     },
     "expose": {},
     "executable": {
       "language": "python"
     },
     "params": {
       "timeLine": {      
         "08": ["01_Enable_SSH"],
         "18": ["02_Disable_SSH"]
       },
       "commands": {
         "01_Enable_SSH": {
           "displayName": "Enable SSH Service",
           "enable": true,
           "method": "PUT",
           "endPoint": "/system/sshserver",
           "payload": {
             "enable": true,
             "port": 22
           }
         },
         "02_Disable_SSH": {
           "displayName": "Disable SSH Service",
           "enable": true,
           "method": "PUT",
           "endPoint": "/system/sshserver",
           "payload": {
             "enable": false,
             "port": 22
           }
         }
       }
     }
   }
   ```

   1. First of all, you need to define all the AIG APIs (operation tasks) you would like to invoked under **params / commands** JSON section.

      Command Name: **02_Disable_SSH**

      | Key         | Value               | Desc                                                         |
      | ----------- | ------------------- | ------------------------------------------------------------ |
      | displayName | Disable SSH Service | Function will display the task status by displayName         |
      | enable      | true                | enable/disable this task, even it be arranged into schedule. |
      | method      | PUT                 | The method for this REST API end point.                   |
      | endPoint    | /system/sshserver   | The REST API end point to be call.                        |
      | payload     | {.....}             | The payload will submit to this REST API                  |

   2. Second, you need to define when to invoke which Commands you defined, under **params / timeLine** JSON section.

      - You don't need to declare full time range. Just the hours with commands.
      - You can declare multiple commands for an hour with JSON string array.

3. Understand index.py

   ```python
   if __name__ == "__main__":
       config = package.Configuration()
       params = config.parameters()
       
       timeLine = params["timeLine"]
       commands = params["commands"]
       
       now = datetime.now()
       timeFrame = now.strftime("%H")
       print("Trigger schedule: " + timeFrame + " ================")
       
       if timeFrame in timeLine:
           commandList = timeLine[timeFrame]        
           for cmdIdx in commandList:
               cmd = commands[cmdIdx]
               if (cmd["enable"]):
                   print("Call API : " + cmd["displayName"])
                   result = call_API(cmd["method"], cmd["endPoint"], cmd["payload"])
                   print(result["status"])
                   print(result["message"])
       
       print("Exit schedule ================")
       print("")
   ```

   1. index.py will be launch by Function every hour.
   2. index.py will check the current time, and pick up matched commands from timeLine.
   3. index.py will invoke AIG API by command, and output result on log of Function.


### Next Action

If your operation tasks are complex than a simple REST API:

- [(Example) Scheduled Operation Task 2](./Scheduled-Operation-Task-2.md)
