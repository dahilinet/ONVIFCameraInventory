# ONVIF Camera Inventory V4

A Windows desktop utility for discovering ONVIF IP cameras, viewing live RTSP
video and audio, and configuring camera network settings.

The application automatically starts in **Turkish** when the current user's
Windows display language is Turkish, and in **English** for all other languages.

> **Important:** Network configuration uses authenticated ONVIF requests.
> MAC-based IP recovery and vendor-specific Layer-2 configuration are not
> implemented. A camera can appear in discovery without being reachable for
> configuration.

## Features

| Feature | Description |
| --- | --- |
| Camera discovery | WS-Discovery multicast through active local IPv4 adapters. |
| Device inventory | IP address, ONVIF response time, cached MAC address, advertised model/name, endpoint UUID, and service address. |
| Sortable columns | Click any column heading to sort; click again to reverse. IP addresses and response times sort numerically. |
| Response-time measurement | Median of three read-only ONVIF requests, refreshed on each scan. |
| Live preview | RTSP video with selectable camera profiles and TCP or UDP transport. |
| High-resolution playback | Aspect-ratio-preserving bicubic downscaling through PyAV/FFmpeg. |
| Audio controls | Mute, volume from 0 to 100%, and local software gain from 0 to +20 dB. |
| Network configuration | DHCP or static IPv4, subnet mask, default gateway, and DNS. |
| CSV export | Export the inventory in its current displayed order. |
| Automatic language selection | Turkish or English, based on the Windows display language. |
| Background discovery | Network helper processes run without additional console windows. |
| Temporary-file cleanup | Normal extraction cleanup and ownership-verified recovery of registered files left by interrupted runs. |

No separate VLC installation or proprietary camera player is required.

## Requirements

### Running the executable

- 64-bit Windows.
- Access to the camera network.
- ONVIF discovery enabled on the cameras.
- Firewall rules that permit the required discovery and camera traffic:
  - WS-Discovery multicast: UDP port `3702`.
  - ONVIF HTTP/HTTPS: the camera's advertised service port.
  - RTSP: the camera's advertised streaming port, commonly `554`.
  - Additional media traffic when using UDP playback.

The standalone executable includes its Python runtime and playback dependencies.
You do not need to install Python or a separate FFmpeg executable to run it.

### Developing or building

Use 64-bit Python 3.12 on Windows, with Tkinter available.
This is the Python version used for development and verification.

## Quick Start

1. Run `ONVIFCameraInventoryV4.exe`.
2. Click **Discover ONVIF devices**.
3. Click a column heading to sort the results.
4. Click a camera row, or right-click it and select **View camera**.
5. The camera window attempts to connect using `admin` / `123456`, selects the
   lowest advertised video resolution, and starts preview.
6. If the connection fails, enter the correct credentials in the login dialog.
7. Use **Export CSV** to save the inventory.

The built-in credentials are only an initial connection attempt, not a password
search. Use appropriate, unique camera credentials on production networks.

**Passwords are visible in the interface.** Changed credentials are not saved to
files or included in CSV exports.

### Camera window behavior

- Selecting another camera reuses the same popup and stops the previous player.
- Closing the camera popup with **X** stops its player and leaves the inventory
  open. Selecting a camera opens the popup again.
- Closing the main inventory with **X** closes the popup and exits the application.
- **Read camera** reads the selected camera's information and available profiles.
- Choose a profile in **Stream**, then use **Live preview** or **Stop preview**.
- Stop playback before changing the profile or TCP/UDP transport.

## Understanding ONVIF Response Time

The **ONVIF time (ms)** column appears immediately after the IP address.
It shows the median elapsed time of three anonymous, read-only
`GetSystemDateAndTime` requests.

**This is not physical distance, cable length, or a switch/router hop count.**
The measurement includes request and connection overhead and camera processing.
Camera load, PC load, and network congestion can affect the result. A lower
value means a faster application response, not necessarily a closer camera.

Measurements run in the background, with at most eight cameras queried in
parallel and 1.5-second connect/read timeouts. They do not attempt authentication,
follow redirects, or use environment-configured HTTP proxies.

- Unreachable cameras and cameras that reject anonymous queries remain listed
  as **Not measured**, with a reason in the scan warnings.
- Cameras sharing an IP address are not queried for response time.
- Unmeasured entries stay at the bottom in either sort direction.
- Each discovery scan obtains new measurements.
- Response time is currently a GUI-only field and is not included in CSV.

## Changing Camera Network Settings

1. Open a camera and authenticate successfully.
2. Select its network interface.
3. Choose the desired configuration:
   - **DHCP On:** request an automatic IPv4 address.
   - **DHCP Off:** enter a static IP address and subnet mask.
4. Enter a default gateway and DNS server if needed.
5. Click **Save network settings**.
6. Review the selected camera, interface, MAC address, and proposed settings
   before confirming.
7. Rediscover the camera and verify connectivity at its new address.

Complete IPv4 addresses can be pasted into the four-octet fields.
**Revert edits** restores the last values read from the camera, not factory
defaults.

### Network configuration safeguards

- Changes apply to the selected camera, not to the PC's network adapters.
- Duplicate discovered IPs block camera control to avoid targeting the wrong
  device. An IP already listed in the inventory cannot be assigned to another
  camera.
- These checks cannot detect every conflict: offline or undiscovered devices
  may still own an address.
