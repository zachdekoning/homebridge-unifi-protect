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

### Protect Controller Autoconfiguration

This plugin will attempt to autoconfigure all devices it detects attached to a UniFi Protect controller in order to create a more seamless end user experience. In order to do so, the UniFi Protect controller local user account that you configure this plugin to use utilizes the full management role enabled for UniFi Protect. Enabling the full management role is *optional* and primarily required if you want this plugin to configure your UniFi Protect controller to make all RTSP streams available and providing some of the functionality you can selectively enable in additional feature options. You can also choose to manually enable all RTSP streams on all cameras yourself, if you prefer.

Why is enabling all RTSP streams a good idea? There's no performance penalty on the Protect controller end. More importantly, it provides flexibility in the stream quality that's available to use in HBUP which you can further tailor in a granular way using feature options. For this plugin to work correctly, you will need to enable at least one RTSP stream on each camera that you want to see in HomeKit.

Which leads to the final point on autoconfiguration - ***sane*** defaults. [This plugin's north star](https://github.com/hjdhjd/homebridge-unifi-protect#readme) is to make it as easy and seamless to integrate with HomeKit as possible in order to provide a terrific user experience -- and that includes great video streaming performance. By default, this plugin dynamically selects the best streaming quality to use by understanding HomeKit's limitations and using the RTSP profile (High, Medium, or Low) that most closely resembles the quality being requested by the streaming client.

Of course, you can override which RTSP profiles are used in your specific environment. See the video section of the HBUP feature options webUI for more detail.

### Transcoding and Transmuxing

Consistent with the ethos of this plugin, HBUP always strives to default to combining the highest possible quality combined with the best quality that can be delivered within the performance capabilities of the hardware environment HBUP is run on.

Briefly:

* *Transcoding* involves a CPU-intensive conversion of a video stream from one format or quality level to another. The process can be made significantly faster (with caveats and compromises) and less resource intensive through the use of GPU hardware acceleration.
* *Transmuxing* involves repackaging a video stream from one container format or another. No change in format or quality occurs. Notably, it is not a resource intensive activity.

#### Transcoding Customization

HomeKit bitrates are notoriously conservative from a bandwidth and quality perspective - they're downright low and result can result in far less than ideal video quality. You can further customize the bitrates used for local and non-local streaming when transcoded, using HBUP's feature options.

The defaault behavior can be tailored to your preferences, using the appropriate [feature options](https://github.com/hjdhjd/homebridge-unifi-protect/blob/main/docs/FeatureOptions.md#video).

#### How HBUP Stream Quality Selection Works When Transcding
When transcoding, these are the rules used to determine which Protect camera streams (Protect has three stream qualities available: High, Medium, and Low) are used for transcoding.

##### Livestreams

In order, the following occurs:

* If you have HomeKit Secure Video enabled, we will always use the existing video that's being captured through the Protect API. The quality setting used for HKSV event recording will be the input quality for a HomeKit livestream session. You can change this behavior through disabling the use of the livestream API using the appropriate [feature option](https://github.com/hjdhjd/homebridge-unifi-protect/blob/main/docs/FeatureOptions.md#video). If use of the Protect API has been disabled for livestreams, or you aren't using HKSV, keep reading.
* If you’re on a hardware accelerated platform, we always use the highest quality camera stream that's available to us as a starting point when transcoding. It always yields better performance and quality results for fixed-function hardware transcoders (e.g. most GPUs).
* If you’re not on a hardware accelerated platform, we always try to match the quality that’s being requested, with a bias of going higher versus lower, and then passing that along to the software transcoder.

##### HomeKit Secure Video Event Recording

* By default, the highest available stream quality is used to give you the best event recording output you can get before HomeKit further processes and compresses it.
* On Raspberry Pi platform, the software interface to the onboard GPU transcoder has issues dealing with very high bitrate stream quality and HBUP will default to a starting point of **Medium** because of those capability constraints.

#### Customizing Defaults
The behavior described above can be tailored to your environment and taste through the HBUP feature options webUI. Specifically, you can:

* Choose to not use the Protect livestream API for livestreaming. **Default: use the Protect livestream API when possible.**
* Choose to transmux instead of transcode for local clients or not. **Default: transcode local clients.**
* Choose to transmux instead of transcode for high-latency / remote clients. **Default: transcode high-latency / remote clients.**
* Which Protect stream qualities to make available for HBUP. **Default: all stream qualities are available (High, Medium, and Low) that are configured on the Protect camera.**
* Bypass the intelligence applied by HBUP when selecting which Protect stream quality to use for transcoding livestreams by forcing the use of a specific Protect stream quality. **Default: HBUP will decide based on the transcoding rules above.**
* Change the Protect stream quality used for HomeKit Secure Video event recording. **Default: HBUP will use the highest quality Protect stream available.**

***In general, I would discourage most users from changing the defaults unless they have a specific need to do so. But if you're reading this, probably like to tinker with things. 😄 I have taken a lot of time and care to design these behaviors and defaults - they really do exist for good reasons.***
