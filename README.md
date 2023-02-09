# Node-M2M Quick Tour
   1. [Client-Server Using Channel Api](#client-server-using-channel-api)
   2. [Publish-Subscribe Pattern](#publish-subscribe-pattern)
   3. [Edge Computing Using Local Area Networking](https://github.com/Node-M2M/m2m-edge-example)
   4. [Using a Browser Client](#using-a-browser-client)
   5. [Raspberry Pi Remote Control](#raspberry-pi-remote-control)
   6. [Client-Server Using HTTP Api](https://github.com/EdAlegrid/http-api)
   7. [Monitor Data from Remote C/C++ Application through IPC (inter-process communication)](https://github.com/EdAlegrid/cpp-ipc-application-demo)
   8. [Web application demo using the fetch api](https://github.com/EdAlegrid/m2m-web-application-demo)
   9. [Web application demo using only an m2m browser client](https://github.com/EdAlegrid/m2m-browser-client-demo)
   10. [Monitor Data from Remote C# Application through IPC (inter-process communication)](https://github.com/EdAlegrid/csharp-ipc-application-demo)
   11. [File Integrity Monitoring](https://github.com/EdAlegrid/file-integrity-monitoring)
   12. [Create A Simple Gateway Load Balancer](https://github.com/EdAlegrid/gateway-load-balancer)

<br>

[API Reference](https://github.com/Node-M2M/M2M-API)

---
## Client-Server Using Channel Api
![](assets/quicktour.svg)
[](https://raw.githubusercontent.com/EdoLabs/src2/master/quicktour.svg?sanitize=true)

In this quick tour, we will create a simple server application that generates random numbers as its sole service and a client application that will access the random number using a *pull* and *push* method.

Using a *pull-method*, the client will capture the random number as one time function call.

Using a *push-method*, the client will watch the value of the random number. The remote device will check the random value every 5 seconds. If the value changes, it will send or *push* the new value to the client.   

Before you start, ensure you have a [node.js](https://nodejs.org/en/) installation on your client and device computers. [Create an account](https://www.node-m2m.com/m2m/account/create) and register your remote device.

### Remote Device Setup

#### 1. Create a device project directory and install *m2m*.

```js
$ npm install m2m
```

#### 2. Save the code below as *device.js* in your device project directory.

```js
const m2m = require('m2m');

let testData = 'node-m2m';

// the deviceId 100 must must be registered with node-m2m
let device = new m2m.Device(100);

device.connect(() => {
  // set 'random-number' as publish data resource  
  device.publish('random-number', (data) => {
    let rn = Math.floor(Math.random() * 100);
    data.send(rn);
  });
  
  // set 'test-data' as another data resource  
  device.dataSource('test-data', (data) => {
    if(data.payload){
      testData =  data.payload;
    }
    data.send(testData);
  });
});
```
#### 3. Start your device application.

```js
$ node device.js
```

The first time you run your application, it will ask for your full credentials.
```js
? Enter your userid (email):
? Enter your password:
? Enter your security code:

```
The next time you run your application, it will start automatically using a saved user token.

However, after a grace period of 15 minutes, you may need to provide your *security code* to restart your application.

At anytime, if you're having difficulty or issues restarting your application, you can re-authenticate with an `-r` flag. This will refresh your token as shown below.
```js
$ node device.js -r
```

### Remote Client Setup

#### 1. Create a client project directory and install *m2m*.

```js
$ npm install m2m
```

#### 2. Save the code below as *client.js* in your client project directory.

**Method 1**

If you are accessing only one remote device from your client application, you can use this api. 

Create an *alias* object using the client's *accessDevice* method as shown in the code below.

```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(() => {

  // access the remote device using an alias object
  let device = client.accessDevice(100);

  // capture 'random-number' data using a pull method
  device.getData('random-number', (data) => {
    console.log('getData random-number', data); // 97
  });

  // capture 'random-number' data using a push method
  device.subscribe('random-number', (data) => {
    console.log('subscribe random-number', data); // 81, 68, 115 ...
  });

  // update 'test-data' topic using the payload 'node-m2m is awesome'
  device.sendData('test-data', 'node-m2m is awesome', (data) => {
    console.log('sendData test-data', data);
  });

  // then capture the  updated 'test-data' topic
  device.getData('test-data', (data) => {
    console.log('getData test-data', data); // node-m2m is awesome
  });

});
```

**Method 2**

If you will be accessing multiple remote devices from your client application, use this api.

Instead of creating an alias, just provide the *device id* through the various available methods from the client object.

```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(() => {

  // capture 'random-number' data using a pull method
  client.getData({id:100, topic:'random-number'}, (data) => {
    console.log('getData random-number', data); // 97
  });

  // capture 'random-number' data using a push method
  client.subscribe({id:100, topic:'random-number'}, (data) => {
    console.log('subscribe random-number', data); // 81, 68, 115 ...
  });

  // update 'test-data'
  client.sendData({id:100, topic:'test-data', payload:'node-m2m is awesome'}, (data) => {
    console.log('sendData test-data', data);
  });

  // capture updated 'test-data'
  client.getData({id:100, topic:'test-data'}, (data) => {
    console.log('getData test-data', data); // node-m2m is awesome
  });

});
```

#### 3. Start your application.
```js
$ node client.js
```
Similar with remote device setup, you will be prompted to enter your credentials.

You should get a similar output result as shown below.
```js
getData random-number 25
subscribe random-number 76
sendData test-data node-m2m is awesome
getData test-data node-m2m is awesome
```

<br>

## Publish-Subscribe Pattern
![](assets/quicktour.svg)
[](https://raw.githubusercontent.com/EdoLabs/src2/master/quicktour.svg?sanitize=true)

This is  a quick tour using the *publish-subscribe* pattern. It is actually similar with the watch/setData method from the *client-server quicktour* example.

Here the *topic name* is the same with *channel data* or *channel name* resources from remote devices. 
You can subscribe to a specific device or multiple devices by specifying its *device id* along with the *topic name*.

You can query the available devices in your account and discover the resources (topic/channel, gpio and http) available from each device from the browser interface or using a CLI. Check the API for more information.  

<br>

### Remote Device (publisher) Setup

#### 1. Create a device project directory and install *m2m*.

```js
$ npm install m2m
```

#### 2. Save the code below as *device.js* in your device project directory.

```js
const m2m = require('m2m');

let device = new m2m.Device(100);

device.connect(() => {
  // publish 'random-number' topic in your device 
  device.publish('random-number', (data) => {
    let rn = Math.floor(Math.random() * 100);
    data.send(rn);
  });

});
```
#### 3. Start your device application.

```js
$ node device.js
```
### Remote Client (subscriber) Setup

#### 1. Create a client project directory and install *m2m*.

```js
$ npm install m2m
```

#### 2. Save the code below as *client.js* in your client project directory.

**Method 1**

Using an alias method.

```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(() => {

  // access the remote device using an alias object
  let device = client.accessDevice(100);

  // subscribe to 'random-number' topic from device 100
  device.subscribe('random-number', (data) => {
    console.log('subscribe random-number', data); // 81, 68, 115 ...
  });

});
```

**Method 2**

Access remote devices directly from the client object. 

```js
const m2m = require('m2m');

let client = new m2m.Client();

client.connect(() => {

  // subscribe to 'random-number' topic from device 100
  client.subsribe({id:100, topic:'random-number'}, (data) => {
    console.log('subsribe random-number', data); // 81, 68, 115 ...
  });

});
```

#### 3. Start your application.
```js
$ node client.js
```
You should get a similar output result as shown below.
```js
subscribe random-number 76
subscribe random-number 34
...

```

<br>

## Using A Browser Client
<br>

Using the same device setup from the client-server quicktour, we will access the channel resources using a client from the browser.
## Browser Client Setup

#### 1. Login to [node-m2m](https://www.node-m2m.com/m2m/account/login) to create an access token. 

From the manage security section, generate an access token.

#### 2. Install *m2m* in your server.

Copy the minimized file `node-m2m.min.js` from `node_modules/m2m/dist` directory into your server's public javascript directory.

Include `node-m2m.min.js` on your HTML file `<script src="YOUR_SCRIPT_PATH/node-m2m.min.js"></script>`.

This will create a global **NodeM2M** object.

#### 3. Create a client object instance from the global NodeM2M object.

Access the resources from your remote devices from the available methods directly from the client instance as shown below.

```js
<script> 

// Protect your access token at all times  
var tkn = 'fce454138116159a6ad9a4234e71de810a1087fa9e7fbfda74503d9f52616fc5';
 
var client = new NodeM2M.Client(); 

client.connect(tkn, () => {

  // capture 'random-number' data using a pull method
  client.getData({id:100, topic:'random-number'}, (data) => {
    console.log('getData random-number', data); // 97
  });

  // capture 'random-number' data using a push method
  client.subscribe({id:100, topic:'random-number'}, (data) => {
    console.log('subscribe random-number', data); // 81, 68, 115 ...
  });

  // update test-data
  client.sendData({id:100, topic:'test-data', payload:'node-m2m is awesome'}, (data) => {
    console.log('sendData test-data', data);
  });

  // capture updated test-data
  client.getData({id:100, topic:'test-data'}, (data) => {
    console.log('getData test-data', data); // node-m2m is awesome
  });

});

</script>
```

Using your browser dev tools, you should get similar results as shown below. 
```js
getData random-number 25
watch random-number 76
sendData test-data node-m2m is awesome
getData test-data node-m2m is awesome
```
<br>

Check the [m2m browser client web application quick tour](https://github.com/EdAlegrid/m2m-browser-client-demo) for a complete web application using a browser client.

<br>

## Raspberry Pi Remote Control
![](assets/quicktour2.svg)
[](https://raw.githubusercontent.com/EdoLabs/src2/master/quicktour2.svg?sanitize=true)

In this quick tour, we will install two push-button switches ( GPIO pin 11 and 13 ) on the remote client, and an led actuator ( GPIO pin 33 ) on the remote device.

The client will attempt to turn *on* and *off* the remote device's actuator and receive a confirmation response of *true* to signify the actuator was indeed turned **on** and *false* when the actuator is turned **off**.

The client will also show an on/off response times providing some insight on the responsiveness of the remote control system.     

### Remote Device Setup

#### 1. Create a device project directory and install *m2m* and *array-gpio*.
```js
$ npm install m2m array-gpio
```
#### 2. Save the code below as *device.js* in your device project directory.

```js
const { Device } = require('m2m');

let device = new Device(200);

device.connect(() => {
  device.setGpio({mode:'output', pin:33});
});
```

#### 3. Start your device application.
```js
$ node device.js
```

### Remote Client Setup

#### 1. Create a client project directory and install *m2m* and *array-gpio*.
```js
$ npm install m2m array-gpio
```
#### 2. Save the code below as *client.js* in your client project directory.

```js
const { Client } = require('m2m');
const { setInput } = require('array-gpio');

let sw1 = setInput(11); // as ON switch
let sw2 = setInput(13); // as OFF switch

// enable pull-down resistor
sw1.setR(0);
sw2.setR(0);

let client = new Client();

client.connect(() => {

  let t1 = null;
  let device = client.accessDevice(200);

  sw1.watch(1, (state) => {
    if(state){
      t1 = new Date();
      console.log('turning ON remote actuator');
      device.output(33).on((data) => {
        let t2 = new Date();
        console.log('ON confirmation', data, 'response time', t2 - t1, 'ms');
      });
    }
  });

  sw2.watch(1, (state) => {
    if(state){
      t1 = new Date();
      console.log('turning OFF remote actuator');
      device.output(33).off((data) => {
        let t2 = new Date();
        console.log('OFF confirmation', data, 'response time', t2 - t1, 'ms');
      });
    }
  });
});
```
#### 3. Start your application.
```js
$ node client.js
```
The led actuator from remote device should toggle *on* and *off* as you press the corresponding ON/OFF switches from the client.

