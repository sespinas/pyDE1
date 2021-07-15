# pyDE1

## License

Copyright © 2021 Jeff Kletsky. All Rights Reserved.

License for this software, part of the pyDE1 package, is granted under

GNU General Public License v3.0 only

SPDX-License-Identifier: GPL-3.0-only


## Overview

This represents work-in-progress to an API-first implementation of core software for a controller for the DE1.

The extent of functionality is sufficient to upload profiles and pull shots, flush the group, steam, and draw hot water through the API, with stop-at-time, -volume, and -mass. Continuous updates of flow parameters, and state transitions are provided over MQTT. Firmware upload is supported, though not yet revealed in the API.

A "worked example" is available at `examples/find_first_and_load.py` that

* Initializes and starts an MQTT listener, then, through the API
* Determines if a DE1 and scale are connected
* If not, connects to the first-found
* Waits until the DE1 is "ready" (self-initializes without API intervention)
* Uploads a profile
* Sets the stop-at-weight target and disables stop-at-time and stop-at-volume
* Optionally disconnects the DE1 and scale

The APIs are under semantic versioning. The REST-like, HTTP-transport versions can be retrieved from `version` at the document root, and also include the Python and package versions installed. Each of the JSON-formatted, MQTT packets contains a `version` key:value for that payload.

Consumers of these APIs should only need to understand high-level actions, such as "Here is a profile blob, please load it." The operations and choice of connectivity to the devices is "hidden" behind the APIs.

## Revision History

See also CHANGELOG.md

* 2021-07-14 – 0.5.0, "worked example" description
* 2021-07-03 – Updated for release 0.4.0, see also CHANGELOG.md
* 2021-06-26 – Content and organizational updates for release 0.3.0
* 2021-06-22 – Updated for release 0.2.0
* 2021-06-11 – Updated for release 0.1.0
* 2021-06-08 – Initial release

## Support and Discussion

Support and discussion is active at DecentForum.com, on Discord in the Decent Espresso server and, to some extent, on the Espresso Aficianados server in the Manufacturers: decent channel. Support is, unfortunately, ***not*** available through Decent Diaspora on Basecamp.

Thanks to all that have been trying this out and providing valuable feedback!

