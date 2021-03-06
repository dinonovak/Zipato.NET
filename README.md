# LVMS.Zipato.NET
Unofficial open source C# helper library for communication with the Zipabox controller from Zipato. This library connects to the cloud-based API hosted on https://my.zipato.com/zipato-web/api/ (on Amazon AWS, web page only works if you're authenticated with my.zipato.com) AND to the local API http://IP:8080/v2. Local is always over HTTP. All calls support the async/await model.

According to www.zipato.com, Zipato is an interactive security and automation system based on cloud technology. You can control, customize and automate all power devices in your home, watch live video from your home and get instant alerts in case of any security issue. The controller/gateway supports various automation protocols (Z-Wave, ZigBee, KNX, 433 MHz, EnOcean).

This library is under development. Currently supported:
- Secure login to the API
- Retrieve generic info about your Zipabox (firmware, IP addresses etc).
- Retrieve endpoints
- Retrieve devices
- Retrieve attributes and attribute values. NOTE: The remote API returns names and attributeNames. The local API only returns the attributeNames; no names.
- Control endpoints such as On/Off, Position (roller shutters) and others
- Retrieve scenes and run scenes
- Retrieve room list
- Retrieve contacts
- Get meteo information
- Get alarm partitions, zones and ARM/DISABLE the alarm system by PIN
- Retrieve schedules (from the Rules)

This Portable Library is compatible with: (ASP).Net 4.5/4.6, Windows (Phone) 8.1 Universal Apps and Windows Phone 8.1 Silverlight. So you can use this library to automate your building(s) from desktop and web applications or from Windows Universal Apps.

## How to use?
Use the source code from this repository or download the NuGet package: [LVMS.ZipatoNet.Signed](https://www.nuget.org/packages/LVMS.ZipatoNet.Signed/). In this repo, you can find an example application named LVMS.Zipato.NET.TestClient (ConsoleApplication1).
	
	var client = new ZipatoClient(); // for the remote api
	var localClient = new ZipatoClient(apiUrl: "http://192.168.2.99:8080/v2/", localApi:true); // 192.168.2.99 is the local IP in this case
	bool loggedIn = await client.LoginAsync(credentials.UserName, credentials.Password);
	
To retrieve a list of endpoints:	

	var endpoints = await client.GetEndpointsAsync();

Note that this library typically caches data that won't change often such as an endpoint list. Most methods have an optional parameter named allowCache, with true as default value.

There are 3 supported styles:
- Use the object model, pass objects to methods
- Use names, such as endpoint names or scene names
- Use the UUID, unique GUIDs within the Zipato system

Style examples:
```
  bool lightsOn = await client.GetOnOffStateAsync(endpoint);
  bool lightsOn = await client.GetOnOffStateAsync("Office Lights");
  bool lightsOn = await client.GetOnOffStateAsync(Guid.Parse("e79e83d3-6d97-49cc-89a7-cec7baf0c948"));
```
To set the new state of a device, call SetOnOffStateAsync. For example, to turn on a lamp:
  ```
  await client.SetOnOffStateAsync("Office Lights", true);
  ```
To open a roller shutter half way:
```
  await client.SetPositionAsync("Bedroom shutter", 50);
```
To get a list of endpoints that you can turn on or off (because some devices such as a power strips have multiple endpoints):
```
  var onOffEndpoints = await client.GetEndpointsWithOnOffAsync();
```

The following code retrieves a list of security alarm partitions, takes the first partition, waits until the partition is ready (i.e. no movement detected by motion sensors), arms the alarm and disarms the alarm 5 seconds later.

```
var partitions = await client.GetAlarmPartitionsAsync();
var partition = await client.GetAlarmPartitionAsync(partitions[0].Uuid);

while (!await client.IsAlarmPartitionReady(partition.Uuid))
{
	await Task.Delay(TimeSpan.FromSeconds(5));
}
await client.SetAlarmModeAsync(partition, "0000", Enums.AlarmArmMode.AWAY);
await Task.Delay(TimeSpan.FromSeconds(5));
await client.SetAlarmModeAsync(partition, "0000", Enums.AlarmArmMode.DISARMED);
```

To learn about devices being offline
```
var offlineDevices = await client.GetDevicesOfflineAsync();
```

## Contributions

Contributions are welcome. Fork this repository and send a pull request if you have something useful to add.
