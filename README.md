Pi-FM-RDS
=========


## FM-RDS transmitter using the Raspberry Pi

This program generates an FM modulation, with RDS (Radio Data System) data generated in real time. It can include monophonic or stereophonic audio.

It is based on the FM transmitter created by [Oliver Mattos and Oskar Weigl](http://www.icrobotics.co.uk/wiki/index.php/Turning_the_Raspberry_Pi_Into_an_FM_Transmitter), and later adapted to using DMA by [Richard Hirst](https://github.com/richardghirst). Christophe Jacquet adapted it and added the RDS data generator and modulator. The transmitter uses the Raspberry Pi's PWM generator to produce VHF signals.

It is compatible with both the Raspberry Pi 1 (the original one) and the Raspberry Pi 2.

![](doc/vfd_display.jpg)

PiFmRds has been developed for experimentation only. It is not a media center, it is not intended to broadcast music to your stereo system. See the [legal warning](#warning-and-disclaimer).

## How to use it?

Pi-FM-RDS, depends on the `sndfile` library. To install this library on Debian-like distributions, for instance Raspbian, run `sudo apt-get install libsndfile1-dev`.

Pi-FM-RDS also depends on the Linux `rpi-mailbox` driver, so you need a recent Linux kernel. The Raspbian releases from August 2015 have this.

**Important.** The binaries compiled for the Raspberry Pi 1 are not compatible with the Raspberry Pi 2, and conversely. Always re-compile when switching models, so do not skip the `make clean` step in the instructions below!

Clone the source repository and run `make` in the `src` directory:

```bash
git clone https://github.com/vskmanikandan/PiFM.git
cd PiFmRds/src
make clean
make
```

Then you can just run:

```
sudo ./pi_fm_rds
```

This will generate an FM transmission on 107.9 MHz, with default station name (PS), radiotext (RT) and PI-code, without audio. The radiofrequency signal is emitted on GPIO 4 (pin 7 on header P1).


You can add monophonic or stereophonic audio by referencing an audio file as follows:

```
sudo ./pi_fm_rds -audio sound.wav
```

To test stereophonic audio, you can try the file `stereo_44100.wav` provided.

The more general syntax for running Pi-FM-RDS is as follows:

```
pi_fm_rds [-freq freq] [-audio file] [-ppm ppm_error] [-pi pi_code] [-ps ps_text] [-rt rt_text]
```

All arguments are optional:

* `-freq` specifies the carrier frequency (in MHz). Example: `-freq 107.9`.
* `-audio` specifies an audio file to play as audio. The sample rate does not matter: Pi-FM-RDS will resample and filter it. If a stereo file is provided, Pi-FM-RDS will produce an FM-Stereo signal. Example: `-audio sound.wav`. The supported formats depend on `libsndfile`. This includes WAV and Ogg/Vorbis (among others) but not MP3. Specify `-` as the file name to read audio data on standard input (useful for piping audio into Pi-FM-RDS, see below).
* `-pi` specifies the PI-code of the RDS broadcast. 4 hexadecimal digits. Example: `-pi FFFF`.
* `-ps` specifies the station name (Program Service name, PS) of the RDS broadcast. Limit: 8 characters. Example: `-ps RASP-PI`.
* `-rt` specifies the radiotext (RT) to be transmitted. Limit: 64 characters. Example: `-rt 'Hello, world!'`.
* `-ctl` specifies a named pipe (FIFO) to use as a control channel to change PS and RT at run-time (see below).
* `-ppm` specifies your Raspberry Pi's oscillator error in parts per million (ppm), see below.

By default the PS changes back and forth between `Pi-FmRds` and a sequence number, starting at `00000000`. The PS changes around one time per second.


### Clock calibration (only if experiencing difficulties)

The RDS standards states that the error for the 57 kHz subcarrier must be less than ± 6 Hz, i.e. less than 105 ppm (parts per million). The Raspberry Pi's oscillator error may be above this figure. That is where the `-ppm` parameter comes into play: you specify your Pi's error and Pi-FM-RDS adjusts the clock dividers accordingly.

In practice, I found that Pi-FM-RDS works okay even without using the `-ppm` parameter. I suppose the receivers are more tolerant than stated in the RDS spec.

One way to measure the ppm error is to play the `pulses.wav` file: it will play a pulse for precisely 1 second, then play a 1-second silence, and so on. Record the audio output from a radio with a good audio card. Say you sample at 44.1 kHz. Measure 10 intervals. Using [Audacity](http://audacity.sourceforge.net/) for example determine the number of samples of these 10 intervals: in the absence of clock error, it should be 441,000 samples. With my Pi, I found 441,132 samples. Therefore, my ppm error is (441132-441000)/441000 * 1e6 = 299 ppm, **assuming that my sampling device (audio card) has no clock error...**


### Changing PS, RT and TA at run-time

You can control PS, RT and TA (Traffic Announcement flag) at run-time using a named pipe (FIFO). For this run Pi-FM-RDS with the `-ctl` argument.

Example:

```
mkfifo rds_ctl
sudo ./pi_fm_rds -ctl rds_ctl
```

Then you can send “commands” to change PS, RT and TA:

```
cat >rds_ctl
PS MyText
RT A text to be sent as radiotext
TA ON
PS OtherTxt
TA OFF
...
```

Every line must start with either `PS`, `RT` or `TA`, followed by one space character, and the desired value. Any other line format is silently ignored. `TA ON` switches the Traffic Announcement flag to *on*, any other value switches it to *off*.


## Warning and Disclaimer

PiFmRds is an **experimental** program, designed **only for experimentation**. It is in no way intended to become a personal *media center* or a tool to operate a *radio station*, or even broadcast sound to one's own stereo system.

In most countries, transmitting radio waves without a state-issued licence specific to the transmission modalities (frequency, power, bandwidth, etc.) is **illegal**.

Therefore, always connect a shielded transmission line from the RaspberryPi directly
to a radio receiver, so as **not** to emit radio waves. Never use an antenna.

Even if you are a licensed amateur radio operator, using PiFmRds to transmit radio waves on ham frequencies without any filtering between the RaspberryPi and an antenna is most probably illegal because the square-wave carrier is very rich in harmonics, so the bandwidth requirements are likely not met.

I could not be held liable for any misuse of your own Raspberry Pi. Any experiment is made under your own responsibility.


### References

* [EN 50067, Specification of the radio data system (RDS) for VHF/FM sound broadcasting in the frequency range 87.5 to 108.0 MHz](http://www.interactive-radio-system.com/docs/EN50067_RDS_Standard.pdf)


© Released under the GNU GPL v3.
