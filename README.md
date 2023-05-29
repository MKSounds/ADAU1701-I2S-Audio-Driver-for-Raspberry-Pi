# ADAU1701-I2S-Audio-Driver-for-Raspberry-Pi
Generic audio driver to use the I2S interface of the Raspberry Pi for sound output to a dsp or any other I2S ot TDM8 device.

This repo includes the files to setup the I²S-Interface of the Raspberry Pi to use it as a generic audio output (digital soundcard). All devices which don't need any configuration or initialisation via I²C or SPI can be connected. Possible devices are DACs and DSPs (be aware, that the dsp must have an ASRC if the RPi is the clock master). One of the most pupular devices is the ADAU1701 SigmaDSP from Analog Devices. The following configuration will be explained based on the ADAU1701. The configurations are available as stereo I2S format and as 8 channel TDM format (only two channels used), both either as clock slave or clock master.

The testproject is a Sigma Studio project file to test the digital IO-ports of the ADAU1701. Therefore a sine wave is generated, output on MP6, fed back to MP0 and output to the DAC. The aim is to check whether the config of the pins and the wireing of the dsp are correct.


Installation guide Raspbian/RPi OS
----------------------------------
Install git:
> sudo apt-get install git

Clone files:
> git clone https://github.com/MKSounds/ADAU1701-I2S-Audio-Driver-for-Raspberry-Pi.git

Copy overlay file to overlays directory (choose the one you want to use):
> sudo cp ADAU1701-I2S-Audio-Driver-for-Raspberry-Pi/generic_audio_out_i2s_slave.dtbo /boot/overlays

Relink the overlay to the device tree: Therefore config.txt has to be modified. Open the file with:
> sudo nano /boot/config.txt

Search for the following line and comment it with a hash (navigation with arrow keys):
#dtparam=audio=on

Add the following line at the end of the file:
dtoverlay=generic_audio_out_i2s_slave (choose the variant you want to use)

Close the editor by pressing STRG+X and save the changes with y.
Reboot:
> sudo reboot

If you want to play files which don't use 48 kHz sample rate, you have to resample the audio data. Therefore a ALSA config file will be used, which will affect the whole audio output of the system.
Create file and open it (if it's already existing and there is code in it, you have to integrate the new code or delete the existing):
> sudo nano ~/.asoundrc
> 
Copy and paste the following lines (paste by right clicking):

pcm.InterpolatedOutput {
        type plug
        slave {
                pcm "hw:0,0"
                format S24_LE
                rate 48000
        }
}
pcm.!default InterpolatedOutput

Close the editor by pressing STRG+X, save the changes with y and do a reboot.

Installation guide Volumio 3.x
----------------------------------
Install git (should be included in Volumio):
> sudo apt-get install git

Clone files:
> git clone https://github.com/MKSounds/ADAU1701-I2S-Audio-Driver-for-Raspberry-Pi.git

Copy overlay files to overlays directory:
> sudo cp ADAU1701-I2S-Audio-Driver-for-Raspberry-Pi/generic_audio_out_i2s_slave.dtbo /boot/overlays
> sudo cp ADAU1701-I2S-Audio-Driver-for-Raspberry-Pi/generic_audio_out_i2s_master.dtbo /boot/overlays
> sudo cp ADAU1701-I2S-Audio-Driver-for-Raspberry-Pi/generic_audio_out_tdm8_slave.dtbo /boot/overlays
> sudo cp ADAU1701-I2S-Audio-Driver-for-Raspberry-Pi/generic_audio_out_tdm8_master.dtbo /boot/overlays

Integrate the ADAU1701 soundcard in the dropdown menue of Volumio. Open the file dacs.json with:
> sudo nano /volumio/app/plugins/system_controller/i2s_dacs/dacs.json

Add (copy&paste) the following line as new first device in the list of available I²S devices:
{"id":"generic_audio_out_i2s_slave","name":"Generic I2S Driver Slave","overlay":"generic_audio_out_i2s_slave","alsanum":"2","alsacard":"Slave","mixer":"","modules":"","script":"","needsreboot":"yes"},
{"id":"generic_audio_out_i2s_master","name":"Generic I2S Driver Master","overlay":"generic_audio_out_i2s_master","alsanum":"2","alsacard":"Master","mixer":"","modules":"","script":"","needsreboot":"yes"},
{"id":"generic_audio_out_tdm8_slave","name":"Generic TDM Driver Slave","overlay":"generic_audio_out_tdm8_slave","alsanum":"2","alsacard":"Slave","mixer":"","modules":"","script":"","needsreboot":"yes"},
{"id":"generic_audio_out_tdm8_master","name":"Generic TDM Driver Master","overlay":"generic_audio_out_tdm8_master","alsanum":"2","alsacard":"Master","mixer":"","modules":"","script":"","needsreboot":"yes"},

Close the editor by pressing STRG+X and save the changes with y. Reboot:
> sudo reboot

Now you can select the Generic I2S Driver as I2S DAC in the user interface of Volumio.
If needed, you can use a software volume control (in Volume Options) as far as you don't want to regulate the volume on the dsp.
The resampling of audio material which doesn't have a sample rate of 48 kHz (or the sample rate the clocks are incoming, if you use the RPi as clock slave) can be set in the "Audio Resampling" section. The resampling works for playing local media like flash drives (network storage might work as well).
AirPlay users
There's still a problem when using AirPlay, because Shairport-sync sends its data directly to the hardware. As a consequence, the resampling of the Volumio system doesn't affect the AirPlay output.
Sound output via AirPlay doesn't work that easy. The playback is badly stuttering (re-syncing).
The only solution is to exchange the oscillator of the dsp board with a model which outputs a 11,2896 MHz clock. Then the dsp must be configured to 44,1 kHz sample rate. The whole system is working on 44,1 kHz clock base now.
There are no changes in the raspberry driver necessary because the multiplication factor of 48 and 44,1 kHz is the same.
Set the internal resampling of volumio to 44,1 kHz and 24/32 bit to ensure that local material will be resampled for the new system sample rate.

Wireing Raspberry to ADAU1701
-------------------------------------
Pin 12 (PCM_CLK)  -  MP5 and MP11

Pin 35 (PCM_FS)  -  MP4 and MP10

Pin 40 (PCM_DOUT)  -  MP0 (or MP1, MP2, MP3)

Pin 39 (GND)  -  any GND-pin

Settings of the Raspberry Pi (setup from the driver)
----------------------------------------------------
Codec: master mode (clocks provided from external oscillator)

Format: I2S, 1 bclk cycle delay

BCLK = 2 * 32 * Fs = 64 * Fs = 3.072 MHz (I2S)
BCLK = 8 * 32 * Fs = 64 * Fs = 12.288 MHz (TDM)

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
