# Installing a USB RCP Thread Dongle in Home Assistant

## Overview

This guide walks you through adding a USB Radio Co-Processor (RCP) Thread dongle to your Home Assistant instance.
---

## Prerequisites

Before you begin, ensure you have:

- A current Home Assistant OS or Home Assistant Supervised instance Supervised is recommended.
- A compatible USB RCP Thread dongle (e.g., **Our ZB-GW04-OT1**, **Home Assistant Connect ZBT-1**, **SMLIGHT SLZB-07**, or **Silicon Labs MGM210P**)

---

## Step 1: Plug In the USB Dongle

1. If using a Raspberry Pi or similar SBC, prefer a **USB 2.0 port** to avoid potential interference from USB 3.0 with 2.4 GHz radios.

> **Configuration Tip:** Use a short USB extension cable to move the dongle away from the host board. This can significantly improve radio range and reduce interference.
> **Hardware Tip:** For performance and reliability a Mini PC fits well over a Pi and is recommended.

---

## Step 2: Verify the Dongle is Detected

1. In Home Assistant, go to **Settings → System → Hardware**.
2. Click the **"All Hardware"** button.
3. Look for your dongle in the list — it typically appears as a USB serial device such as:
   - `/dev/ttyUSB0`
   - `/dev/ttyACM0`
   - `/dev/serial/by-id/usb-...`

If the dongle does not appear, try a different USB port or reboot the host.

---

## Step 3: Install the OpenThread Border Router Add-on

Home Assistant uses the **OpenThread Border Router (OTBR)** add-on to manage Thread networking.

1. Go to **Settings → Add-ons → Add-on Store**.
2. Search for **"OpenThread Border Router"**.
3. Click on the add-on and select **Install**.
4. Wait for the installation to complete.

---

## Step 4: Configure the Add-on

1. After installation, go to the **OpenThread Border Router** add-on page.
2. Click the **Configuration** tab.
3. Set the `device` field to the path of your USB dongle, for example:

   ```yaml
   device: /dev/ttyACM0
   ```

   Or use the persistent path for reliability:

   ```yaml
   device: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
   ```

4. Set the `baudrate` to `230400` for the ZB-GW04-OT1 other types are commonly `460800` or `115200`. Check your dongle's documentation.
5. Leave other settings at their defaults unless you have a specific reason to change them.
6. Click **Save**.

---

## Step 5: Start the Add-on

1. Go to the **Info** tab of the OpenThread Border Router add-on.
2. Toggle **"Start on boot"** to enabled.
3. Click **Start**.
4. Check the **Log** tab to confirm the add-on has started successfully. You should see output similar to:

   ```
   [info] Starting OpenThread Border Router
   [info] Thread interface: wpan0
   [info] Border Router started successfully
   ```

---

## Step 6: Enable Thread in Home Assistant

1. Go to **Settings → Thread → ha-thread-xxxx**.
2. Click on the information icon symbol (Circled i).
3. Note: Capture the **TLV** text — you will need it when pairing ESPHome Thread devices.

---

## Useful Resources

- [Home Assistant Thread Documentation](https://www.home-assistant.io/integrations/thread/)
- [OpenThread Border Router Add-on](https://github.com/home-assistant/addons/tree/master/openthread_border_router)
- [Home Assistant Connect ZBT-1 Setup Guide](https://www.home-assistant.io/connectzbt1/)
- [Matter Integration](https://www.home-assistant.io/integrations/matter/)
- [Gelidus Research Hardware](https://www.gelidus.ca/)
