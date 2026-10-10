# Setting Up Arduino/ESP32/ESP8266/ETC.

## ESP32 Setup

NOTE: **Libraries** and **Board Managers** are two different things

1. Install `esp32` by **Espressif Systems** in the board manager.
2. `Tools` > `Board` > `esp32` > Select `ESP32 Dev Module`.

## Allowing physical port in Ubuntu

```bash
# Display All Device Files (ports are among these)
ls /dev/*

# Display All Connected Arduino Ports
ls /dev/ttyACM*

# Display All Connected USB-to-Serial Ports (most ESP32 and ESP8266 boards show up here because they use a USB-to-serial chip such as the CP210x or CH340)
ls /dev/ttyUSB*

# Allow A Port (Example : /dev/ttyACM0)
# NOTE: this is temporary; the permission resets when the device is unplugged or the machine reboots
sudo chmod a+rw /dev/ttyACM0

# Permanent alternative: add your user to the `dialout` group (log out and back in afterwards)
sudo usermod -aG dialout $USER
```
