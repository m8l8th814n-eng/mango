# mangowm-ng

mango with HDR, blur, rounded corners, box shadows and the rest of the scenefx
effects running on the **Vulkan** renderer. No patches to apply: every change
lives in these three branches.

| what | repo | branch |
| --- | --- | --- |
| wlroots fork, split scene/output Vulkan render passes | `m8l8th814n-eng/wlroots-vkfx` | `vulkan-effects` |
| scenefx, Vulkan blur/corners/shadows + HDR color transforms | `m8l8th814n-eng/scenefx` | `vulkan-hdr` |
| the compositor | `m8l8th814n-eng/mango` | `mangowm-ng` |

Build in that order — mango links both by pkg-config (`wlroots-vkfx-0.20`,
`scenefx-0.5`).

## Arch

```sh
git clone -b mangowm-ng https://github.com/m8l8th814n-eng/mango.git
cd mango/packaging
(cd wlroots-vkfx && makepkg -si)
(cd scenefx      && makepkg -si)
(cd mango        && makepkg -si)
```

`wlroots-vkfx` installs alongside a stock `wlroots0.20`, it does not replace it.

## Any other distro

```sh
PREFIX=/usr/local

git clone -b vulkan-effects https://github.com/m8l8th814n-eng/wlroots-vkfx.git
meson setup wlroots-vkfx/build wlroots-vkfx --prefix=$PREFIX -Dexamples=false
meson install -C wlroots-vkfx/build

export PKG_CONFIG_PATH=$PREFIX/lib/pkgconfig:$PKG_CONFIG_PATH

git clone -b vulkan-hdr https://github.com/m8l8th814n-eng/scenefx.git
meson setup scenefx/build scenefx --prefix=$PREFIX
meson install -C scenefx/build

git clone -b mangowm-ng https://github.com/m8l8th814n-eng/mango.git
meson setup mango/build mango --prefix=$PREFIX
meson install -C mango/build
```

Build deps: meson, ninja, glslang, vulkan-headers, wayland-protocols, libdrm,
pixman, libxkbcommon, libinput, libdisplay-info, libliftoff, seatd, lcms2,
cjson, pango, xorg-xwayland.

## Running it

The Vulkan renderer is not the wlroots default, so ask for it:

```sh
WLR_RENDERER=vulkan mango
```

HDR is per monitor, in `~/.config/mango/config.conf`:

```
monitorrule=eDP-1,...,hdr:2
hdr_sdr_nits=400
```

`hdr:1` uses static mastering values, `hdr:2` reads them from the panel's EDID.
`hdr_sdr_nits` is the SDR white level inside an HDR signal (203 is the spec
default, higher makes SDR content brighter). `monitorrule=...,icc:/path.icc`
applies an ICC profile as the output color transform instead.
