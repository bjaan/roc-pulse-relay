# Roc Toolkit Receiver to PulseAudio server relay: roc-pulse-relay
Service that streams real-time audio from a Roc Sender to a PulseAudio server.

![roc-pulse-relay overview](https://github.com/bjaan/roc-pulse-relay/blob/main/main.png?raw=true)

The Roc Toolkit's Sender will allow you to stream stable CD-quality audio over the network. This service will stream that audio to your PulseAudio server.

This litte Docker container will start a service that will allow you to connect any digital audio source from the Roc Toolkit, this can be as simple as its *roc-send*, that streams an audio file over the network, or a virtual audio device that support the Roc Toolkit real-time audio streaming protocol.

Or, use this service when you've set-up a Virtual Speakers as a Roc Virtual Audio Device in macOS. This can be a real Mac, where you need to play audio through another device, or this can be a macOS running in a VM-host (like in QEMU or VirtualBox) to connect to your speakers over the PulseAudio service that is running on your host device.  

The PulseAudio service can be run on a Raspberry Pi, another Linux, or a Windows PC as a Windows Service, for Roc-Steady Sound!

# Installation

These assume that you already have the following things configured on your (host) machine:
* PulseAudio server running and working audio on your host, test with the _pacat < /dev/urandom_ command if that is working, it should let you hear static over your speakers.
* Docker installed on the same machine with PulseAudio installed, either under WSL 2 on Windows or directly on Linux.

For the example use, mentioned above, to have working Virtual Speakers working on macOS, the _Roc Virtual Audio Device for macOS_ device set-up in macOS - ready to connect, see below for [instructions](#-Installation-of-the-Roc-Virtual-Audio-Device-for-macOS)
 
# Testing

```sh
pacat < /dev/urandom
docker cp CantinaBand60.wav ???:/CantinaBand60.wav
roc-send -vv -s rtp+rs8m://127.0.0.1:10001 -r rs8m://127.0.0.1:10002 -i file:CantinaBand60.wav
```
# Installation of the Roc Virtual Audio Device for macOS

TODO

# Links
*Roc Toolkit* Real-time audio streaming over the network. https://github.com/roc-streaming/roc-toolkit/
*PulseAudio modules for Roc Toolkit* https://github.com/roc-streaming/roc-pulse/
*Roc Virtual Audio Device for macOS* https://github.com/roc-streaming/roc-vad/