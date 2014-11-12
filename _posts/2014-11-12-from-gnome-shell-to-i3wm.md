---
layout: post
title: From Gnome Shell to i3wm
---
My work laptop is a little bit underpowered and Gnome Shell has been bothering me to no end, so I decided today to switch over to the tiling [i3wm](http://i3wm.org) window manager. I have tried a few different ones in the past, but never stuck with them for some reason. I was feeling adventurous and fed up, so today felt like a good day to make the switch.

Following are the things I had to change to make my life work with i3wm. I will update this list as time goes on and I find more things.

- Install `pcmanfm` for file browsing. Sometimes it's nice to have a visual understanding of what's in my folders.
- Run `ibus-setup` and disable the "Next input method" keybinding. `Ctrl-space` is used in Emacs to set the mark, so this was really cramping my editing.
- Add the following lines to `~/.i3/config` for my externa keyboard's volume control:

  ```
  bindsym XF86AudioRaiseVolume exec /usr/bin/pactl set-sink-volume @DEFAULT_SINK@ -- '+10%'
  bindsym XF86AudioLowerVolume exec /usr/bin/pactl set-sink-volume @DEFAULT_SINK@ -- '-10%'
  bindsym XF86AudioMute exec /usr/bin/pactl set-sink-mute 0 toggle
  ```

  These lines use `pactl`, PulseAudio's command line interface, to raise, lower, and mute the system's audio.


Resources:

- http://i3wm.org/docs/refcard.html
- http://ninjaducks.in/hacking/gnome-3-to-i-3/
