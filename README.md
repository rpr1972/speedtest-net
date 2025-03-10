![Project Logo](https://raw.githubusercontent.com/rpr1972/speedtest-net/main/logo.png)
# Speedtest-net
Unofficial Speedtest.net docker image/add-on with integration to Home Assistant through MQTT. This project is not affiliated with Okla's Speedtest in any way.

Although being an <u>unofficial</u> speedtest.net image, it uses custom code written in Rust to connect to speedtest.net official servers, much the same way the Web version of speedtest.net does. The results are then passed to Home Assistant via MQTT integration. The image can run on standalone docker (for those running Home Assistant in a docker container) or as a HAOS add-on.

## Difference between this project and Speedtest-mqtt

This project aims to use the official Okla's servers to get the statistics and then transfer them to Home Assistant. It has the benefit of only requiring the user to install the image in standalone docker or as a HAOS add-on to start using it. The project <u>Speedtest-mqtt</u> (https://github.com/rpr1972/speedtest-mqtt), on the other hand, uses its own set of client/server applications to gather the statistics, and as such, requires the installation of the server part in some external infrastructure - like a cloud provider. Despite this requirement, it can provide more accurate results, since it uses its own exclusive server - avoiding many of the bottlenecks that speedtest.net servers have (lots of users, high CPU and/or network usage of the provider that is hosting the server, etc). It's up to you to decide which one is better/enough for your use case.

## Environment variables
This image needs some environment variables to run properly. They are listed below:

**CHECK_UPDATES:** Indicates whether the application should check for updates [true or false]. **Default:** [true].

**DOWNLOAD_SIZE_MB:** Amount of data to use for download tests, in MB, as a float [min: 1.0, max: 500.0]. **Default:** [25.0].

**DOWNLOAD_THREADS:** Number of threads to use for download tests [min: 2, max: 16]. **Default:** [4].

**MQTT_ADDR:** MQTT server IP address. **Default:** [127.0.0.1].

**MQTT_USER:** MQTT server user. **Optional**.

**MQTT_PASS:** MQTT server password. **Optional**.

**UNIQUE_ID:** Identifier for the object created in Home Assistant. It will be appended to the integration name (like speedtest_net_<UNIQUE_ID>). The purpose is to allow the creation of multiple containers to test multiple links.

**UPLOAD_SIZE_MB:** Amount of data to use for upload tests, in MB, as a float [min: 1.0, max: 500.0]. **Default:** [2.0].

**UPLOAD_THREADS:** Number of threads to use for upload tests [min: 2, max: 16]. **Default:** [4].

**VERBOSE:** Indicates whether the application should show extra informative messages [true or false]. **Default:** [false].

**TZ:** Your timezone.

The MQTT user/pass information is optional, and if not provided, the client will authenticate as "anonymous" (at least, in Mosquitto server). In the case of the "**CHECK_UPDATES**" variable, only a "false" (lowercase) value will disable the updates (since the default is "true"). Any other value (or absence of value) is considered a "true" equivalent. Besides that, when set to "false", the "update" sensor will be removed from HA, since there is no point in having a sensor that does nothing. Of course, the sensor will be recreated if the setting is changed back to "true". When running as an add-on, this variable will be set to false, since HAOS already checks add-ons for newer versions.

The DOWNLOAD_XXX and UPLOAD_XXX variables allow fine tunning of the respective tests, since there are many variables that can interfere with the results in all Internet speed tests: server location (inside/outside your ISP infrastructure), server capacity (how much CPU/RAM it has), server load (how many users are running tests at the same time), network capacity/load of your ISP, etc. Because of all these variables, results may vary considerably even when run in close sequence - that's the nature of Internet speed tests that use shared servers, like Okla's Speedtest. Also, the technology of your Internet access plays an important role: fiber networks will be much faster/responsive than DSL and/or satellite. So, based on that fact, there is now a way to fine tune how the tests are run. 

For example: in a fiber access network with 700/350 Mbps of bandwidth, using Okla's Speedtest servers located inside the ISP, I get a latency <= 1ms. In this scenario, I use 8 threads down, with a download size of 150.0 MB and 4 threads up, with an upload size of 70.0 MB. This is more than enough to achieve ~705/366 Mbps - with these numbers being higher than the contracted bandwidth due to "burst-mode" that many providers offer. On the other hand, for a Starlink connection, I get latencies of around 60 ms and in this scenario I use 16 threads down, with a download size of 10.0 MB and 4 threads up, with an upload size of 2.5 MB. This is enough to saturate the bandwidth and get results as high as 405/39 Mbps - being the average something like 230/17 Mbps.

If you enable the VERBOSE flag, you can see messages like "**Server at ... transmitted less bytes than requested**" or "**Server at ... could not receive data**" in the log file. This indicates that the server might be under load or is configured to accept a smaller number of connections per client. In this cases, you should decrease the DOWNLOAD_XXX and/or UPLOAD_XXX to adjust for that. In the Starlink scenario described above, one could use 8 threads for download, with a download size of 20.0 MB, or even 10.0 MB.

One thing to note is: ISPs always want you to use speedtest.net servers inside their infrastructure, because you get better results, but that does not indicate that they have good uplinks on their own. Sometimes, they provide you the contracted bandwidth inside their network, but when you access other content in the Internet, you clearly see that your bandwidth is not on par with what you contracted. Usually, this happens when their oversubscription is very high - meaning that they have too much customers using their uplinks at the same time. Personally, I think that is much more realistic to choose speedtest.net servers outside your ISP infrastructure - this way you can detect occasional annomalies in your ISP network (that happened to me, by the way).

Bottom line is:

   * These kind of Internet speed tests are not scientifical - although they can provide a very good estimate;
   * Shared servers produce results that oscilate more than dedicated servers (like the speedtest-mqtt project);
   * Each access technology has its own set of values for tunning;
   * Start running tests and watching the log for warnings/errors. Tune these variables to suit your requirements.

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
