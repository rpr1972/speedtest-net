## 1.0.2

* Changed the MQTT retain flag for the error sensor, so that it keeps the last known state on MQTT reconnections (in case of Home Assistant restarts, for example).
* Minor code refactoring.

## 1.0.1

* Added an error sensor to the integration, to show errors reported by the underlying speedtest.net linux client.
* Minor code refactoring to accomodate the new sensor.

## 1.0.0

* Initial release
