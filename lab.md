# Lab Setup

The lightning network user documentation was created using the following software and configurations.

-  ### [Oracle VirtualBox](https://www.virtualbox.org/ "Oracle VirtualBox")
VirtualBox is a powerful x86 and AMD64/Intel64 virtualization product for enterprise as well as home use.

 Interacting with our bitcoin node is easily done from a Linux machine, and it is my preference to do so.
 
 You can easily `SSH` into your node on Windows as well using PuTTY. 

-  ### [Xubuntu](https://xubuntu.org/ "Xubuntu")
Xubuntu provides a configurable, user friendly, minimal desktop GUI, and stability.

- ### [Terminator](https://terminator-gtk3.readthedocs.io/en/latest/ "Terminator")
"At its simplest Terminator is a terminal emulator like xterm, gnome-terminal, konsole, etc."


## Inital Setup

### Update System
```
sudo apt update ; sudo apt upgrade -y
```
### Terminator
```
sudo apt update ; sudo apt install terminator -y
```

Terminator allows us to multiplex our terminal session, so we can easily control multiple sessions.
[![Terminator](https://i.imgur.com/Ml4Jimp.png "Terminator")](https://i.imgur.com/Ml4Jimp.png "Terminator")

Split Terminal Window Vertically | `CTRL` + `SHIFT` + `E`
[![Terminator Multiplex](https://i.imgur.com/A4rA6hN.png "Terminator Multiplex")](https://i.imgur.com/A4rA6hN.png "Terminator Multiplex")

Split Terminal Window Horizontally | `CTRL` + `SHIFT` + `O`
[![Terminator Multiplex](https://i.imgur.com/csF9M5z.png "Terminator Multiplex")](https://i.imgur.com/csF9M5z.png "Terminator Multiplex")

[![Terminator Multiplex](https://i.imgur.com/ubV47oD.png "Terminator Multiplex")](https://i.imgur.com/ubV47oD.png "Terminator Multiplex")
