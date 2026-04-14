# esphome-yardgate
Yard gate controller based on ESP Home with HomeAssistant integration via MQTT

in order to compile and deploy this project you need to get https://esphome.io


```podman run --rm -v "${PWD}":/config --device=/dev/ttyUSB0 -it esphome/esphome run ./yardgate.yml```



BOM

# Deployed on PRODINo ESP32EX board
# https://kmpelectronics.eu/shop/prodino-esp32ex/
# board schematics: https://kmpelectronics.eu/wp-content/uploads/2019/12/ProDINoESP32-ETHv02-Schematic.pdf
# flashing example: https://kmpelectronics.eu/tutorials-examples/prodino-esp32-versions-tutorial/