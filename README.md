 # MAVFWD
Added stream parsing with official mavlink library.
mavfwd is a utility for broadcasting a mavlink telemetry stream between wifibroadcast and uart devices, for organizing one- or two-way telemetry between UAV and ground station. mavfwd is included in the OpenIPC FPV firmware as a lighter alternative to mavlink routerd and is usually used on the air part (camera).
Also, mavfwd can monitor the mavlink RC _CHANNELS package, and call the script `channels.sh` (located at /usr/bin or /usr/sbin) passing the channel number and its value to it when changing as $1 and $2 parameters. An example of channels.sh is present.
This allows for controlling the camera via the Remote Control Transmitter
```
Usage: mavfwd [OPTIONS]
-m --master      Local MAVLink master port (%s by default)
-b --baudrate    Serial port baudrate (%d by default)
-o --out         Remote output port (%s by default)
-i --in          Remote input port (%s by default)
-c --channels    RC Channel to listen for commands (0 by default) and call channels.sh
-w --wait        Delay after each command received(2000ms defaulr)
-a --aggregate   Aggregate packets in frames (1 no aggregation, 0 no parsing only raw data forward) (%d by default) 
-f --folder      Folder for file mavlink.msg (default is current folder)
-t --temp        Inject SoC temperature into telemetry
-v --verbose     Display each packet, default not       
--help         Display this help
```
Example :

```mavfwd --master /dev/ttyAMA0 --baudrate 115200 --out 192.168.1.20:14550 -c 7 -w 3000 -a 10 -t -f /tmp/```
Will read on the first UART with baudrade 115200 and will listen for values in RC channel 7 that come from the Remote Control via Flight Controller.
Every time the value is changed with more than 5% the bash script ```channels.sh {Channel} {Value}`````` will be started with params.
The script then can check the Value param (usually between 1000 and 2000) and do the tasks needed - reconfigure encoder, switch IR mode, restart WiFi, change MCS index. etc.
To protect the system from overloading (when rotating a pot on the Tx ), the script will not be started again for 3000ms.
Packets will be aggregated in chunks of 10 into one UDP frame. A MAVLINK_MSG_ID_ATTITUDE from the FC will flush the buffer. 
This way the OSD will be updated with the same rate and no lag will be added.
 -a 15 : will flush the cached messages into one UDP packet after count of message reaches 15
 -a 1024 : will flush the cached messages into one UDP packet total length of all message reaches 1024 bytes
in both cases the buffer will be flushed if there are at least 3 packets and a MAVLINK_MSG_ID_ATTITUDE is received.
Temperature will be read from the board and will be injected into the mavlink stream each second via MAVLINK_MSG_ID_RAW_IMU 27 message.

Option to send text from the cam. The file mavlink.msg in {tempfolder} is monitored and when found, all data from it are send as plain text message via   MAVLINK_MSG_ID_STATUSTEXT 253 to the ground station and the file is deleted.
Sample:
echo "Huston, this is IPC" >/tmp/mavlink.msg

killall -usr1 mavfwd   # will send text message to ground station even no data are received in the serial port, can be used to test Camera to Ground connection.