- Blank gateway/DNS fields mean **leave unchanged**.
- `0.0.0.0` explicitly clears the corresponding IPv4 gateway/DNS entries.
- DHCP changes automatic IPv4 addressing only; it does not change gateway/DNS.
- Gateway and DNS settings are device-wide. Additional addresses, IPv6 values,
  and DNS search domains are preserved where applicable.
- Network writes are **not atomic**. Partial or unacknowledged changes are
  reported explicitly, without automatic retry or rollback.
- Preview is stopped before a network change. The camera may become unreachable
  at its old address.
- Acknowledgment does not prove connectivity at the new address.
- No reboot is sent automatically. If required, reboot the camera manually
  when safe.
- Factory defaults and vendor-specific default-parameter locks are not changed.

## Discovery Limitations

### Different subnets and reset cameras

A camera that has reverted to a factory IP may still appear if its WS-Discovery
response reaches the PC, even when its address is outside the PC's subnet.
Listing does not require a successful ping, ARP entry, authenticated login,
response-time measurement, or preview.

However, viewing and configuring that camera still require usable IP
connectivity. You may need to add a temporary secondary IPv4 address to the
appropriate Windows adapter manually. The application does not do this for you.

### Duplicate IP addresses

Distinct discovered ONVIF endpoints on the same IP remain separate inventory
entries. Their cached MAC is marked ambiguous, and IP-based control is blocked.

The application cannot guarantee discovery of every conflicting device,
especially when devices share the same endpoint identity or cannot return
traffic. Recovering those devices individually by MAC requires a compatible
vendor-specific Layer-2 tool.

### Other limitations

- WS-Discovery is generally limited to the local network segment; it is not
  normally routed across VLANs or routers.
- A camera must send a discovery response that reaches the PC to be listed.
- This is not a general-purpose RTSP port or IP-range scanner.
- MAC addresses in the main inventory come from the Windows ARP cache and may
  be unavailable. Authenticated camera interface information can provide a MAC
  in the camera window.
- Model/name information depends on the camera's advertised discovery scopes.
- Live preview supports ONVIF Media1 and codecs supported by the bundled FFmpeg
  build. Media2-only cameras are not supported.
- A camera without an audio track cannot provide sound.
- Audio gain affects local playback only, not the camera microphone settings.
  High gain can amplify noise and cause clipping.

## CSV Export

CSV files use UTF-8 with a byte-order mark and contain these columns:

```text
IP, MAC, Model/name, Endpoint UUID, XAddrs, Scopes
```

Rows follow the current inventory sort order. Column names remain in English
regardless of the interface language.

## Security and Responsible Use

Use this application only on cameras and networks you own or are authorized
to administer.

- Discovery and response-time measurements do not change camera settings.
- Camera configuration requires credentials and explicit confirmation.
- No brute-force password guessing or bulk network configuration is performed.
- Authenticated ONVIF requests use HTTP Digest and WS-Security UsernameToken
  digest. Camera and PC clocks may need to be synchronized.
- Requests are restricted to services on the selected camera IP. Redirects and
  environment-configured HTTP proxies are disabled.
- HTTP-based ONVIF and ordinary RTSP transport do not provide transport
  encryption. Use a trusted, appropriately isolated camera network.

## Run from Source

Open PowerShell in the repository root:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r .\src\requirements.txt
.\.venv\Scripts\python.exe .\src\onvif_inventory.py
```

The application uses Requests, PyAV/FFmpeg, NumPy, SoundDevice/PortAudio, and
Tkinter. See [requirements.txt](src/requirements.txt) for dependency constraints.

## Build a Standalone EXE

Using the virtual environment above:

```powershell
.\.venv\Scripts\python.exe -m pip install pyinstaller
.\.venv\Scripts\python.exe .\src\build.py
```

The build produces `ONVIFCameraInventoryV4.exe` in the repository root.
Intermediate files are written under `src\work`.

Close running instances of the application before rebuilding the executable.
The build includes the camera icon, playback dependencies, dependency/license
metadata, and the runtime cleanup hook.

## Tests

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s .\src -v
```

Automated tests use simulated camera responses and do not change physical camera
settings. Tests cover discovery, sorting, response-time measurement, network
request construction, partial failures, player lifecycle, cleanup ownership,
and automatic language selection.

Real-device checks have included discovery and read-only response-time
measurement on 40 cameras, and main/substream playback on PA4 and MC200E3 cameras.
Network configuration writes have not been verified on physical cameras.
Compatibility with every ONVIF camera is not guaranteed.

## Project Layout

```text
README.md                    Project overview and usage
ONVIFCameraInventoryV4.exe    Standalone Windows application
src\                         Source code, tests, icon, and build helper
    README.md                Detailed implementation and operational notes
    requirements.txt         Runtime dependency constraints
    build.py                 PyInstaller build entry point
    work\                    Generated build intermediates
```

For implementation details, temporary-file cleanup behavior, and additional
operational notes, see [the technical README](src/README.md).

## License

The original project source is intended for publication under the **MIT License**.
Include the MIT license text and the appropriate copyright notice in a `LICENSE`
file when publishing the repository.

Third-party components retain their own licenses. Publishing the original source
under MIT does not relicense bundled libraries or the FFmpeg build. Preserve
required notices and review the redistribution requirements of the exact
dependencies included in a binary release.

## Developer

**Oğuz Ersöz**  
[info@ulusalguvenlik.com](mailto:info@ulusalguvenlik.com)
