---
title: 'Enable Wifi On/Off button on Netgear WNDR3800 in Gargoyle router firmware'
categories:
  - Science & Technology
  - Router
tags:
  - Router
abbrlink: 1348488357
date: 2016-05-10 07:20:00
---

Recently, I upgraded my home router's firmware to Gargoyle instead of the Openwrt before. However, after the installation process, I surprisedly found that the Wifi On/Off button stopped working. This button is very convenient for me to turn wireless function off and on before and after the night sleep. I can't lose it.

<!-- more -->

1. Remote login your router via ssh.

   ```bash
   ssh root@x.x.x.x
   ```

2. Open and edit `/etc/hotplug.d/button/00-button` file, add the following line shown below. You have to be sure that the line is exactly added before the last brace.

   ```bash
       [ "$ACTION" = "$action" -a "$BUTTON" = "$button" -a -n "$handler" ] && {
           [ -z "$min" -o -z "$max" ] && eval $handler
           [ -n "$min" -a -n "$max" ] && {
               [ $min -le $SEEN -a $max -ge $SEEN ] && eval $handler 
           }
       }
       # Add the following line here.
       logger the button was $BUTTON and the action was $ACTION
   }
   ```

3. Press the Wifi On/Off button on the front panel, then execute the following command to check which label this button is used.

   ```bash
   # logread
   ```

   At the end of what this command dumped, you could find something like:

   ```
   user.notice.root: the button was XXX and the action was pressed/released
   ```

   `XXX` is the label of the button you just pressed.

   For my WNDR3800, it should be `BTN_2`.

4. Create a new shell scrip `/sbin/wifitoggle`. Append the following content to it.

   ```bash
   #!/bin/sh

   STATUS=`wifi status | grep -m 1 up | sed -e 's/^[ \t]*//' -e 's/"up": //'`

   if [ "$STATUS" == "true," ]; then
      wifi down
      logger Wifi button pressed, wifi going down
   else
      wifi up
      logger Wifi button pressed, wifi going up
   fi
   ```

   This simple shell script will toggle the wireless function on and off by using the `wifi` command.

5. Grant the execute privilege to the script:

   ```bash
   chmod a+x /sbin/wifitoggle
   ```

6. Bind this script with Wifi button.

   Execute the commands step by step.

   ```bash
   uci set system.wifi_toggle=button
   uci set system.wifi_toggle.button=BTN_2
   uci set system.wifi_toggle.action=pressed
   uci set system.wifi_toggle.handler=/sbin/wifitoggle
   uci commit system
   ```

7. Test it!

---

### ¶ The end

