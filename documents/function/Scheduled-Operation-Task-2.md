# Scheduled Operation Task 2

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

This document demonstrates how to use **Function** to perform a hourly based task by invoking AIG REST API with your business logic.


------

### Download and Setup

1. Download [schedule2.tar.gz](./samples/scheduler2.tar.gz) and unpack it.
   ```
   scheduler2/
   - index.py
   - package.json
   ```

2. Understand package.json

   ```json
   {
     "name": "scheduler2",
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
         "08": ["Enable_AID()"],
         "18": ["Disable_AID()"],
       }
     }
   }
   ```
   
   - You need to define when to invoke which Commands, under **params / timeLine** JSON section.
   
      - You don't need to declare full time range. Just the hours with commands.
      - You can declare multiple commands for an hour with JSON string array.
   
3. Understand index.py - **Main section**

   ```python
   if __name__ == "__main__":
       config = package.Configuration()
       params = config.parameters()
       
       timeLine = params["timeLine"]
       
       now = datetime.now()
       timeFrame = now.strftime("%H")
       print("Trigger schedule: " + timeFrame + " ================")
       
       if timeFrame in timeLine:
           commandList = timeLine[timeFrame]        
           for cmdName in commandList:   
               print("Invoke : " + cmdName)   
               result = eval(cmdName)
               print(result["status"])
               print(result["message"])
       
       print("Exit schedule ================")
       print("")
   ```
   
   - index.py will be launch by tpFunc every hour.
   - index.py will check the current time, and pick up matched commands from timeLine.
   - index.py will call python's method by matched name of commands,  .

4. Implement all the Commands, for example **Enable_AID() method**

   ```python
   def Enable_AID():
       result = {}
       config = get_AID_configuration()    
       if config != None:
           if config["provisioning"]["enable"]:
               result["status"] = "success"
               result["message"] = "Do nothing, the enable already true"
           else:
               config["provisioning"]["enable"] = True
               result = call_API("PUT", "/azure-device", config)
       else:
           result["status"] = "fail"
           result["message"] = "Can't read AID configuration"
       return result
   ```

   - This method will be called by main process, according to the declaration on package.json
   - You can implement it by business logic.

