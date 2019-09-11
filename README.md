# Raspberry-Pi-Temperature-Control

## Temperatur der CPU ermitteln
``vcgencmd measure_temp``

## LEDs und Lüfter in Abhängigkeit der Temperatur schalten
temp_ampel.sh
```
#!/bin/sh

#echo "2">/sys/class/gpio/export
#echo "14">/sys/class/gpio/export
#echo "15">/sys/class/gpio/export
#echo "18">/sys/class/gpio/export

echo "out">/sys/class/gpio/gpio2/direction
echo "out">/sys/class/gpio/gpio14/direction
echo "out">/sys/class/gpio/gpio15/direction
echo "out">/sys/class/gpio/gpio18/direction

TEMP=`vcgencmd measure_temp | cut -c6,7`

if [ $TEMP -le 40 ]
then
echo "0">/sys/class/gpio/gpio2/value
echo "1">/sys/class/gpio/gpio14/value
echo "0">/sys/class/gpio/gpio15/value
echo "0">/sys/class/gpio/gpio18/value
elif [ $TEMP -ge 40 ] && [ $TEMP -lt 45 ]
then
echo "1">/sys/class/gpio/gpio2/value
echo "1">/sys/class/gpio/gpio14/value
echo "1">/sys/class/gpio/gpio15/value
echo "0">/sys/class/gpio/gpio18/value
elif [ $TEMP -ge 45 ] && [ $TEMP -lt 65 ]
then
echo "1">/sys/class/gpio/gpio2/value
echo "0">/sys/class/gpio/gpio14/value
echo "1">/sys/class/gpio/gpio15/value
echo "0">/sys/class/gpio/gpio18/value
elif [ $TEMP -ge 65 ] && [ $TEMP -lt 70 ]
then
echo "1">/sys/class/gpio/gpio2/value
echo "0">/sys/class/gpio/gpio14/value
echo "1">/sys/class/gpio/gpio15/value
echo "1">/sys/class/gpio/gpio18/value
else
echo "1">/sys/class/gpio/gpio2/value
echo "0">/sys/class/gpio/gpio14/value
echo "0">/sys/class/gpio/gpio15/value
echo "1">/sys/class/gpio/gpio18/value
fi
```

## Dienst einrichten
### init.d

Link in init.d erstellen

``ln -s /root/tmpstrg/temp_ampel_system.sh /etc/init.d/temp_ampel``

#### Start-/Stopscript
temp_ampel_system.sh
```
#! /bin/bash

### BEGIN INIT INFO
# Provides:          temp_ampel
# Required-Start:    $start
# Required-Stop:     $shutdown
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: temperature ampel service
# Description:       Run temperature ampel service
### END INIT INFO

# Carry out specific functions when asked to by the system
case "$1" in
  start)
    echo "Starting temerature ampel..."
    sudo bash -c 'cd /root/tmpstrg && ./temp_ampel_init.sh'
    ;;
  stop)
    echo "Stopping temperature ampel..."
    sudo bash -c 'cd /root/tmpstrg && ./temp_ampel_down.sh'
    sleep 2
    ;;
  *)
    echo "Usage: /etc/init.d/temp_ampel {start|stop}"
    exit 1
    ;;
esac

exit 0
```
#### Systemstart
temp.ampel_init.sh
```
#!/bin/sh
echo "2">/sys/class/gpio/export
echo "14">/sys/class/gpio/export
echo "15">/sys/class/gpio/export
echo "18">/sys/class/gpio/export

echo "out">/sys/class/gpio/gpio2/direction
echo "out">/sys/class/gpio/gpio14/direction
echo "out">/sys/class/gpio/gpio15/direction
echo "out">/sys/class/gpio/gpio18/direction

echo "0">/sys/class/gpio/gpio2/value
echo "1">/sys/class/gpio/gpio14/value
echo "0">/sys/class/gpio/gpio15/value
echo "0">/sys/class/gpio/gpio18/value
```

#### Shutdown
temp_ampel_down.sh
```
#!/bin/sh

echo "0">/sys/class/gpio/gpio2/value
echo "0">/sys/class/gpio/gpio14/value
echo "0">/sys/class/gpio/gpio15/value
echo "0">/sys/class/gpio/gpio18/value
```

#### Runlevel updaten
update-rc.d temp_ampel defaults

## Minütlich ausführen
``crontab -e``
```
*/1 * * * * /root/tmpstrg/./temp_ampel.sh
```
