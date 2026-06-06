# General mythtv tips.
* * *
* * *

### Frontend connect errors on later versions of mariadb/mysql due to default encryption enabled on clients.
ERROR GIVEN:
     
    QMYSL: Unable to connect
    Database error was:
    TLS/SSL error : SSL is required, but the server does not support it


Test starting the frontend with :

      export MARIADB_TLS_DISABLE_PEER_VERIFICATION=1

If that works add it to your environment or a mythfrontend startup script.

* * *

## Can't connect using mythfrontend clients from other lan hosts
By default you can connect with mythfrontend using localhost after mythbackend setup or using UPnP.

*A lot of people just use a VPN solution that puts them right on the mythtv server host itself, but if you don't 
want to run a vpn for this and/or want local lan setup read below.*


### To connect from a non localhost machine running mythfrontend without using VPN direct to host:

1. For your setup for mariadb server, check your settings for bind-address.

   On debian this is at: /etc/mysql/mariadb.conf.d/50-server.cnf

   The change below will open up port 3306 on all interfaces, change to a specific ip if that's what you want.
   
         #bind-address            = 127.0.0.1
         bind-address            = 0.0.0.0
   

3. You will also need to add specific permissions for your clients that will connect as well.

   Eg. If you have a local network of '192.168.1.0/24' you can either add permissions to mariadb on a per client(a)
or per network(b) basis.

      a. Example to allow only 'mythtv' user from 192.168.1.101 host access your mythbackend/mariadb server where you already have a mythconverg db.

     
        CREATE USER IF NOT EXISTS 'mythtv'@'192.168.1.101' IDENTIFIED WITH mysql_native_password;
        ALTER USER 'mythtv'@'192.168.1.101' IDENTIFIED BY 'mythtv';
        GRANT ALL ON mythconverg.* TO mythtv@192.168.1.101;
        FLUSH PRIVILEGES;
        GRANT CREATE TEMPORARY TABLES ON mythconverg.* TO mythtv@192.168.1.101;
        FLUSH PRIVILEGES;
        ALTER DATABASE mythconverg DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
     

      b. Example to allow all hosts on 192.168.1.0/24 for 'mythtv' user to access your mythbackend/mariadb server where you already have a mythconverg db.
      

        CREATE USER IF NOT EXISTS 'mythtv'@'192.168.1.%' IDENTIFIED WITH mysql_native_password;
        ALTER USER 'mythtv'@'192.168.1.%' IDENTIFIED BY 'mythtv';
        GRANT ALL ON mythconverg.* TO 'mythtv'@'192.168.1.%';
        FLUSH PRIVILEGES;
        GRANT CREATE TEMPORARY TABLES ON mythconverg.* TO 'mythtv'@'192.168.1.%';
        FLUSH PRIVILEGES;
        ALTER DATABASE mythconverg DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;




To add either of the above you can enter the commands manually or import them from a file you created:

    sudo mysql < mc-singlehost-allow-192.168.1.101.sql
    
      or
    
    sudo mysql < mc-localnet-allow-192.168.1.0.sql
    
* * *

## RPI2 overclock
The Raspios raspi-config tool has a, overclock for the rpi2 that works quite well even with no heat sink/additional cooling.
It seems to really make a difference when using mythfrontend, much more responsive overall.
Since it is included in raspios-config, I think it's a fairly safe option to use.
This should work on most non raspios distros in /boot/config.txt

```
[pi2]
arm_freq=1000
core_freq=500
sdram_freq=500
over_voltage=2
```

### Set the default gov to 'performance' by appending this to your:
/boot/cmdline.txt or /boot/firmware/cmdline.txt

```
cpufreq.default_governor=performance
```

* * *

## RPI2 v1.1 limitations
* This model is not recommended for use as a mythbackend.
* This model can not boot directly from usb, you would have still have to use a sd card for your vfat/boot part1
* The original rpi2 v1.1 can experience bandwidth issues with its arm cortex-a7 cpu(s) even overclocked.
     * This is most likely to happen when your using multiple high bandwidth usb devices.
     * This can happen easily if your using an external usb hard disk, even a self powered one.
     * When you connect a external drive and mutliple usb devices they are going to increase your overall TDP
       to where it is not the most effecient use. 
* Though some people use this as a mythbackend/mythfrontend in ultra low power scenarios. Careful consideration
   of its limitations have to be addressed.
  * <b> Image artifacting is one side effect of a stressed/overtaxed rpi2 even in a dedicated backend. This can show up in your recordings and frontends. Limited your usb connected devices to 'help' avoid this. </b>

