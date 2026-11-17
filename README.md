# OpenAir Multitrack + Raspberry Pi: Quick Setup

## 1. Prerequisites

- Raspberry Pi with Raspberry Pi OS installed or equivalent, a suitable power supply, Internet access and SSH access.
- OpenAir Multitrack with its power supply and antennas, receiving ADS-B traffic and a valid GNSS fix.
- OpenAir firmware with the Beast timestamp correction required for MLAT with `radarcape_gps`. Confirm availability for your receiver with AVIONIX.
- Both devices connected to the same router or LAN, preferably by Ethernet.
- Accurate antenna latitude, longitude and altitude, plus a unique station name.

Run the commands below on the Pi. Example addresses and coordinates must be replaced with your own.

## 2. Find and fix the OpenAir IP address

Find the receiver's local IP address on your router's **DHCP lease page**, then open that IP in a browser.

If you cannot find its local IP, or need access from another network, open `http://multitrack.openair1090.com/<ID>`. Replace `<ID>` with the **last part of the serial number** on the receiver's label: for example, S/N `1090_868_929` gives ID `929`. Enter the receiver's **MAC address**, printed on the label, as the password.

In **Network config** or **Config → LAN**, switch from DHCP to **Static**. Enter an available IP address on your LAN, outside the router's DHCP pool or excluded from it, along with the correct subnet mask, gateway and DNS settings. Save the changes and allow the receiver to restart if required.

**Write down the fixed IP address** and confirm you can open it from the Pi's network. Use this address in all following steps. The examples below use `192.168.1.60`.

## 3. Install and configure readsb

Follow the [readsb installation instructions](https://github.com/wiedehopf/adsb-scripts/wiki/Automatic-installation-for-readsb#installation).

Then open its configuration:

```bash
sudo nano /etc/default/readsb
```

Set the device type to `none`:

```bash
RECEIVER_OPTIONS="--device-type none"
```

In the existing `NET_OPTIONS` line, keep the local port settings and add this connector once, inside the quotes:

```text
--net-connector 192.168.1.60,30005,beast_in
```

Save with `Ctrl+O`, Enter, then `Ctrl+X`, and restart:

```bash
sudo systemctl restart readsb
```

The local map is available at `http://PI_IP/tar1090/`, replacing `PI_IP` with the Pi's address. `radarcape_gps` is not a readsb setting.

## 4. Install and configure adsb.fi

Follow the [adsb.fi installation instructions](https://github.com/adsbfi/adsb-fi-scripts#2-install-the-feed-client).

Then open:

```bash
sudo nano /etc/default/adsbfi
```

Replace the existing source settings with:

```bash
INPUT="192.168.1.60:40001"
INPUT_TYPE="radarcape_gps"
```

Use the **OpenAir's IP** and keep the other station settings. Save, then restart and check:

```bash
sudo systemctl restart adsbfi-feed adsbfi-mlat
systemctl is-active readsb adsbfi-feed adsbfi-mlat
sudo journalctl -u adsbfi-mlat -n 30 --no-pager
```

All three services should be active. Confirm the local map updates and the MLAT client receives messages and connects to its server. Allow several minutes for synchronization; MLAT positions also depend on traffic and nearby stations.
