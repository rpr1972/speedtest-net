## 2.0.4

* Fixed wrong MQTT topic name for version update.

## 2.0.3

* Code refactoring to make some routines more efficient.
* Fixed typos in some messages.

## 2.0.2

* Changed the way the application handles latency test failures when choosing the fastest server: now, it retries the test with the next server on the list.

## 2.0.1

* Changed the way data is generated for the upload test: instead of copying the whole dataset in each thread, now creates the dataset only once and clone the pointer to each thread, drastically minimizing memory usage.

## 2.0.0

* Moved away from speedtest-cli project: the application is using new custom code written in Rust to run the speed tests nativelly.
* Inclusion of more variables to fine tune the tests (see documentation).
* Major code refactoring to accomodate the new architecture.

## 1.0.2

* Changed the MQTT retain flag for the error sensor, so that it keeps the last known state on MQTT reconnections (in case of Home Assistant restarts, for example).
* Minor code refactoring.

## 1.0.1

* Added an error sensor to the integration, to show errors reported by the underlying speedtest.net linux client.
* Minor code refactoring to accomodate the new sensor.

## 1.0.0

* Initial release
