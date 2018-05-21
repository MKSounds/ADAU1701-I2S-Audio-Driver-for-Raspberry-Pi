# Generic-I2S-Audio-Driver-for-Raspberry-Pi
Generic audio driver to use the I2S interface of the Raspberry Pi for sound output


This repo includes the files to setup the I²S-Interface of the Raspberry Pi to use it as a generic audio output (digital soundcard). All devices which don't need any configuration or initialisation via I²C or SPI can be connected. Possible devices are DACs and DSPs which work at a fixed sample rate (and don't have an ASRC). One of the most pupular devices is the ADAU1701 SigmaDSP from Analog Devices. The following configuration will be explained based on the ADAU1701.

The installation guide is available on my website: https://digital-audio-labs.jimdofree.com/raspberry-pi/generischer-i2s-treiber/.

Settings of the Raspberry Pi
---------------------------------
Codec: master mode (clocks provided from external oscillator)

Format: I2S

BCLK = 2 * 32 * Fs = 64 * Fs = 3.072 MHz

LRCLK = Fs

Settings of the ADAU1701
--------------------------------
Sample rate: 48 kHz

Bit depth: 24 bit

Data format: I2S

MCLK = 256 * Fs = 12.288 MHz

BCLK = 64 * Fs = 3.072 MHz   --> Internal / 16

LRCLK = Fs                   --> Internal / 1024

LRCLK polarity: falling edge (LRCLK = 0 = left channel, LRCLK = 1 = right channel)

BCLK data change: falling edge

Transmission: MSB first

MSB position: delayed by 1 BCLK
