The instructions are written for a TP-LINK [TL-WR841Nv5](http://wiki.openwrt.org/toh/tp-link/tl-wr841nd) and a [TL-MR3020](http://wiki.openwrt.org/toh/tp-link/tl-mr3020)/[TL-WR703N](http://wiki.openwrt.org/toh/tp-link/tl-wr703n) flashed with OpenWrt 12.09 `r34185` (Attitude Adjustment). Other ar71xx devices may work as long as a properly interfaced GPIO pin is used.
### Other devices that may work: ###
  * TL-WR740N/[TL-WR741ND](http://wiki.openwrt.org/toh/tp-link/tl-wr741nd) before v4
  * [TL-WR841N](http://wiki.openwrt.org/toh/tp-link/tl-wr841nd) before v8

## Preparation ##
Kill any existing ntp daemons, unnecessary processes and disable wireless. Dedicate your router for serving NTP if possible.

[Download](http://openwrt-stratum1.googlecode.com/files/openwrt-stratum1.tar.gz) and untar to your router's `/tmp` directory

## GPIO Setup (TL-WR841N v5) ##

Examine the GPIO list:
```
root@OpenWrt:/tmp# cat /sys/kernel/debug/gpio
GPIOs 0-17, ath79:
 gpio-0   (tp-link:green:qss   ) out hi
 gpio-1   (tp-link:green:system) out hi
 gpio-11  (reset               ) in  hi
 gpio-12  (qss                 ) in  hi
 gpio-13  (tp-link:green:lan1  ) out lo
 gpio-14  (tp-link:green:lan2  ) out lo
 gpio-15  (tp-link:green:lan3  ) out lo
 gpio-16  (tp-link:green:lan4  ) out lo
 gpio-17  (tp-link:green:wan   ) out lo
```
We will use the QSS button (GPIO 12) since it is easy to solder into and is pulled up. Connect a TTL/CMOS 3.3V/5V PPS source as shown below:

![http://openwrt-stratum1.googlecode.com/files/pps.png](http://openwrt-stratum1.googlecode.com/files/pps.png)

You only need to supply the Schottky diode D1 (1N5817 or similar). A signal diode (1N4148) may be used.

**WARNING:** Check the polarity of the button with a known ground point, as indicated by the voltage levels.

**NOTE:** Disconnect the PPS source from the QSS button before turning on or rebooting the router, otherwise OpenWrt will enter failsafe mode. Connect after OpenWrt has finished starting.

## GPIO Setup (TL-MR3020/WR703N) ##

See the OpenWrt page about the using the [TL-MR3020 GPIO](http://wiki.openwrt.org/toh/tp-link/tl-mr3020#adding.i2c.bus) or the [TL-WR703N GPIO](http://pragti.ch/kippycam/Adding%20an%20I2C%20interface%20to%20the%20TL-WR703N.html). Remove the pulldown resistor (`R17`) then connect a 3.3V PPS source directly to GPIO 29. **A 5V source will need a level shifter.**

**FOR OTHER DEVICES:** GPIOs pulled down require a different method of interfacing. Some GPIOs need to be at a certain value when the device is turned on, see [this](http://wiki.openwrt.org/toh/tp-link/tl-wr703n/ar9331_pinout#gpios). Tell us if you successfully used other devices or GPIOs.

## PPS modules ##
Remove the `gpio_button_hotplug` module, then insert the PPS modules using the proper GPIO number: (see PpsGpioPollDriver)
```
root@OpenWrt:/tmp# rmmod gpio_button_hotplug
root@OpenWrt:/tmp# insmod lib/modules/3.3.8/pps_core.ko
root@OpenWrt:/tmp# insmod lib/modules/3.3.8/pps-gpio-poll.ko gpio=12
```
Check the dmesg output:
```
root@OpenWrt:/tmp# dmesg | tail
.....
[325884.950000] pps_core: LinuxPPS API ver. 1 registered
[325884.960000] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[325890.880000] pps pps0: new PPS source pps_gpio_poll
[325890.890000] pps-gpio-poll: Registered GPIO 12 as PPS source
```
Check if PPS is working properly:
```
root@OpenWrt:/tmp# usr/bin/ppstest /dev/pps0
trying PPS source "/dev/pps0"
found PPS source "/dev/pps0"
ok, found 1 source(s), now start fetching data...
source 0 - assert 1353943592.000033218, sequence: 16 - clear  0.000000000, sequence: 0
source 0 - assert 1353943593.000033288, sequence: 17 - clear  0.000000000, sequence: 0
source 0 - assert 1353943594.000033284, sequence: 18 - clear  0.000000000, sequence: 0
source 0 - assert 1353943595.000033229, sequence: 19 - clear  0.000000000, sequence: 0
source 0 - assert 1353943596.000033270, sequence: 20 - clear  0.000000000, sequence: 0
source 0 - assert 1353943597.000033231, sequence: 21 - clear  0.000000000, sequence: 0
```
You should see one line printed per second, without timeouts.

## NTPD setup ##
Create `/etc/ntp` and `/tmp/ntp`

`/etc/ntp.conf` should at least contain the following:
```
server 127.127.22.0 minpoll 3 maxpoll 3 iburst
fudge 127.127.22.0 flag2 0 flag3 0

server 0.pool.ntp.org minpoll 10 iburst prefer
server 1.pool.ntp.org minpoll 10 iburst prefer
server 2.pool.ntp.org minpoll 10 iburst prefer

driftfile /etc/ntp/drift

#Uncomment if you want to collect stats:
#statistics loopstats
#statsdir /tmp/ntp
#filegen peerstats file peers type day link enable
#filegen loopstats file loops type day link enable
```

Start ntpd (be sure to start the correct one in `/tmp/usr/bin`)

## Additional Notes ##
  * You may see `pps-gpio-poll: missed PPS pulse` in the `dmesg` output during periods of high CPU/network activity or the clock slewing.
  * `ntpq -pn` will show the PPS source having a jitter of 15us. The actual jitter is lower, this is a result of ntpd's precision calculation.
  * The NTP timestamps returned is about 30us late due to delays in timestamp calculation and probably in the transmit path.