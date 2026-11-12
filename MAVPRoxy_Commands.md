



## MAVProxy Cheatsheet


| Command | Description |
| :--- | :--- |
| `sudo mavproxy.py --master=/dev/ttyTHS1 --baudrate=57600` | Connect Jetson UART to Pixhawk telemetry port |
| `sudo mavproxy.py --master=/dev/ttyACM0 --baudrate=115200 --show-errors` | Test direct USB connection to Pixhawk; adjust device path as needed |
| `rc N PWM` | Set RC channel N override to PWM (PWM = 0 disables override) |
| `watch HEARTBEAT` | Print each heartbeat message as it arrives from the Pixhawk (confirms the link is alive) |
| `watch off` | Stop watching messages |
| `status HEARTBEAT` | Show the latest received HEARTBEAT message |
| `link list` | List all links |
| `link set N` | Set link N to primary |
| `link add X` | Add new link |
| `link remove N` | Remove link |
| `wp list` | Download and show waypoints |
| `fence list` | Download and show geofence |
| `fence enable` | Enable geofence |
| `fence disable` | Disable geofence |
| `arm throttle` | Arm vehicle |
| `disarm` | Disarm vehicle |
| `disarm force` | Force vehicle disarm (heli) |
| `auto` | Engage auto mode |
| `wp set N` | Set current auto waypoint to N |
| `setspeed N` | Override auto speed to N m/s |
| `param show X` | Show current value of parameter X |
| `param set X N` | Set parameter X to value N |
| `param download` | Download parameter definitions from ArduPilot website |
| `param help X` | Display definition of parameter X |
| `mode loiter` | Engage loiter mode |
| `mode rtl` | Engage rtl mode |
| `mode manual` | Engage manual mode |
| `mode guided LAT LON ALT` | Engage guided mode and move to specified location |
| `relay set N [0\|1]` | Set relay N to open or closed |
| `servo set N PWM` | Override servo N to PWM value |
| `terrain check LAT LON` | Get terrain height at specified location |
| `formatsdcard` | Formats SD card |
| `watch HEARTBEAT` | Hearbeat message should print about 1hz. Purpose is to verify connection between companion computer and Flight Controller |