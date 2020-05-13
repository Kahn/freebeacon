# Introduction

FreeDV beacon:

+ Listens for FreeDV signals, then transmits a reply.
+ Supports FreeDV 1600/700C/700D modes.
+ Places the received signal files on a web server.
+ Requires a Linux machine with a sound card and RS232-PTT (or Hamlib CAT) interface to your radio.
+ Just one sound card is required.
+ Can run on machines as small as a Raspberry Pi.

When a "trigger" string is detected in the rx FreeDV text message (e.g. "hello beacon", or
the beacon callsign), the beacon will transmit a signal report back to you.

It requires a `txfilename` wave file to transmit, e.g. some one saying "Hi, I am a FreeDV
beacon blah blah".  The signal report is encoded into the transmit text message.  Make the
wave file long enough so that the signal report is repeated a few times, say 30 seconds.
Transmit will stop when the `txfilename` wave file is played once.

Freebeacon saves the received audio from the radio AND the decoded audio as wavefiles.
Use `wavefilewritepath` to specify where they are written.  The file name is a date and time stamp.
The length is limited to 60 seconds. If you set `wavefilewritepath` to a listable webserver
directory the files will be available for download on the Web.  To avoid filling your file
system write a cron job to clean these files up once a day.

If your input audio device is stereo note we only listen to the left channel.

If you have a RS232 serial port (specified with "-c") RTS and DTR is raised on transmit
to key your transmitter, and lowered for receive.

If you are using Raspberry Pi you can use one of the GPIOs for PTT control of your
transmitter using the `--rpigpio` option.  You need to use the BCM GPIO number so `--rpigpio 11`
uses pin 23 of the GPIO connector.

Note FreeDV 700C doesn't have a text channel, so we just record the received file
without triggering a Tx response.

A whole lot of code was lifted from freedv-dev for this program.

## Credits

+ David Rowe, John Nunan
+ Richard Shaw - CMake
+ Bob Wisdom - Hamlib, FreeDV 700C & 700D modes
+ Initially developed Dec 2015
+ Refactored for GitHub, added Hamlib, FreeDV 700C & 700D modes in 2020

## Building and Installation

### Dependencies

Install the build tools.

1. Debian based operating systems.

   ```sh
   sudo apt install git cmake sox libsamplerate0-dev portaudio19-dev libsndfile1-dev libhamlib-dev
   ```

### Building Method 1 - Install codec 2 libraries on your system

1. Build and install codec2.

    ```sh
    git clone https://github.com/drowe67/codec2.git
    cd codec2
    mkdir build_linux
    cd build_linux
    cmake ../
    make
    sudo make install
    ```

    Note: On my Ubuntu 18 and RPi I had to add an extra search path to the
    ld.conf.d directory to match the path the codec2 .so was installed
    in.

1. Check it has found libcodec2.so using:

    ```sh
    ldconfig -v | grep codec2
    ```

1. Build FreeBeacon:

    ```sh
    cd ~
    git clone https://github.com/drowe67/freebeacon.git
    cd freebeacon
    mkdir build_linux
    cd build_linux
    cmake ../
    make
    ```

### Building Method 2 - without system wide codec 2 installation

If you don't want to have codec2 installed system wide use this
process.

1. Build codec2 without installing.

   ```sh
   git clone https://github.com/drowe67/codec2.git
   cd codec2 && mkdir build_linux && cd build_linux
   cmake ../ && make
   ```

1. Instruct freebeacon cmake to use local codec2 directory.

     ```sh
    cd ~
    git clone https://github.com/drowe67/freebeacon.git
    cd freebeacon && mkdir build_linux cd build_linux
    cmake -DCODEC2_BUILD_DIR=~/codec2/build_linux ..
    make
    ```

### Testing

Plug in your USB sound card and USB RS232 devices.  Use alsamixer
to adjust levels on your sound card. F6 lets you select sound cards

Testing your PTT by making it jump straight into tx mode:

  ```sh
    ./freebeacon -c /dev/ttyUSB1 --txfilename ~/codec2-dev/wav/vk5qi.wav --dev 4 -v --trigger hello -t
  ```

Testing sound cards on RPi:

   ```sh
   $ arecord -l

     **** List of CAPTURE Hardware Devices ****
     card 1: Audio [RIGblaster Advantage Audio], device 0: USB Audio [USB Audio]
     Subdevices: 1/1
     Subdevice #0: subdevice #0

   $ arecord -D hw:1,0 -f S16_LE -r 48000 test.wav
   $ aplay test.wav
   ```

As a first step try playing freebeacon_test.wav from another PC into the freebeacon
machine input, this file has the trigger string "hello" in the txt msg.

### Usage

Usage:

  ```sh
    ./freebeacon -h
  ```

List sound devices

  ```sh
    ./freebeacon -l
  ```

Example usage:

  ```sh
    ./freebeacon -c /dev/ttyUSB1 --txfilename ~/codec2-dev/wav/vk5qi.wav --dev 4 -v --trigger hello
  ```
