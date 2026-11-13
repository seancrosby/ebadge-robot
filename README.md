# ebadge-robot
Robot built around BYUI eBadge Board

## Camera

Hardware: Seeed Studio XIAO ESP32-S3 Sense

URL: https://www.seeedstudio.com/XIAO-ESP32S3-Sense-p-5639.html?srsltid=AYCX9vLVfrRYBy0KLQ04KPs5JPmnX7SDeQ4ej73z_hxWiYsDhBlCGIi2

Purchase: https://www.amazon.com/dp/B0C69FFVHH?ref=ppx_yo2ov_dt_b_fed_asin_title

Wiki: https://wiki.seeedstudio.com/xiao_esp32s3_pin_multiplexing/

### USB webcam firmware

The camera firmware lives in `camera/usb_webcam` and turns the XIAO ESP32-S3 Sense into a USB Video Class (UVC) camera. Video is sent over USB; the COM port is only for flashing and recovery. The tested setup uses the OV3660 camera module.

The firmware is based on [JacobFV/esp32-usb-webcam](https://github.com/JacobFV/esp32-usb-webcam) at commit 6d37c95d9248e07f4135a2a163238c2e5bf12870, with local XIAO OV3660, Windows, and input-validation updates. The source is Apache-2.0 licensed.

#### Build and first flash on Windows

1. Install the PlatformIO IDE extension in VS Code and open `camera/usb_webcam` as the project.
2. Connect the XIAO with a USB data cable. In the tested setup, its bootloader/programming port was `COM14`.
3. From the PlatformIO terminal in `camera/usb_webcam`, build and upload:

   ```powershell
   pio run
   pio run -t upload --upload-port COM14
   ```

   The project pins Espressif32 platform 6.9.0 and ESP-IDF 5.3.1. The first build downloads the compiler, framework, and ESP-IDF components and can take several minutes.

4. After reboot, Windows Device Manager should list `UVC CAM1` under Cameras. Open the Windows Camera app and select `UVC CAM1` to view the stream. The current recovery serial interface enumerates as `COM17`; Windows can assign a different number.

#### Re-enter the bootloader for a later upload

Open the current CDC recovery port, send `bootloader`, and press Enter:

```powershell
pio device monitor -p COM17 -b 115200
```

The board restarts and reappears as a ROM bootloader serial port. Check Device Manager for its current COM number, then upload with that number:

```powershell
pio run -t upload --upload-port COM14
```

Replace `COM17` and `COM14` above with the current port numbers shown by Windows. The firmware code and build notes are in `camera/usb_webcam/README.md`.

### Stream quality

The current stream tops out at 640x480 at 15 fps with JPEG quality 12. The OV3660 sensor itself can capture higher resolutions, but the current UVC modes are kept conservative for stable USB streaming. The OV3660 is fixed-focus and has no autofocus; JPEG quality changes can improve compression detail but cannot fix optical focus.
