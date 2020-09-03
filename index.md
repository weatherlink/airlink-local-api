# Introduction
The AirLink provides an HTTP interface to retrieve current air quality data.

Current condition data is formatted as a JSON document containing air quality data from the particulate matter sensor as well as readings from the on-board temperature / humidity sensor.

## Format of HTTP requests

The path for current conditions data is `/v1/current_conditions`.

The full URL is:

    http://<airlink-ip-address-here>/v1/current_conditions

For example, if the IP address of the AirLink is 192.168.1.3, then the URL would be:

    http://192.168.1.3/v1/current_conditions

Using curl:

    curl -X GET -H "application/json" http://192.168.1.3/v1/current_conditions

## Response format

### General response format
The general format of all API responses is a JSON object that always contains exactly two keys - `data` and `error`.

* The `data` field contains a JSON object storing the API response payload, or potentially `null` in the event of an error. The contents of the data object will be described below.
* The `error` field is usually `null` on success, but in the event of an error, will be an object with two fields - `code` and `message`.
  * `code` is a numeric error code
  * `message` is a human readable error message string describing the error.

### `current_conditions` data format
For AirLink current conditions, the data object contains the following fields:
* `did` - the device serial number as a string, e.g. "001D0A100000"
* `name` - the user controlled device name string, as set during registration
* `ts` - the UNIX timestamp of the moment the response was generated, with a resolution of seconds. If the time has not yet been synchronized from the network, this will instead measure the time in seconds since bootup.
* `conditions` - a list of current condition data records, one per logical sensor. For the v1 API for AirLink, there will always be exactly one data record in the list.

### Sensor data format
Each sensor data object contains at a minimum, the following fields:
* `lsid` - the numeric logic sensor identifier, or `null` if the device has not been registered.
* `data_structure_type` - a numeric enumeration identifying the kind of sensor data.

### Combined air quality and temperature / humidity sensor data
For AirLink combined air quality and temperature / humidity data, the data structure type is currently `6`.

Apart from the common fields described above, the following fields are defined:
* `temp` - the most recent valid air temperature reading in °F
* `hum` - the most recent valid humidity reading in %RH
* `dew_point` - the dew point temperature calculated from the most recent valid temperature / humidity reading in °F
* `wet_bulb` - the wet bulb temperature calculated from the most recent valid temperature / humidity reading and user elevation in °F
* `heat_index` - the heat index temperature calculated from the most recent valid temperature / humidity reading in °F
* `pm_1_last` - the most recent valid PM 1.0 reading calculated using atmospheric calibration in µg/m^3.
* `pm_2p5_last` - the most recent valid PM 2.5 reading calculated using atmospheric calibration in µg/m^3.
* `pm_10_last` - the most recent valid PM 10.0 reading calculated using atmospheric calibration in µg/m^3.
* `pm_1` - the average of all PM 1.0 readings in the last minute calculated using atmospheric calibration in µg/m^3.
* `pm_2p5` - the average of all PM 2.5 readings in the last minute calculated using atmospheric calibration in µg/m^3.
* `pm_10` - the average of all PM 10.0 readings in the last minute calculated using atmospheric calibration in µg/m^3.
* `pm_2p5_last_1_hour` - the average of all PM 2.5 readings in the last hour calculated using atmospheric calibration in µg/m^3.
* `pm_2p5_last_3_hours` - the average of all PM 2.5 readings in the last 3 hours calculated using atmospheric calibration in µg/m^3.
* `pm_2p5_nowcast` - the weighted average of all PM 2.5 readings in the last 12 hours calculated using atmospheric calibration in µg/m^3.
* `pm_2p5_last_24_hours` - the weighted average of all PM 2.5 readings in the last 24 hours calculated using atmospheric calibration in µg/m^3.
* `pm_10_last_1_hour` - the average of all PM 10.0 readings in the last hour calculated using atmospheric calibration in µg/m^3.
* `pm_10_last_3_hours` - the average of all PM 10.0 readings in the last 3 hours calculated using atmospheric calibration in µg/m^3.
* `pm_10_nowcast` - the weighted average of all PM 10.0 readings in the last 12 hours calculated using atmospheric calibration in µg/m^3.
* `pm_10_last_24_hours` - the weighted average of all PM 10.0 readings in the last 24 hours calculated using atmospheric calibration in µg/m^3.
* `last_report_time` - the UNIX timestamp of the last time a valid reading was received from the PM sensor (or time since boot if time has not been synced), with resolution of seconds.
* `pct_pm_data_last_1_hour` - the amount of PM data available to calculate averages in the last hour (rounded down to the nearest percent)
* `pct_pm_data_last_3_hours` - the amount of PM data available to calculate averages in the last 3 hours (rounded down to the nearest percent)
* `pct_pm_data_nowcast` - the amount of PM data available to calculate averages in the last 12 hours (rounded down to the nearest percent)
* `pct_pm_data_last_24_hours` - the amount of PM data available to calculate averages in the last 24 hours (rounded down to the nearest percent)