* * *

## RPI Backend recommendations:

### RPIZERO2 W
* As minimal, you can try a rpizero2.
     * Limit its usb connected devices as they are shared with the cpu.
     * Limit the com scanning
     * Limit bandwidth on its wifi 2.4G as this is shared with the cpu/usb  and its lower powered cpu freq.

### RPI3b+ / RPI3a / RPI2(v1.2/v1.3)     
* These work but still not ideal when using a external usb device.
     * Using its wifi at 5G can overload the usb bus if also using several usb devices.
  
### RPI4
 * The rpi4b works great as a backend.
    * Includes 32bit/64bit userland options.
    * Has dedicated usb 3.0 connected to the cpu(s) via a  pcie link.
    * Has analog audio io port for legacy devices.

### RPI5
 * The rpi5 works awesome as a backend.
     *  Increased cpu frequencies.
     *  Upgraded to RP1 I/O controller.
     *  Has PCIE 2 header for various HAT addon options.
     *  Has a fan header. Use a large heat sink if not using a fan.
     *  Has a power button.
     *  Has RTC and battery connect header.(Effecient time management even in off grid use)
     *  Its a-76 cpu(s) are armv8-a isa, which means it will only boot a 64bit kernel.
   
      * Note you could use the rpi5 with a 64bit kernel and 32bit userland, but this can cause problems in a development system as its cpu can be detected wrongly. For a hybrid 64/32bit RPI5 dev system you would likely want to use cross compiling options or emulate a 32bit environment with an option that lets you specify a arm 32bit host cpu.

* * * 

## RPI Frontend recommendations:

*Anything lower than rpi2 v1.1 will likely not work well. Expect random choppy playback on anything > standard def.
   * You could play with using v4l2 codecs on these to make HD work to some level, but that may disable CC/Subtitles.

* All the rpi can use accelerated GPU decoding and hardware deinterlacing GLSL when using mp4/h264/avc formats,
   RPI4/5 can also do x265/hevc.
   * Lowerend slower devices still may require profile tuning for best playback.
* mpeg2 will be by default using software ffmpeg decoding and can use limited GLSL for deinterlacing(deint)

### RPI2 v1.1
    * Can be VERY difficult to get consistent smooth playback on mpeg2 sources at 1080p.
          * Performance and cpu specific tuning/optimisation required to get decent playback.
          * Wired eth0 recommended.
          * Specific optimized playback profiles.
          * For 1080p Mpeg2 playback OpenGL High Quality(using GLSL Kernel) will not be smooth, try:
                    * OpenGL Normal (GLSL linearblend) profile with 1x deint@ Medium quality.
                    * No 2x deint.
          * Disable "Smooth Transitions" @ 'Setup>Appearance>Theme/Screen Settings>Smooth Transitions>'

### RPIZERO2 / RPI2(v1.2/v1.3) / RPI3a/RPI3b
    * Can be challenging to get consistent smooth playback on mpeg2 sources at 1080p.
          * Performance and cpu specific tuning/optimisation recommended. 
             * RPIZERO2/RPI3A with only 512M ram may require vc4-kms-v3d CMA tuning.
              * Disable "Smooth Transitions" @ 'Setup>Appearance>Theme/Screen Settings>Smooth Transitions>'
          * Specific optimsized opengl playback profile testing recommended for best playback.
               * For 1080p Mpeg2 playback OpenGL High Quality(GLSL Kernel) may not be smooth, try:
                    * OpenGL Normal (GLSL kernel/linearblend) profile with 1x deint@high/medium quality
                    * No 2x deint.

### RPI4/RPI5
     * Smooth playback on most sources.
     * Standard default opengl playback profiles should work with the exception of no 2x deinterlacing.
          * Disable 2x deinterlacing in profiles used.
     * Has GPU accelerated playback for both mp4/h264(avc) and h265(hevc)
     * Can support vulkan.

* * *

### Playback power use examples.*DEVICE ONLY measured, not monitor/tv. 
* Using wifi/eth is not a big difference in power.
* Wifi use is not bad when using a strong AP, but due to possible interferences/spectrum congestion, playback can be predictably better on wired.

* RPI2(v1.1)/RPI3a
    <2W idle and ~3-3.5W or < with playback of 1080p mpeg2

* RPI3b/RPI4
    <3W idle and ~4-4.5W or < with playback of 1080p mpeg2


  
