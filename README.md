# lora-tracker

This repo contains code for building lora based trackers for Burning Man 2024.

## Platforms

### Lilygo T-Beam SoftRF or Meshtastic

These are two versions of what appears to largely be the same product with the difference being that the Meshtastic version includes a OLED display. I'm not sure there are other differences.

The T-Beam product had a 1.0 version but currently the latest version seems to be v1.1 or v1.2. The difference is the PMU. V1.1 PMU is AXP192. V1.2 PMU is AXP2101. What I have is v1.2 with AXP 2101

After SoftRF/Meshtastic there are SMA Base or IPEX base variants. This is the connector for the Lora antenna. SMA is the bigger "stick through the case" connector and IPEX is the small surface mount connector. Both variants have a GPS antenna connected via IPEX and a WiFI/Bluetooth 3-D on-board antenna with an IPEX connector present for the 2.4 Ghz antenna as well. 

[Lilygo T-Beam SoftRF Product Page](https://www.lilygo.cc/products/t-beam-softrf?variant=43170158477493)

That page includes the following image about v1.0 that I think is mostly correct-ish for 1.2

[Pin Usage](images/lilygo-pin-diagram-t-beam_2048x2048.jpg)

The MCU is an ESP32-DOWDQ6 (regular esp32, not S or C series), with IPS6404 PSRAM (Not sure how big), and 4MB of flash in a W25Q32. As always the PSRAM and Flash share the high bandwidth SPI port.

#### IPS6404 PSRAM

    ESP32 -> IPS6404
    .
    GPIO16 -> CS#
    (SD_DATA_0) SDO/SD0 -> SO/SIO1
    (SD_DATA_1) SDI/SD1 -> SI/SIO0
    (SD_DATA_2) SHD/SD2 -> SIO3
    (SD_DATA_3) SWP/SD3 -> SIO2
    GPIO17 -> SRAM_CLK -> SCLK

Power from VDD_SDIO

I'm not sure why SD0, SD1 from the ESP32 are swapped or why SD2, SD3 are also swapped. Can't say I care.

#### W25Q32 4MB Flash

    ESP32 -> W25Q32
    .
    (SD_CMD) SCS/CMD -> /CS
    (SD_DATA_0) SDO/SD0 -> DO
    (SD_DATA_1) SDI/SD1 -> DI
    (SD_DATA_2) SHD/SD2 -> /HOLD
    (SD_DATA_3) SWP/SD3 -> /WP
    (SD_CLK) SCK/CLK -> FLASH_CLK -> CLK

Power from VDD_SDIO

This seems to use "hold" and "WP" signals, whatever those are. The clock signals between flash and psram are connected as shown on the schematic but there is this little 0 value resistor section where these can clearly be swapped around. Again, not sure I care other than not trying to use these elsewhere.

#### LoRA Chip (915Mhz = Sx1262)

The LoRA chip is a LilyGo module, ubt it seems like the 915 MHz US version is the Sx1262 internally. However the Sx1276 (which I think is uncommon) does supposedly cover the same frequency range. Oh, it's selectable when ordering. Great. Hopefully they have different SPI addresses...

    ESP32 -> TTGOLORA
    .
    (GPIO23) IO23 -> IRQ/RESET
    (GPIO18) IO18 -> NSS/SEL
    (GPIO5) IO5 -> SCK
    (GPIO27) IO27 -> MISI/SDI
    (GPIO19) IO19 -> MISO/SDO
    (GPIO26) IO26 -> IO0
    (GPIO33) IO33 -> HPDIO1 -> IO1
    (GPIO32) IO32 -> HPDIO2 -> IO2

Not connected are 3 signals (IO3/RX), (IO4/TX), and (SDN/IO5). So I presume that is a UART interface and don't know what SDN is.

#### GPS NEO-6M/M8N

    ESP32 -> NEO-6M/M8N
    .
    RXD1 -> RXD1
    TXD1 -> TXD1
    IO37 -> (LED_G2) -> TIME_PULSE
    .
    D1- -> USB_DM
    D1+ -> USB_DP

Power is GPS_VDD

There is also a 24AA32A 32k EEPROM and a battery to support GPS hot start.

It seems like there is a connection to the USB data lines. So does this show up as a USB device???

#### PMY AXP2101

    ESP32 -> AXP2101
    .
    (GPIO35) IO35 -> PMU_IRQ
    (GPIO22) IO22 -> SCK
    (GPIO21) IO21 -> SDA

    LORA_VCC -> ALD02
    VSYS -> ALDOIN
    GPS_VDD -> ALDO3

In addition to managing charging it looks like LORA and GPS can be totally powered off or on via the PMU, which I guess is cool or whatever.

The schematic seems to say the blue LED is tied to charge and red to the GPS pulse, but this seems to be the other way around in reality.

#### USB Serial CP2104

    ESP32 -> CP2104
    .
    RST/EN -> DTR -> DTR
    U0RXD -> TXD
    U0TXD -> RXD
    (GPIO0) IO0 -> RTS -> RTS

Power is VIO33

The RTS/DTR might be listed backwards here. I'm sure they are the regular way round like they should be. The RTS/DTR signals go through a UMH3N, presumably a tranistor pack. 


## Ported Libraries

This is the official [github for the LoRa series](https://github.com/Xinyuan-LilyGO/LilyGo-LoRa-Series) which contain an Arduino project for various T-Beam hardware versions as well as other hardware. This project includes a number of libraries that either support ESP-IDF or need porting.

In a number of cases the official arduino library needs to be consulted for options or operation order.

[XPowersLib for PMU](https://github.com/lewisxhe/XPowersLib) - Seems to support ESP-IDF directly including with an example




----------

Look into this for direct access to GPS module!
https://github.com/semuconsulting/PyGPSClient