Notes:
* With the exception of fields ending in `_last`, all `pm_n_xxx` fields are calculated using a rolling window with one minute granularity that is updated once per minute a few seconds after the end of each minute.
* After power loss or in rare circumstances after a soft restart, the data used to compute PM averages may be lost and require 24 hours to fully refill. The `pct_pm_data_xxx` fields can be used to estimate the time interval actually covered by each average.
* After a soft restart, the time may be off by a few minutes until the device resynchronizes the time from a network time server. During this period where the time is uncertain, no additional data will be added to the PM averages. PM fields ending in `_last` will still update in real-time.
* Current conditions data do not include calculated Air Quality Indices. The calculation
of AQI varies from country to country and may require inputs that are not available to a single sensor. For AQI calculations using the latest AQI models, please visit [WeatherLink.com](https://www.weatherlink.com).

### Sample JSON
Below is a sample of the JSON response from `/v1/current_conditions`, after running the output through a JSON formatter:
```
{
  "data":{
    "did":"001D0A100021",
    "name":"My AirLink",
    "ts":1599150192,
    "conditions":[
      {
        "lsid":123456,
        "data_structure_type":6,
        "temp":75.8,
        "hum":54.3,
        "dew_point":58.2,
        "wet_bulb":62.7,
        "heat_index":76.0,
        "pm_1_last":1,
        "pm_2p5_last":1,
        "pm_10_last":1,
        "pm_1":0.96,
        "pm_2p5":1.21,
        "pm_2p5_last_1_hour":2.30,
        "pm_2p5_last_3_hours":2.29,
        "pm_2p5_last_24_hours":4.81,
        "pm_2p5_nowcast":2.30,
        "pm_10":1.21,
        "pm_10_last_1_hour":2.84,
        "pm_10_last_3_hours":2.80,
        "pm_10_last_24_hours":6.03,
        "pm_10_nowcast":2.84,
        "last_report_time":1599150192,
        "pct_pm_data_last_1_hour":100,
        "pct_pm_data_last_3_hours":100,
        "pct_pm_data_nowcast":100,
        "pct_pm_data_last_24_hours":80
      }
    ]
  },
  "error":null
}
```

### Old versions
Older versions of firmware may report a data structure type of `5`. The format is nearly identical to the type 6 document described above, except that some of the fields were not named consistently and have been renamed:

| Type 6 Name           | Type 5 Name             |
| --------------------- | ----------------------- |
| `pm_10`               | `pm_10p0`               |
| `pm_10_last_1_hour`   | `pm_10p0_last_1_hour`   |
| `pm_10_last_3_hours`  | `pm_10p0_last_3_hours`  |
| `pm_10_last_24_hours` | `pm_10p0_last_24_hours` |
| `pm_10_nowcast`       | `pm_10p0_nowcast`       |


## HTTP Server Performance
The embedded web server can process approximately three requests per second.
Attempting to poll with multiple connections more often than three times per second may cause some HTTP requests to be dropped.

## Device discovery

AirLink devices advertise their presence on the local network using [DNS Service Discovery](https://tools.ietf.org/html/rfc6763) over [multicast DNS (mDNS)](https://tools.ietf.org/html/rfc6762).
These technologies are also known as Zeroconf or Bonjour or Avahi.

### DNS details
If you are familiar with DNS service discovery, the two most important parameters are the service name and the domain.
For AirLink, the service name is `_airlink._tcp` and the domain is `local`.

For most libraries, those two parameters should be all you need to plug in to start discovering AirLink devices.
In some cases, the library may expect the full DNS service record name, which for AirLink is:

    _airlink._tcp.local.

Note: there is a trailing dot at the end, and it is significant!

## Manually finding the IP address or hostname of an AirLink
As mentioned above, it is possible to programatically find the name and IP address
of AirLink devices on the local network using mDNS and DNS-SD. In the event that
you only need to find out the IP address for one device, there are some alternatives
that may be simpler.

AirLink devices identify themselves using a hostname of the form `airlink-<did>`,
where `did` is the last six digits of the serial number.
For example, for an AirLink device with a serial number of `001D0A100008`, the hostname would be `airlink-100008`.

### Via router admin page
When requesting an IP address via DHCP, the AirLink will identify itself to the router
using the hostname described above. Some routers have built-in administrative pages
that can be used to view a list of connected devices with their hostname and IP address.

### Via mDNS
If your computer supports mDNS name resolution, you can also try accessing an AirLink
device using its mDNS hostname, which is just the hostname with `.local`, e.g. `airlink-100008.local`.

### Via WeatherLink.com
If you are the registered owner of the device on [WeatherLink.com](https://www.weatherlink.com), you can find the
device's IP address on the "Health Data" page, listed as "AirLink IP Address".
