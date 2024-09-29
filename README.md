# Roc Toolkit Receiver to PulseAudio relay
Service that streams real-time audio from a Roc Sender to a PulseAudio server.

![roc-pulse-relay overview](https://github.com/bjaan/roc-pulse-relay/blob/main/main.png?raw=true)

The Roc Toolkit's Sender will allow you to stream stable CD-quality audio over the network. This service will stream that audio to your PulseAudio server.

This little Docker container will start a service that will allow you to connect any digital audio source from the Roc Toolkit, this can be as simple as its *roc-send*, that streams an audio file over the network, or a virtual audio device that support the Roc Toolkit real-time audio streaming protocol.

Or, use this service when you've set-up a Virtual Speakers as a Roc Virtual Audio Device in macOS. This can be a real Mac, where you need to play audio through another device, or this can be a macOS running in a VM-host (like in QEMU or VirtualBox) to connect to your speakers over the PulseAudio service that is running on your host device.  

The PulseAudio service can be run on a Raspberry Pi, another Linux, or a Windows PC as a Windows Service, for Roc-Steady Sound!

# Installation

These assume that you already have the following things configured on your (host) machine:
* PulseAudio server running and working audio on your host, test with the _pacat < /dev/urandom_ command if that is working, it should let you hear static over your speakers.
* Docker installed on the same machine with PulseAudio installed, either under WSL 2 on Windows or directly on Linux.

For the example use, mentioned above, to have working Virtual Speakers working on macOS, the _Roc Virtual Audio Device for macOS_ device set-up in macOS - ready to connect, see below for [instructions](#-Installation-of-the-Roc-Virtual-Audio-Device-for-macOS)

And of course, some knowledge how to work with the terminal, GitHub repositories, and Docker.

1. check out the the GitHub repository: run
```sh
git clone https://github.com/bjaan/roc-pulse-relay
```
2. for the AMD64 architecture (modern PC hardware): change directory in to the AMD64 _Dockerfile_ folder (TODO other architectures)
```sh
cd roc-pulse-relay\roc-pulse-relay-amd64
```
3. run the Docker Compose Build command, this will use both the  _Dockerfile_ and the _compose.yaml_ to build a new Docker Image, called _bjaan/roc-pulse-relay-amd64_
```sh
docker compose build
```
4. run the Docker Compose Build command, this will create a Docker Container based on the _bjaan/roc-pulse-relay-amd64_ Docker Image, and start it up
```sh
docker compose up -d
```
5. _Roc Toolkit Receiver to PulseAudio relay_ is now running, it can be stopped, and restarted now through controlling the new container through Docker.

# Testing PulseAudio relaying

TODO clearer instructions

1. From your terminal test the following command, when you get static on your speakers, all is fine, when you get silence, it means that your PulseAudio is either not working or your audio configuration is not working speakers
```sh
pacat < /dev/urandom
```
Hint: run the following command to figure out where your PulseAudio service is running:
```sh
echo $PULSE_SERVER
```

2. From the _Exec_ screen on the the _roc-pulse-relay-amd64_ container in Docker Desktop, the following command should result in static on your speakers, when you get silence the network connection between docker and your PulseAudio server is not working
```sh
pacat < /dev/urandom
```

3. From your terminal copy a .wav-file (e.g. https://www2.cs.uic.edu/~i101/SoundFiles/CantinaBand60.wav) to the _roc-pulse-relay-amd64_ container, replace the _???_ with its container ID. (You can copy it from the Docker Desktop window, it is right below the container name, e.g. 6982f5560704455c149e8ce16f206ec639c7498ed1abfffb7cea3065da1556e9)

```sh
docker cp CantinaBand60.wav ???:/CantinaBand60.wav
```

4. From the _Exec_ screen on the the _roc-pulse-relay-amd64_ container in Docker Desktop, the following command should result in the Cantina Band's song playing on your speakers, when you get silence the network connection between docker and your PulseAudio server is not working

```sh
roc-send -vv -s rtp+rs8m://127.0.0.1:10001 -r rs8m://127.0.0.1:10002 -i file:CantinaBand60.wav
```
# Installation of the Roc Virtual Audio Device for macOS

TODO

# Running PulseAudio as a Windows service and use it in WSL 2

The built-in _WSLg PulseAudio_ might be unstable or giving glitchy sound, in that case it is recommended to install a proper PulseAudio server on Windows and repoint WSL 2 to that service rather than the built-in WSLg one.

1. Install PulseAudio for Windows: downloaded from here https://pgaskin.net/pulseaudio-win32 and follow this guide to get it installed as a service: https://www.linuxuprising.com/2021/03/how-to-get-sound-pulseaudio-to-work-on.html

2. modify the _.bashrc_ file that will set the _$PULSE_SERVER_ environment variable when ever a WSL 2 session is started, to point to the Windows Service hosting PulseAudio. Using e.g. `nano ~/.bashrc` command (_CTRL+O_ to save):
	1. Comment out this line - in case it exists - to disable WSLg pulse server: `export PULSE_SERVER=unix:/mnt/wslg/runtime-dir/pulse/native`
	2. Add these lines, this will set the  _$PULSE_SERVER_ variable and ensure that any app needing to connect to PulseAudio for sound will connect to the Windows Service instead:
```sh
export HOST_IP="$(ip route |awk '/^default/{print $3}')"
export PULSE_SERVER="tcp:$HOST_IP"
```

3. Test
```sh
pacat < /dev/urandom
```

# Links
* _Roc Toolkit_ Real-time audio streaming over the network. https://github.com/roc-streaming/roc-toolkit/
* _PulseAudio modules for Roc Toolkit_ https://github.com/roc-streaming/roc-pulse/
* _Roc Virtual Audio Device for macOS_ https://github.com/roc-streaming/roc-vad/