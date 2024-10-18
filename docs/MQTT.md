<SPAN ALIGN="CENTER" STYLE="text-align:center">
<DIV ALIGN="CENTER" STYLE="text-align:center">

[![homebridge-unifi-protect: Native HomeKit support for UniFi Protect](https://raw.githubusercontent.com/hjdhjd/homebridge-unifi-protect/main/images/homebridge-unifi-protect.svg)](https://github.com/hjdhjd/homebridge-unifi-protect)

# Homebridge UniFi Protect

[![Downloads](https://img.shields.io/npm/dt/homebridge-unifi-protect?color=%230559C9&logo=icloud&logoColor=%23FFFFFF&style=for-the-badge)](https://www.npmjs.com/package/homebridge-unifi-protect)
[![Version](https://img.shields.io/npm/v/homebridge-unifi-protect?color=%230559C9&label=Homebridge%20UniFi%20Protect&logo=ubiquiti&logoColor=%23FFFFFF&style=for-the-badge)](https://www.npmjs.com/package/homebridge-unifi-protect)
[![UniFi Protect@Homebridge Discord](https://img.shields.io/discord/432663330281226270?color=0559C9&label=Discord&logo=discord&logoColor=%23FFFFFF&style=for-the-badge)](https://discord.gg/QXqfHEW)
[![verified-by-homebridge](https://img.shields.io/badge/homebridge-verified-blueviolet?color=%23491F59&style=for-the-badge&logoColor=%23FFFFFF&logo=homebridge)](https://github.com/homebridge/homebridge/wiki/Verified-Plugins)

## Complete HomeKit support for the UniFi Protect ecosystem using [Homebridge](https://homebridge.io).
</DIV>
</SPAN>

`homebridge-unifi-protect` is a [Homebridge](https://homebridge.io) plugin that provides HomeKit support to the [UniFi Protect](https://ui.com/camera-security) device ecosystem. [UniFi Protect](https://ui.com/camera-security) is [Ubiquiti's](https://www.ui.com) video security platform, with rich camera, doorbell, and NVR controller hardware options for you to choose from, as well as an app which you can use to view, configure and manage your video camera and doorbells.

### MQTT Support

[MQTT](https://mqtt.org) is a popular Internet of Things (IoT) messaging protocol that can be used to weave together different smart devices and orchestrate or instrument them in an infinite number of ways. In short - it lets things that might not normally be able to talk to each other communicate across ecosystems, provided they can support MQTT.

`homebridge-unifi-protect` will publish MQTT events if you've configured a broker in the controller-specific settings. Supported MQTT capabilities include:

  * Camera-specific RTSP information.
  * Doorbell message events. See [doorbell message events](#doorbell-messages) for additional details.
  * Doorbell ring events. Including triggering via MQTT.
  * Liveview-related events, including the security system accessory and security alarm.
  * Motion events.
  * Snapshot events, including publishing the actual images over MQTT.
  * And many more, documented below.

### Configuration

This documentation assumes you know what MQTT is, what an MQTT broker does, and how to configure it. Setting up an MQTT broker will not be covered here. There are plenty of guides available on how to do so just a search away.

You can configure MQTT settings in the plugin webUI. The settings are:

| Configuration Setting | Description
|-----------------------|----------------------------------
| `mqttUrl`             | The URL of your MQTT broker. **This must be in URL form**, e.g.: `mqtt://user:password@1.2.3.4`.
| `mqttTopic`           | The base topic to publish to. The default is: `unifi/protect`.

> [!IMPORTANT]
> **mqttUrl** must be a valid URL. Just entering a hostname will result in an error. The URL can use any of these protocols: `mqtt`, `mqtts`, `tcp`, `tls`, `ws`, `wss`.

When events are published, by default, the topics look like:

> ```sh
> unifi/protect/1234567890AB/motion
> unifi/protect/ABCDEF123456/doorbell
> ```

In the above example, `1234567890AB` and `ABCDEF123456` are the MAC addresses of your cameras. We use MAC addresses as an easy way to guarantee unique identifiers that won't change. `homebridge-unifi-protect` provides you information about your cameras and their respective MAC addresses in the homebridge log on startup. Additionally, you can use the UniFi Protect app or webUI to lookup what the MAC addresses are of your cameras, should you need to do so.

### <A NAME="publish"></A>Topics Published
The topics and messages that `homebridge-unifi-protect` publishes are:

| Topic                 | Protect Device Type | Message Published
|-----------------------|---------------------|------------------
| `alarm`               | Sensor              | `true` or `false` when a UniFi Protect sensor detects a recognized alarm sound.
| `ambientlight`        | Sensor              | Ambient light level, in lux, on a UniFi Protect sensor.
| `chime`               | Chime               | A number between 0 and 100 that represents the current volume of the chime as a percentage.
| `contact`             | Sensor              | `true` or `false` when a UniFi Protect sensor detects contact. Note: UniFi Protect will set this state differently depending on the placement type you select in the Protect app or the Protect webUI.
| `doorbell`            | Camera              | `true` when the doorbell is rung. Each press of the doorbell will trigger a new event.
| `humidity`            | Sensor              | Humidity percentage level on a UniFi Protect sensor.
| `leak`                | Sensor              | `true` or `false` when a UniFi Protect sensor detects a leak.
| `light`               | Light               | `true` or `false` when a UniFi Protect light is on or off.
| `light/brightness`    | Light               | A number between 0 and 100 that represents the current brightness as a percentage.
| `liveview`            | Viewport            | The current liveview being displayed on a UniFi Protect Viewport.
| `liveviews`           | Camera              | `[{"name": "LiveviewName", "state": true},{"name": "AnotherLiveview", "state": false}]`. `state` can be `true` or `false`, indicating whether a liveview scene is active.
| `message`             | Doorbell            | `{"message":"Some Message","duration":60}`. See [Doorbell Messages](#doorbell-messages) for additional documentation.
| `motion`              | Multiple            | `true` when motion is detected. `false` when the motion event is reset.
| <CODE>motion/smart/<I>object</I></CODE> | Cameras | `true` when a smart motion event is detected for *object*. `false` when the smart motion event is reset. Valid *object* types are currently: `licensePlate`, `person`, `package` `vehicle`. If what you want to listen for is whether a motion event is sent to HomeKit, use the `motion` topic instead.
| <CODE>motion/smart/<I>object</I>/metadata</CODE> | Cameras | `JSON` when a smart motion event such as a license plate is detected and Protect has metadata that may be useful associated with the detection event.  The JSON is the unfiltered metadata JSON as provided by UniFi Protect's telemetry for that event. Currently, this is only valid for `licensePlate` smart motion events.
| `occupancy`           | Multiple            | `true` when occupancy is detected. `false` when occupancy is no longer detected.
| `rtsp`                | Camera              | `{"Name": "URL"}`. Represents a JSON containing all the valid RTSP URLs that can be used to stream from this camera. `Name` is the name assigned by UniFi Protect to the RTSP URL. `URL` represents the URL that can be used for streaming.
| `securitysystem`      | Camera              | One of `Alarm`, `Away`, `Home`, `Night`, `Off`. This message is published every time the security state is set.
| `snapshot`            | Camera              | A [data URL](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs) containing a base64-encoded JPEG of the snapshot that was requested (either by HomeKit or MQTT).
| `systeminfo`          | Controller          | `JSON` containing the current Protect controller system information.
| `telemetry`           | Controller          | All UniFi Protect telemetry from the realtime events API. This is the raw feed - you're on your own to parse through it for the messages you may be interested in. This is published to the controller MAC address.
| `temperature`         | Sensor              | Temperature, in Celsius, on a UniFi Protect sensor.
| `tone`                | Chime               | `buzzer` or `chime` when a UniFi Protect chime plays the corresponding tone.

Messages are published to MQTT when an action occurs on a camera, controller, or doorbell that triggers the respective event, or when an MQTT message is received for one of the topics `homebridge-unifi-protect` subscribes to. For example, snapshot images are published every time HomeKit requests a snapshot as well as when a request is received through MQTT to trigger a new snapshot.

### <A NAME="subscribe"></A>Topics Subscribed
The topics that `homebridge-unifi-protect` subscribes to are:

| Topic                   | Protect Device Type | Message Expected
|-------------------------|---------------------|------------------
| `alarm/get`             | Sensor              | `true` will trigger a publish event of the current alarm state for a UniFi Protect sensor.
| `ambientlight/get`      | Sensor              | `true` will trigger a publish event of the ambient light level, in lux, for a UniFi Protect sensor.
| `chime/get`             | Chime               | `true` will trigger a publish event of the volume level for a UniFi Protect doorbell.
| `chime/set`             | Chime               | A number between 0 and 100 that will set the volume level for a UniFi Protect doorbell.
| `contact/get`           | Sensor              | `true` will trigger a publish event of the current contact state for a UniFi Protect sensor. Note: UniFi Protect will set this state differently depending on the placement type you select in the Protect app or the Protect webUI.
| `doorbell/set`          | Doorbell            | `true` will trigger the camera (if set as doorbell) or doorbell to generate a ring event.
| `humidity/get`          | Sensor              | `true` will trigger a publish event of the humidity level, as a percentage, for a UniFi Protect sensor.
| `leak/get`              | Sensor              | `true` will trigger a publish event of the current leak sensor state for a UniFi Protect sensor.
| `light/get`             | Light               | `true` or `false` when a UniFi Protect light is on or off.
| `light/set`             | Light               | `true` or `false` to turn on or off a UniFi Protect light.
| `light/brightness/get`  | Light               | `true` will trigger a publish event of the light brightness level, as a percentage, for a UniFi Protect light.
| `light/brightness/set`  | Light               | A number between 0 and 100 that will set the volume level of a UniFi Protect light.
| `liveview/get`          | Viewport            | `true` will request that the plugin publish the current liveview being displayed on a UniFi Protect Viewport.
| `liveview/set `         | Viewport            | The name of a liveview to be set as the currently displayed liveview on a Protect viewer.
| `liveviews/get`         | Camera              | `true` will trigger a publish event of all liveviews to the `liveviews` topic.
| `liveviews/set `        | Camera              | A JSON-compatible array in the format `[{"name": "view1", "state": true }, ...]` This will activate or deactivate one of more liveviews, depending on the respective state.
| `message/get`           | Doorbell            | `true` will trigger a publish event of the current message JSON for the doorbell. See [Doorbell Messages](#doorbell-messages) for additional documentation.
| `message/set`           | Doorbell            | A JSON in the format `{"message":"Some Message","duration":60}`. See [Doorbell Messages](#doorbell-messages) for additional documentation.
| `motion/get`            | Multiple            | `true` will trigger a publish event of the motion sensor state.
| `motion/set`            | Multiple            | `true` will trigger a motion event on the motion sensor.
| `occupancy/get`         | Multiple            | `true` will trigger a publish event of the occupancy sensor state.
| `rtsp/get`              | Camera              | `true` will request that the plugin publish a message to the `rtsp` topic containing a JSON of RTSP URLs for the camera or doorbell.
| `securitysystem/get`    | Camera              | `true` will trigger a publish event of the current state of the security system accessory.
| `securitysystem/set`    | Camera              | One of `AlarmOff`, `AlarmOn`, `Away`, `Home`, `Night`, `Off`. This will set the respective state on the security system accessory.
| `snapshot/set`          | Camera              | `true` will trigger the camera to generate a snapshot.
| `temperature/get`       | Sensor              | `true` will trigger a publish event of the temperature, in Celsius, for a UniFi Protect sensor.
| `tone/set`              | Chime               | A value of either `buzzer` or `chime` will play the selected tone on a UniFi Protect chime.

Some messages, such as those for the liveviews and securitysystem topics, are controller-specific. To use these topics, make sure you use the controller MAC address when you create your topic strings.

### <A NAME="doorbell-messages"></A>Doorbell Messages
Doorbell messages are a fun feature available in UniFi Protect doorbells. You can read the [doorbell documentation](https://github.com/hjdhjd/homebridge-unifi-protect/blob/main/docs/Doorbell.md) for more information about what the feature is and how it works.

Doorbell messages are published to MQTT using the topic `message`. `homebridge-unifi-protect` will publish the following JSON every time the plugin sets a message to the `message` topic:

> ```js
> { "message": "Some Message", "duration": 60}
> ```

| Property          | Description
|-------------------|----------------------------------
| `message`         | This contains the message that's set on the doorbell. An empty message, `""`, will reset the message display on the doorbell.
| `duration`        | This contains the duration that the message is set for, in seconds.

The accepted values for `duration` are:

| Duration Value    | Description
|-------------------|----------------------------------
| `0`               | This specifies that the message will be on the doorbell screen indefinitely.
| `number`          | This specifies that the message will be on the doorbell screen for `number` seconds, greater than 0.
| none              | A missing duration property will use the UniFi Protect default value of 60 seconds.

`homebridge-unifi-protect` subscribes to messages sent to the topic `message/set`. If you publish an MQTT message to the `message/set` topic containing a JSON using the above format, you can set the message on the doorbell LCD. This should provide the ability to arbitrarily set any message on the doorbell, programmatically.

`homebridge-unifi-protect` subscribes to messages sent to the topic `message/get`. If you publish an MQTT message containing `true` to the `message/get` topic, a message will be published to the `message` topic containing the current doorbell message and remaining duration in the JSON message format above.

> [!NOTE]
>   * MQTT support is disabled by default. It's enabled when an MQTT broker is specified in the configuration.
>   * MQTT is configured per-controller. This allows you to have different MQTT brokers for different Protect controllers, if needed.
>   * If connectivity to the broker is lost, it will perpetually retry to connect in one-minute intervals.
>   * If a bad URL is provided, MQTT support will not be enabled.
