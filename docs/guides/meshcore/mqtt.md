---
title: MeshCore Observer
tags:
MQTT
Info
MeshCore
---
# Setting up a MeshCore Observer for ChiMesh
## What is a MeshCore Observer?
A MeshCore Observer is a MeshCore node (repeater, room server, or companion device) that listens to nearby mesh traffic and reports what it hears to an MQTT broker over the internet. ChiMesh uses observer data to power network analysis, coverage mapping, and reliability reporting across the Chicagoland area via [CoreScope](http://corescope.chimesh.org).
Observers can be configured to share only advertisement packets, enough to appear on the node map, without exposing the contents of other traffic. You can stop sharing data at any time.

!!! note
    Observer firmware is only available for **supported devices**. Check that your hardware is compatible before proceeding. Not all MeshCore devices support the packet logging firmware required for this setup.

## Choose Your Setup Method
There are two ways to run a ChiMesh Observer:

## Method 1: Native Observer Firmware
### Step 1: Download the Firmware
Use the LetsMesh onboarding page to find and download the correct firmware for your device variant:
[https://analyzer.letsmesh.net/observer/onboard](https://analyzer.letsmesh.net/observer/onboard)
!!! note
    Files labeled **-merged** are for **fresh installs only** and will completely erase your flash. If you are updating from an existing MeshCore version, download the non-merged variant.
### Step 2: Flash the Firmware
**Fresh install:** Use the [MeshCore Web Flasher](https://flasher.meshcore.dev/) and select **Custom** to upload your downloaded file.
<br>
**Updating via OTA:** If you already have MeshCore running, you can update wirelessly using the companion app:
1. **Connect to the Device**
    - Open the MeshCore companion app
    - Select your device to be updated
    - Log in with the **admin** password
2. **Open Remote Management**
    - Tap the **···** menu (top right)
    - Choose **Remote Management**
3. **Launch the Command Line**
    - Scroll to the bottom
    - Tap **Command Line**
    - Enter `start ota`
4. **Connect to the OTA Network**
    - On your phone or computer, join the Wi-Fi network: `MeshCore-OTA`
5. **Upload the Firmware**
    - Open a browser and go to `http://192.168.4.1/update`
      *(or use the IP address assigned by the DHCP server on your router)*
    - Select and upload your downloaded `.bin` file
    - Wait for the update to complete
6. **Verify the Update**
    - Reopen the companion app
    - Confirm the firmware version under Remote Management
### Step 3: Apply ChiMesh Observer Settings
Once your firmware is installed, open the **Command Line** under Remote Management and enter the following configuration:
```
set timezone America/Chicago
set repeat off                          # <-- dedicated observers only, skip if this node also repeats
set path.hash.mode 2
set advert.interval 240
set flood.advert.interval 168           # <-- dedicated observer; use 72 or 48 if also acting as a repeater
set mqtt.iata ORD
set mqtt1.preset analyzer-us
set mqtt2.preset analyzer-eu
set mqtt3.preset chimesh
set mqtt.owner your-primary-companion-device-pub.key    # <-- optional but encouraged
set mqtt.rx on
set mqtt.tx advert
set wifi.ssid your-wifi-network
set wifi.pwd your-wifi-password
set bridge.enabled on
reboot
```
!!! note
    Replace `your-primary-companion-device-pub.key` with your companion device's actual public key. Replace `your-wifi-network` and `your-wifi-password` with your real Wi-Fi credentials, **do not wrap them in quotes**.
!!! warning
    `set repeat off` is for **dedicated observers only**. If this node is also serving as a mesh repeater, omit that line and consider using a lower `flood.advert.interval` value (72 or 48).
---
## Method 2: Computer Bridge (USB + Raspberry Pi)
!!! warning
    The **Room Server** role is not recommended for most deployments. Only use it if your node is placed in a difficult indoor location inside a large building where repeater mode is not suitable. If you are unsure which role is right for your setup, ask in the [ChiMesh Discord](https://ChiMesh.org/discord) before proceeding.
### Requirements
- Any supported MeshCore repeater **or room server** node with **existing firmware**
- A Linux device (Raspberry Pi or similar) with internet access
- USB connection between the node and the Linux device
### Step 1: Get Compatible Firmware
Your node needs firmware that includes **packet logging** support. Download the appropriate version for your device from the [MeshCore Web Flasher](https://flasher.meshcore.dev/) using the **Custom** option, or grab a pre-built packet-logging build for your variant.
### Step 2: Connect and Run the Install Script
With your node connected to your Linux device via USB, run the following from your terminal:
```bash
curl -fsSL https://raw.githubusercontent.com/Cisien/meshcoretomqtt/main/install.sh | bash
```
When asked whether to enable the LetsMesh Packet Analyzer, enter `y`.
!!! note
    This script is designed for **Repeater and Room Server** nodes only. For more details on configuration options, see the [meshcoretomqtt README](https://github.com/Cisien/meshcoretomqtt).
!!! note
    During setup, you will be asked to enter an **IATA airport code** for your region. For the Chicago area, use `ORD`. Make sure to use the correct IATA code, it is not the same as an ICAO code. A quick search for your nearest major airport will confirm it.
### Step 3: ChiMesh MQTT
The install script will configure the standard MQTT uplink. After setup, your observer data will flow to ChiMesh and appear on [CoreScope](http://ChiMesh.org) within a few minutes of your first advertisement packet being heard.
!!! note
    It may take up to **5 minutes** after your observer first connects before it appears in the Observers list. Your node must have an advertisement heard before it will show up in the map or dropdown, but packet data will still be recorded in the meantime.
---
## Additional Notes
- These settings are also required for upcoming ChiMesh(Core) services as they are announced
- `set mqtt3.preset chimesh` is what connects your observer to the ChiMesh network specifically
- The `mqtt.owner` field is optional but highly encouraged, it links your observer to your primary companion device

Thank you for supporting [ChiMesh.org](http://ChiMesh.org) - we hope to see you on MeshCore MQTT soon!
