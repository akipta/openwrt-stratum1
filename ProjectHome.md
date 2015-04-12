Inspired by the [Raspberry Pi NTP](http://www.satsignal.eu/ntp/Raspberry-Pi-NTP.html) projects, this hack allows you to run a stratum 1 NTP server driven by a PPS signal on Atheros [ar71xx (ath79)](https://dev.openwrt.org/wiki/ar71xx) routers. A [polling PPS GPIO driver](PpsGpioPollDriver.md) is used, giving a low jitter PPS source to `ntpd`. NMEA on the serial port is not yet covered.

[Installation](Installation.md) instructions

[Building](Building.md) from source


## Performance ##
![http://openwrt-stratum1.googlecode.com/files/offset.png](http://openwrt-stratum1.googlecode.com/files/offset.png)

TL-WR841Nv5 in a thermally stable environment, PPS source `minpoll 2`

The downward going spikes every hour are probably due to a scheduled task using the CPU and slightly heating the TCXO.