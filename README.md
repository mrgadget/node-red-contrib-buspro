# node-red-contrib-hdlbus
Node-Red implementation of HDL BusPro (SmartBus) protocol http://hdlautomation.com - forked from [node-red-contrib-buspro](https://github.com/efa2000/node-red-contrib-buspro), based on [smart-bus](https://github.com/caligo-mentis/smart-bus).

The motivation for creating this node set was to fill a perceived gap in functionality.  This package provides:
* state storage of devices - all level updates are stored in a JS object
* input/output nodes for device channels
* input/output nodes for universal switches
* a get node for device channels (based on stored values)
* a button color node to change the LED color of the new capacitive touch buttons
* a panel brightness node to facilitate dimming panels eg. at night
* input/output nodes enabling the creation of a virtual HVAC device
* raw nodes to allow you to craft messages to send/receive anything on the HDL network

This is by no means perfect - happy to hear any suggestions.  

If you're looking for a event timer - check out [eztimer](https://github.com/mrgadget/node-red-contrib-eztimer).

## BusPro-Controller
The node that holds connection to IP Gateway of BusPro (Smart-Bus) network (or broadcast address if using broadcast).

### Config
```js
defaults: {
            host: {value:"",required:true},   // HDL SmartBus gateway IP 
            port: {value:6000,required:true,validate:RED.validators.number()},    // and port, default: 6000 
            subnetid: {value: 1, required: true, validate: RED.validators.number()}, // Connector address in HDL network (Subnet ID)
            deviceid: {value: 99, required: true, validate: RED.validators.number()} // Connector address in HDL network (Device ID)
        }
```

## hdl-channel-in
Trigger flows based on pre-determined channel/level messages.
#### Output Payload
```js
{ "level": 0 }
```

## hdl-channel-out
Send channel level request.
#### Input Payload (Optional - used to override/specify any values)
```js
{
    "address": "1.50",
    "channel": 1,
    "level" 100
}
```

## hdl-channel-get
Returns the current level of a given channel.  This is obtained from the level store maintained by watching events (rather than requiring a request/response to get the current level). 
#### Input Payload (Optional - used to override/specify any values)
```js
{
    "address": "1.50",
    "channel": 1,
}
```
#### Output Payload
```js
{
    "address": "1.50",
    "channel": 1,
    "level" 100
}
```

## hdl-uv-in
Trigger a flow based on a pre-determined uv switch change.
#### Output Payload
```js
{ "state": false }
```

## hdl-uv-out
Send UV switch request

#### Input Payload (Optional - used to override/specify any values)
```js
{
    "address": "1.50",
    "switch": 1,
    "state" true
}
```

## hdl-btn-color
Send request to update button color.  This commands supports updating both the **on** and **off** colors independantly.  This does not survive a device reboot.

#### Input Payload (Optional - used to override/specify any values)
1. White
2. Red
3. Green
4. Blue
5. Orange
```js
{
    "colorOff": 1
    "colorOn": 4
}
```

## hdl-panel-brightness
Send request to update panel brightness (1-255).  Note that low brightness values can impact the colors that can be produced.

#### Input Payload (Optional - used to override/specify any values)
```js
{
    "address": "1.50",
    "brightness": 1,
}
```

## hdl-virtual-hvac-in
Listens for HVAC request messages targetted at the specified address (eg. if you're emulating a HVAC device at `1.51`, use `1.51` in the address field) and outputs  the data as a JSON payload

### Output Payload
```js
{
    "acstatus": 1, //Status: 1 on, 0 off
    "acno": 1, //AC No, set in DLP
    "setupmode": 1, //Mode: 0 Cooling, 1 Heating, 2 Fan, 3 Auto, 4 Dry
    "setupspeed": 0, //Fan Speed: 0 Auto, 1 High, 2 Medium, 3 Low.
    "temperature": {
        "type": 0, //Type: 0 celsius, 1 farenheit
        "now": 21, //DLP Temprature
        "cooling": 21, //Cooling Temprature
        "heating": 21, //Heating Temprature
        "auto": 21, //Auto Temprature
        "dry": 21 //Dry Temprature
    },
    "modeandfan": 48, //Mode and Fan always 48
    "currentmode": 21, //Current Mode Temprature
    "sweep": 0 //Sweep
}
```

## hdl-virtual-hvac-out
Accepts a JSON payload (_must_ be complete) and transmits to the broadcast (255.255) address with a source of the specified address (eg. if you're emulating a HVAC device at `1.51`, use `1.51` in the address field).

### Input Payload
```js
{
    "acstatus": 1, //Status: 1 on, 0 off
    "acno": 1, //AC No, set in DLP
    "setupmode": 1, //Mode: 0 Cooling, 1 Heating, 2 Fan, 3 Auto, 4 Dry
    "setupspeed": 0, //Fan Speed: 0 Auto, 1 High, 2 Medium, 3 Low.
    "temperature": {
        "type": 0, //Type: 0 celsius, 1 farenheit
        "now": 21, //DLP Temprature
        "cooling": 21, //Cooling Temprature
        "heating": 21, //Heating Temprature
        "auto": 21, //Auto Temprature
        "dry": 21 //Dry Temprature
    },
    "modeandfan": 48, //Mode and Fan always 48
    "currentmode": 21, //Current Mode Temprature
    "sweep": 0 //Sweep
} 
```

## hdl-raw-in
Receive (any) commands from HDL (Smart-Bus) network

### Output Message
```js
msg:{
    "sender": "1.2" //ID of Sender Device
    "target": "255.255" //ID of Target Device
    "code": 50    //Integer with command operation code
    "payload": {}   //Object with decoded data or raw buffer if data can not be parsed automatically
}
```

## hdl-raw-out
Send (any) commands to the HDL (Smart-Bus) network

### Input Message
```js
msg:{
    "target": "1.52" //ID of Target Device
    "code": 49    //Integer with command operation code
    "payload": { //Object with data or raw buffer 
            "channel": 2,
            "level": 100
  	}   
}
```

# Change Log

## 1.1.4
* Added input/output nodes for creating virtual HVAC device.