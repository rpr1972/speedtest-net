![Project Logo](https://raw.githubusercontent.com/rpr1972/speedtest-net/main/logo.png)
# Speedtest-net
Unofficial Speedtest.net docker image/add-on with integration to Home Assistant through MQTT. This project is not affiliated with Okla's speedtest in any way.

Although being an <u>unofficial</u> speedtest.net image, it uses the **speedtest-cli** (https://github.com/sivel/speedtest-cli) linux client to connect to the speedtest.net official servers. The results are then passed to Home Assistant via MQTT integration. The image can run on standalone docker (for those running Home Assistant in a docker container) or as a HAOS add-on.

## Difference between this project and Speedtest-mqtt

This project aims to use the official Okla's infrastructure to get the statistics and then transfer them to Home Assistant. It has the benefit of only requiring the user to install the image in standalone docker or as a HAOS add-on to start using it. The project <u>Speedtest-mqtt</u> (https://github.com/rpr1972/speedtest-mqtt), on the other hand, uses its own set of client/server applications to gather the statistics, and as such, requires the installation of the server part in some external infrastructure - like a cloud provider. Despite this requirement, it can provide more accurate results, since it uses its own exclusive server - avoiding many of the bottlenecks that speedtest.net servers have (lots of users, high CPU and/or network usage of the provider that is hosting the server, etc). It's up to you to decide which one is better/enough for your use case.

## Environment variables
This image needs some environment variables to run properly. They are listed below:

**CHECK_UPDATES:** Indicates whether the application should check for updates ["true" or "false"]. **Default:** ["true"].

**MQTT_ADDR:** MQTT server IP address.

**MQTT_USER:** MQTT server user (new in 1.3.6). **Optional**.

**MQTT_PASS:** MQTT server password (new in 1.3.6). **Optional**.

**UNIQUE_ID:** Identifier for the object created in Home Assistant. It will be appended to the integration name (like speedtest_net_<UNIQUE_ID>). The purpose is to allow the creation of multiple containers to test multiple links.

**TZ:** Your timezone.

The MQTT user/pass information is optional, and if not provided, the client will authenticate as "anonymous" (at least, in Mosquitto server). In the case of the "**CHECK_UPDATES**" variable, only a "false" (lowercase) value will disable the updates (since the default is "true"). Any other value (or absence of value) is considered a "true" equivalent. Besides that, when set to "false", the "update" sensor will be removed from HA, since there is no point in having a sensor that does nothing. Of course, the sensor will be recreated if the setting is changed back to "true". When running as an add-on, this variable will be set to false, since HAOS already checks add-ons for newer versions.

## Directory mappings

There is only one directory that must be mapped, **if** running in standalone docker:

**/log**  (where the logfile **speedtest-net.log** is located)

This is because in the add-on the log will be shown directly in the HAOS UI - which retrieves them from journald.

## Home Assistant OS add-on

This image can be installed as an add-on in HAOS. To do that, follow the standard procedure for adding repositories to Home Assistant: click on Settings > Add-ons > Add-on Store (the button on the bottom right of the screen) and then click on the three dots on the upper right of the screen > Repositories and then add the link bellow:

https://github.com/rpr1972/speedtest-net

Then, after the add-on is installed, go to the "Configuration" tab and fill the variables shown there (which are the same environment variables explained above) and start the application.

## Versions (tags)

There are always two tags for each version and one multiarch tag for the current version. There are versions for the AMD64 (amd64-[VERSION] and amd64-latest) and ARM64 (arm64-[VERSION] and arm64-latest). The tag containing only the [VERSION] is multiarch - meaning it can be used to install the correct version on the desired platform without the need to specify the architecture. The ARM64 versions are for the ARMv8 or AARCH64 platforms.

## Docker image
To get the image, please go to https://hub.docker.com/repository/docker/robrosa/speedtest-net/general
