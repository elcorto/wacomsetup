About
-----

* Map Wacom tablet to primary screen (`xrandr | grep primary, xsetwacom ...
  MapToOutput ...`)
* Scale and crop Wacom active area to screen aspect ratio.
* For now only stylus area aspect ratio (e.g. 16/10 = 1.6) `<=` screen aspect
  ratio (e.g. 16/9 = 1.777...) supported.
* **We hard-coded some custom settting for our Wacom model (end of the script).
  Deactivate if needed. See also [#1](https://github.com/elcorto/wacomsetup/issues/1)**

Requirements
------------

* xsetwacom (Debian: xserver-xorg-input-wacom)
* Python3


X11 frame geo
-------------

We assume `W1xH1+W0+H0` (example `3840x2160+0+0`) with `+w0+h0` = `+0+0`, where
`+` means upper left `(w0, h0)`.

```
(w0, h0)
(0,  0)     w
     +--------------+
     |              |
     |              | h
     |              |
     +--------------+
                    (w1, h1)
                    (3840, 2160)
```

There is also the syntax `-w0-h0` which means `w1` and `h1` are relative to lower
right `(w0,h0)`.

We usually have this setup (from `xrandr`):

```
DP1-1 connected primary 3840x2160+0+0 (normal left inverted right x axis y axis) 600mm x 340mm
eDP1 connected 1920x1080+3840+0 (normal left inverted right x axis y axis) 310mm x 170mm
```

or

```
DP-1-1 connected primary 3840x2160+0+0 (normal left inverted right x axis y axis) 600mm x 340mm
eDP-1 connected (normal left inverted right x axis y axis)
```

which is a 4K screen (`DP1-1`) left of a 1080p laptop (`eDP1`):

```
DP1-1 eDP1
```

i.e.

```
xrandr --output eDP1 --mode 1920x1080 --pos 3840x0 --rotate normal
       --output DP1-1 --primary --mode 3840x2160 --pos 0x0 --rotate normal
```

which is set by `arandr`. Not tested if `--right-of` creates the same geo
`1920x1080+3840+0` as `--pos 3840x0` does.

We ignore `w0,h0` != `0,0` which is the case in multi monitor setups as just shown
(`1920x1080+3840+0` instead of `1920x1080+0+0`). This is irrelevant for aspect
correction, since we use `MapToOutput` to map the stylus area to a screen instead
of using absolute coordinates. Should work in any kind of monitor setup
(untested).


Scaling mode
------------

Currently we only support stylus area aspect ratio (e.g. 16/10 = 1.6) `<=`
screen aspect ratio (e.g. 16/9 = 1.777...). In this case we scale the stylus
active area by removing some pixels at the bottom of the active area where we
don't draw often anyway:

```
+--------------+       +--------------+
| 16/10 = 1.6  |  -->  | 16/9 = 1.777 |
| stylus orig  |       | stylus new   |
| area         |       +--------------+
+--------------+
```

In the other case, we'd need to keep the height but remove a strip of pixels
from the left and/or right of the active area. This can be easily added but we
don't have that use case. For now we throw an exception in that case. PRs
welcome!

```
+--------------+
| 16/10 = 1.6  |
|              |
|              |
+--------------+

      |
      |
      v

 +----------+
 | 4/3      |
 | = 1.333  |
 |          |
 +----------+
```

Wacom frame geo
---------------

```
w0 h0 w1 h1
```

example:

```sh
$ xsetwacom get "Wacom Intuos Pro S Pen stylus" Area
0 0 31920 19950
```

Wacom Button mapping (Intuos Pro S, 2020 model)
-----------------------------------------------

See

* https://askubuntu.com/a/660063
* https://wiki.archlinux.org/index.php/Wacom_tablet#Remapping_buttons

From

```
/usr/share/libwacom/intuos-pro-s.tablet
/usr/share/libwacom/layouts/intuos-pro-s.svg
```

we find

```
2 B
3 C
4 D
1 A (buttom middle wheel)
5 E
6 F
7 G
```

but that's wrong. On our device `B=1,C=2,...`. Bummer.
