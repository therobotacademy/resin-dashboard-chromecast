# WPE Webkit + Chromecast Monitoring for the RaspberryPi

This project, a fork of [resin-wpe](https://github.com/resin-io-playground/resin-wpe) provides an easy way of running [WPE
Webkit](https://www.igalia.com/wpe/) on a screen that can be cast to, while using the WPE browser as a splash screen. WPEWebkit is a full
featured browser that takes advantage of the GPU to provide hardware
accelerated CSS, WebGL, and HTML5 video.

## Showcase

[![IYoutube Example](https://img.youtube.com/vi/phj2hwZIioo/0.jpg)](https://www.youtube.com/watch?v=phj2hwZIioo)

[Full blog post about the project.](https://github.com/resin-io-playground/resin-wpe)

* Rendering WebGL [abstract shapes](https://mrdoob.neocities.org/023/)
	* <img src="http://i.imgur.com/RiP5gwe.gif" />
* Playing [html5 flappy bird](http://hyspace.io/flappy/)
	* <img src="http://i.imgur.com/X0folHz.gif" width="300px" />
* Playing HD video on [YoutubeTV](https://youtube.com/tv)
	* <img src="http://i.imgur.com/wpRygCt.gif" />

## Table of contents

* [Table of contents](#table-of-contents)
* [Getting started](#getting-started)
* [Configuration](#configuration)
    * [Displaying on an HDMI screen](#displaying-to-an-hdmi-screen)
    * [Displaying on a PiTFT 2.8"](#displaying-to-pitft-28)
		* [Capacitive touchscreen](#capacitive-touchscreen)
		* [Resistive touchscreen](#resistive-touchscreen)
* [Controlling content](#controlling-content)
	* [Loading a URL](#loading-a-url)
	* [Offline content](#offline-content)
	* [Enabling mouse](#enabling-mouse)
* [Chromecast Support](#chromecast-support)

## Getting started

* Sign up on [resin.io](https://dashboard.resin.io/signup)
* Go through the [getting started guide](http://docs.resin.io/raspberrypi/nodejs/getting-started/) and create a new application
* Clone this repository to your local workspace
* Add the _resin remote_ to your local workspace using the useful shortcut in the dashboard UI ![remoteadd](http://i.imgur.com/no1DNSx.png)
* `git push resin master`
* See the magic happening, your device is getting updated Over-The-Air!

## Configuration

A lot of the configuration of this project is about setting up `config.txt`.
The way you do this on resin.io is by setting some special fleet configuration
variables. If you don't know how to do this, you can [find more info
here](https://docs.resin.io/configuration/advanced/#modifying-config-txt-remotely-).

First, we need to set the GPU memory to something suitable for hardware
accelerated graphics. You can set only one of the following if you know how
much RAM your Pi has or you can set all of them and your Pi will autoselect
based on its memory. **If you don't set any of these, WPE won't work.**

| Key                                 | Value
|-------------------------------------|----------
|**`RESIN_HOST_CONFIG_gpu_mem_256`**  | **`128`**
|**`RESIN_HOST_CONFIG_gpu_mem_512`**  | **`196`**
|**`RESIN_HOST_CONFIG_gpu_mem_1024`** | **`396`**

### Displaying on an HDMI screen

WPE will display to the primary framebuffer (HDMI output) by default. If you
don't set the resolution, it will be autodetected on boot up. Depending on what
you want to do, you might find the performance at FullHD resolution isn't
enough. In that case you can force the HDMI to 720p by adding the following
fleet configuration variables:

| Key                               | Value
|-----------------------------------|--------
|**`RESIN_HOST_CONFIG_hdmi_group`** | **`1`**
|**`RESIN_HOST_CONFIG_hdmi_mode`**  | **`4`**

### Displaying on a PiTFT 2.8"

<img src="https://i.imgur.com/VaIPJ0L.jpg" alt="PiTFT picture" width="350px" />

Displaying to a secondary framebuffer is accomplished by rendering on the
primary and continuously copying the resulting image to the secondary. The
reason for this is that the Pi can only render hardware accelerated graphics to
the primary framebuffer. A side effect of this is that you can't display
different content or have different resolutions between the HDMI and your PiTFT
screen.

Since the rendering will happen to the primary framebuffer, we need to force
the HDMI output to be 320x240, our native resolution. You can do this with the
following variables. If don't set those, the Pi will render on a higher
resolution and then scale down the image to fit the Pi screen.

| Key                                       | Value
|-------------------------------------------|-------------------------
|**`RESIN_HOST_CONFIG_hdmi_force_hotplug`** | **`1`**
|**`RESIN_HOST_CONFIG_hdmi_group`**         | **`2`**
|**`RESIN_HOST_CONFIG_hdmi_mode`**          | **`87`**
|**`RESIN_HOST_CONFIG_hdmi_cvt`**           | **`320 240 60 1 0 0 0`**

#### Capacitive touchscreen

**Warning:** The PiTFT based on the `ft6x06_ts` kernel module is not currently
supported out of the box on resin.io. If your screen says "FT6206" at the back,
it's probably one of those.

| Key                              | Value
|----------------------------------|--------------------------------------------------------
|**`RESIN_HOST_CONFIG_dtoverlay`** | **`pitft28-capacitive,rotate=90,speed=62000000,fps=60`**

#### Resistive touchscreen

| Key                              | Value
|----------------------------------|--------------------------------------------------------
|**`RESIN_HOST_CONFIG_dtoverlay`** | **`pitft28-resistive,rotate=90,speed=62000000,fps=60`**

## Controlling content

### Loading a URL

To configure the URL displayed by webkit, set the **`WPE_URL`** environment
variable. The default value is [Youtube TV](http://www.youtube.com/tv)

### Offline content

If you want your device to display content even without internet, you can add
your content in the docker image and point WPE to them. Append a similar
Dockerfile fragment to your project:

```Dockerfile
COPY public_html /var/lib/public_html

ENV WPE_URL="file:///var/lib/public_html/index.html"
```

### Enabling mouse

To enable the mouse cursor you have to set the following environment variable.
Keep in mind that there are some known issues with mouse support when the
webpage includes iframes.

| Key                    | Value
|------------------------|---------
|**`WPE_BCMRPI_CURSOR`** | **`1`**

## Chromecast Support

This build of resin-wpe uses pychromecast to monitor the status of a Chromecast that is assumed to be plugged into the same TV.
When the chromecast is closed, a watchdog script notices, and sends a CEC 'as' (active source) command to the TV.
Functionally this allows you to set a custom Chromecast splash screen, something google has yet to allow.

To enable this functionaltiy, set the following environment variable:

| Key                    | Value
|------------------------|---------
|**`CC_NAME`** | **`{Name of the Chromecast}`**

## Known Issues

* Sound is only output through HDMI
