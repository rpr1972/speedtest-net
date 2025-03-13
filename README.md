![Project Logo](https://raw.githubusercontent.com/rpr1972/speedtest-net/main/logo.png)
# Speedtest-net
<ins>Unofficial</ins> Speedtest.net docker image/add-on with integration to Home Assistant through MQTT. This project <ins>is not affiliated</ins> with Ookla's Speedtest.net in any way.

Though it’s an <ins>unofficial</ins> Speedtest.net image, this project leverages custom Rust-based code to link up with Speedtest.net’s official servers, mirroring the functionality of their web version. The results are then smoothly relayed to Home Assistant through MQTT integration. You can run this image either in a standalone Docker container - perfect for those using Home Assistant in Docker - or as a convenient HAOS add-on, offering flexibility for different setups.

### <ins>Difference between this project and speedtest-mqtt</ins>

This project harnesses Ookla’s official Speedtest.net servers to collect network stats and seamlessly feeds them into Home Assistant. Its simplicity shines through - just install the image in a standalone Docker container or as an HAOS add-on, and you’re ready to go. By contrast, the Speedtest-mqtt project (https://github.com/rpr1972/speedtest-mqtt) relies on its own custom client/server setup to gather data, meaning you’ll need to deploy the server component on external infrastructure, like a cloud provider. While that extra step adds some setup effort, it can deliver more precise results by using a dedicated server, sidestepping the pitfalls that often plague Speedtest.net servers - think heavy user traffic, strained CPU or network resources at the hosting provider, or interference from HTTP proxies and caches. Ultimately, it’s your call to weigh which option best fits your needs.

### <ins>Environment variables</ins>
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

### <ins>Directory mappings</ins>

There is only one directory that must be mapped, **if** running in standalone docker:

**/log**  (where the logfile **speedtest-net.log** is located)

This is because in the add-on the log will be shown directly in the HAOS UI - which retrieves them from journald.

### <ins>Home Assistant OS add-on</ins>

This image can be installed as an add-on in HAOS. To do that, follow the standard procedure for adding repositories to Home Assistant: click on Settings > Add-ons > Add-on Store (the button on the bottom right of the screen) and then click on the three dots on the upper right of the screen > Repositories and then add the link bellow:

https://github.com/rpr1972/speedtest-net

Then, after the add-on is installed, go to the "Configuration" tab and fill the variables shown there (which are the same environment variables explained above) and start the application.

### <ins>Versions (tags)</ins>

There are always two tags for each version and one multiarch tag for the current version. There are versions for the AMD64 (amd64-[VERSION] and amd64-latest) and ARM64 (arm64-[VERSION] and arm64-latest). The tag containing only the [VERSION] is multiarch - meaning it can be used to install the correct version on the desired platform without the need to specify the architecture. The ARM64 versions are for the ARMv8 or AARCH64 platforms.

### <ins>Docker image</ins>
To get the image, please go to https://hub.docker.com/repository/docker/robrosa/speedtest-net/general