See also [https://github.com/jeffsf/pyDE1](https://github.com/jeffsf/pyDE1) where the *alpha* branch is current. 

## What's New in 0.5.0

_**Please see CHANGELOG.md for more details**_

### New

Bluetooth scanning with API. See `README.bluetooth.md` for details

API can set scale and DE1 by ID, by first_if_found, or None

A list of logs and individual logs can be obtained with GET `Resource.LOGS` and `Routine.LOG`

`ConnectivityEnum.READY` added, allowing clients to clearly know if the DE1 or scale is available for use.

> NB: Previous code that assumed that `.CONNECTED` was the terminal state
> should be modified to recognize `.READY`.

`examples/find_first_and_load.py` demonstrates stand-alone connection to a DE1 and scale, loading of a profile, setting of shot parameters, and disconnecting from these devices.

### Major Changes

HTTP API PUT/PATCH requests now return a list, which may be empty. Results, if any, from individual setters are returned as dict/obj members of the list.

On an error return to the inbound API, an exception trace is provided, when available. This is intended to assist in error reporting.

Some config parameters moved into `pyDE1.config.bluetooth`

"find_first" functionality now implemented in `pyDE1.scanner`

`de1.address()` is replaced with `await de1.set_address()` as it needs to disconnect the existing client on address change. It also supports address change.

`Resource.SCALE_ID` now returns null values when there is no scale.

There's virtually nothing left of `ugly_bits.py` as its functions now can to be handled through the API.

On connect, if any of the standard register reads fails, it is logged with its name, and retried (without waiting).

An additional example profile was added. EB6 has 30-s ramp vs EB5 at 25-s. Annoying rounding errors from Insight removed.

#### Resource Version 2.0.0

> NB: Breaking change: `ConnectivityEnum.READY` added. See Commit b53a8eb
> 
> Previous code that assumed that `.CONNECTED` was the terminal state
> should be modified to recognize `.READY`.

Add

```
    SCAN = 'scan'
    SCAN_DEVICES = 'scan/devices'
```

```
    LOG = 'log/{id}'
    LOGS = 'logs'
```

### Deprecated

`stop_scanner_if_running()` in favor of just calling `scanner.stop()`

`ugly_bits.py` for manual configuration now should be able to be handled through the API. See `examples/find_first_and_load.py`

### Removed

`READ_BACK_ON_PATCH` removed as PATCH operations now can return results themselves.

`device_adv_is_recognized_by` class method on DE1 and Scale replaced by registered prefixes

Removed `examples/test_first_find_and_load.py`, use `find_first_and_load.py`


## Requirements

Python 3.8 or later.

Available through `pip`:

* `bleak`
* `aiologger`
* `asyncio-mqtt`
* `paho-mqtt`
* `requests`

An MQTT broker compatible with MQTT 5 clients, such as `mosquitto 2.0` (see [below](#installing-mosquitto))

The Raspberry Pi version of Debian *Buster* ships with Python 3.7, which does not support named `asyncio.Task()` The "walrus operator" is also used.

Python 3.9 is expected to be part of Debian "next". Until that time, https://github.com/pyenv/pyenv can be used to install a version of your choice. On a RPi 3B, a complete build too under 15 minutes.

Development work is being done on *Buster* with Python 3.9.5 on a RPi 3B at this time.

The `bleak` library is supported on macOS, Linux, and Windows. Some development has also been done under macOS.

## Short-Term Priorities

* Profile and "history" database
* Manage unexpected disconnects and reconnects
* Abort long-running actions, such as uploading firmware
* Daemonize and provide Debian-compatible service script


## Known Gaps

* Timeouts on certain locks and await actions
* Single-command read of the DE1 debug register
* Clean, descale, transport
* Clean up the imports
* More doc strings and typing
* Stand-alone documentation
* Quick-start guide

## Other Work

* Onboard, unattended sleep timeout with override (GUI or HA can provide complex "scheduler")
* Background firmware update
* MQTT will and MQTT 5 message expiry time


## Status — Alpha

This code is work in progress and is neither feature-complete nor fully tested. 

Although most features are working, as described in Section 15 and elsewhere of the GPLv3.0 `LICENSE`:

> THERE IS NO WARRANTY FOR THE PROGRAM, TO THE EXTENT PERMITTED BY
APPLICABLE LAW.  EXCEPT WHEN OTHERWISE STATED IN WRITING THE COPYRIGHT
HOLDERS AND/OR OTHER PARTIES PROVIDE THE PROGRAM "AS IS" WITHOUT WARRANTY
OF ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING, BUT NOT LIMITED TO,
THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
PURPOSE.  THE ENTIRE RISK AS TO THE QUALITY AND PERFORMANCE OF THE PROGRAM
IS WITH YOU.  SHOULD THE PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF
ALL NECESSARY SERVICING, REPAIR OR CORRECTION. 



## Some Older Notes of Explanatory Value

_**Please see CHANGELOG.md for newer details**_

### 0.2.0

#### Inbound Control and Query API

An inbound API has been provided using a REST-like interface over HTTP. The API should be reasonably complete in its payload and method definitions and comments are welcomed on its sufficiency and completeness.

Both the inbound and outbound APIs run in separate *processes* to reduce the load on the controller itself.

GET should be available for the registered resources. See, in `src/pyDE1/dispatcher`

* `resource.py` for the registered resources, and
* `mapping.py` for the elements they contain, the expected value types, and how they nest.

`None` or `null` are often used to me "no value", such as for stop-at limits. As a result, though similar, this is not an [RFC7368 JSON Merge Patch](https://datatracker.ietf.org/doc/html/rfc7386).

In Python notation, `Optional[int]` means an `int` or `None`. Where `float` is specified, a JSON value such as `20` is permitted.

GET presently returns "unreadable" values to be able to better show the structure of the JSON. When a value is unreadable, `math.nan` is used internally, which is output as the JSON `NaN` token.

GET also returns empty nodes to illustrate the structure of the document. This can be controlled with the `PRUNE_EMPTY_NODES` variable in `implementation.py`

Although PATCH has been implemented for most payloads, PUT is not yet enabled. PUT will be the appropriate verb for`DE1_PROFILE` and `DE1_FIRMWARE` as, at this time, in-place modification of these is not supported. The API mechanism for starting a firmware upload as not been determined, as it should be able to abort as it runs in the background, as well as notify when complete. Profile upload is likely to be similar, though it occurs on a much faster time scale.

> The Python `http.server` module is used. It is not appropriate for exposed use.
> There is no security to the control and query API at this time.
> See further https://docs.python.org/3/library/http.server.html

It is likely that the server, itself, will be moved to a uWSGI (or similar) process. 

With either the present HTTP implementation or a future uWSGI one, use of a webserver, such as `nginx`, will be able to provide TLS, authentication, and authorization, as well as a more "production-ready" exposure.


#### Other Significant Changes

* `ShotSampleWithVolumeUpdates` (v1.1.0) adds `de1_time`. `de1_time` and `scale_time` are preferred over `arrival_time` as, in a future version, these will be estimates that remove some of the jitter relative to packet-arrival time.

* To be able to keep cached values of DE1 variables current, a read-back is requested on each write. 
* `NoneSet` and `NONE_SET` added to some `enum.IntFlag` to provide clearer representations
* Although `is_read_once` and `is_stable` have been roughed in, optimizations using them have not been done
* Disabled reads of `CUUID.ReadFromMMR` as it returns the request itself (which is not easily distinguishable from the data read. These two interpret their `Length` field differently, making it difficult to determine if `5` is an unexpected value or if it was just that 6 words were requested to be read.
* Scaling on `MMR0x80LowAddr.TANK_WATER_THRESHOLD` was corrected.


### 0.1.0

#### Outbound API

An outbound API (notifications) is provided in a separate process. The present implementation uses MQTT and provides timestamped, source-identified, semantically versioned JSON payloads for:

* DE1
	* Connectivity
	* State updates
 	* Shot samples with accumulated volume
 	* Water levels
* Scale
 	* Connectivity
 	* Weight and flow updates
* Flow sequencer
 	* "Gate" clear and set
	  	* Sequence start
	  	* Flow begin
	  	* Expect drops
	  	* Exit preinfuse
	  	* Flow end
	  	* Flow-state exit
	  	* Last drops
	  	* Sequence complete
  	* Stop-at-time/volume/weight
  		* Enable, disable (with target)
  		* Trigger (with target and value at trigger)

An example subscriber is provided in `examples/monitor_delay.py`. On a Raspberry Pi 3B, running Debian *Buster* and `mosquitto` 2.0 running on `::`, median delays are under 10 ms from *arrival_time* of the triggering event to delivery of the MQTT packet to the subscriber.

Packets are being sent with *retain* True, so that, for example, the subscriber has the last-known DE1 state without having to wait for a state change. Checking the payload's `arrival_time` is suggested to determine if the data is fresh enough. The *will* feature of MQTT has not yet been implemented.

A good introduction to MQTT and MQTT 5 can be found at HiveMQ:

* https://www.hivemq.com/mqtt-essentials/
* https://www.hivemq.com/blog/mqtt5-essentials-part1-introduction-to-mqtt-5/

One good thing about MQTT is that you can have as many subscribers as you want without slowing down the controller. For example, you can have a live view on your phone, live view on your desktop, log to file, log to database, all at once.

#### Scan For And Use First DE1 And Skale Found

Though "WET" and needing to be "DRY", the first-found DE1 and Skale will be used. The Scale class has already been designed to be able to have each subclass indicate if it recognizes the advertisement. Once DRY, the scanner should be able to return the proper scale from any of the alternatives. 

Refactoring of this is pending the formal release of `BleakScanner.find_device_by_filter(filterfunc)` from [bleak PR #565](https://github.com/hbldh/bleak/pull/565)


## High Level Functionality

* Connect by address to DE1
* Read and decode BLE characteristics 
* Encode and write BLE characteristics
* Read and decode MMR registers
* Encode and write MMR registers
* Upload firmware
* Parse JSON profile (v2) and upload
* Connect by address to SkaleII
* Scale processing for weight and flow, including period estimation
* Stop-at-time
* Stop-at-volume
* Stop-at-weight
* Enable/disable "shot" logging
* Outbound API over MQTT
* Basic connectivity tracking
* Find and use first DE1 and Skale
* Inbound control and query API over HTTP

The main process runs under Python's native `asyncio` framework. There are many tutorials out there that make asynchronous programming *look* easy. "Hello world!" is always easy. For a better understanding, I found Lynn Root's *[asyncio: We Did It Wrong](https://www.roguelynn.com/words/asyncio-we-did-it-wrong/)* to be very insightful.


<a name="installing-mosquitto"></a>
## Installing Mosquitto 2.0

The example outbound API uses MQTT 5. If you don't already have a local MQTT 5 broker configured, there are some public test servers ("brokers"), such as https://test.mosquitto.org/, that can let you try things out quickly. A local broker is better from both from a security standpoint and for delay. The preferred configuration is to have a broker running on the same machine as this code on a loopback interface. Unfortunately, the [`paho` library does not support Unix domain sockets](https://github.com/eclipse/paho.mqtt.c/issues/864) at this time.

The example outbound API does not use encryption as it runs over a socket local to the host, the data is not considered "sensitive", and there is no control over the DE1. Token-based authentication, 
such as password, should be done over an encrypted channel if can be "snooped" by others.

Mosquitto 2.0 is a MQTT broker that supports MQTT 5. Older distributions only supply 1.x versions, such as 1.5.7 on 
Debian *Buster.* Debian *Bullseye* is showing that it will support 2.0.10 at this time. 

Mosquitto 2.0 can be installed onto Debian systems without needing to build from source using the 
[Mosquitto Debian Repository](https://mosquitto.org/blog/2013/01/mosquitto-debian-repository/). The usual caveats around making personal decisions about which sources you trust apply.

You likely will want both `mosquitto` (the broker) and `mosquitto-clients`.

Installing on RPi will enable the `mosquitto.service` using `/etc/mosquitto/mosquitto.conf`. 
If you've used v1.x in the past, I'd suggest reading [the release announcement](https://mosquitto.org/blog/2020/12/version-2-0-0-released/)
as well as the notes on [migrating from 1.x to 2.0](https://mosquitto.org/documentation/migrating-to-2-0/)


## Notes

The code is littered with TODOs and personal notes. Ray may find his name mentioned with some loose thoughts about changes. *These are loose thoughts worthy of some future discussion, not blockers and not direct requests!*