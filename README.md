# DeMoD Audio – Advanced PipeWire Configuration for NixOS

**Copyright © 2025–2026 DeMoD LLC**  
**License: BSD 3-Clause** (see [LICENSE](./LICENSE) or below)

A modular, high-performance PipeWire audio configuration for NixOS, optimized for:

- Low-latency gaming & streaming (OBS, Proton, etc.)
- Bluetooth high-quality codec support (LDAC HQ, aptX HD, etc.)
- Light-to-moderate pro-audio workflows (Ardour, JACK apps)
- Seamless integration with both **KDE Plasma 6** and **Hyprland**

## Features

- Balanced low-latency defaults (~21 ms quantum) suitable for most gaming/streaming use cases
- Optional ultra-low latency mode for professional audio production
- High-quality Bluetooth codec negotiation (SBC-XQ, LDAC HQ, aptX HD, mSBC, hardware volume)
- Disabled libcamera monitor (avoids unnecessary CPU usage when using v4l2loopback/OBS virtual cam)
- Adaptive desktop integration:
  - Native Plasma 6 volume widget (no redundant tray applet)
  - `pasystray` included for Hyprland / non-Plasma sessions
- High-quality resampling in PulseAudio compatibility layer
- Realtime scheduling via rtkit

## Usage

Add this module to your NixOS flake or configuration:

```nix
{
  imports = [
    # ... other modules
    ./modules/audio.nix   # or inputs.demod-audio.nix or wherever you placed it
  ];

  custom.audio = {
    enable = true;

    # Optional fine-tuning (defaults are usually good)
    lowLatency.enable      = true;           # balanced gaming/streaming latency
    # lowLatency.proAudio  = true;          # uncomment for ~1–4 ms latency (risk of xruns)
    bluetooth.highQualityCodecs = true;      # LDAC HQ, aptX HD, etc.
    disableLibcameraMonitor = true;          # recommended for OBS/v4l2loopback users
  };
}
```

## Recommended companion packages (already included when enabled)

- `pavucontrol` – powerful GUI mixer
- `easyeffects` – system-wide EQ, compressor, RNNoise noise suppression
- `qpwgraph` / `helvum` – visual PipeWire patchbays
- `pasystray` (only when not using Plasma 6)

## Troubleshooting

Common issues and fixes when using this PipeWire module:

| Symptom                                      | Likely Cause                              | Solution / Check                                                                 |
|----------------------------------------------|-------------------------------------------|----------------------------------------------------------------------------------|
| No sound / applications don't see output     | PipeWire daemon not running               | `systemctl --user status pipewire pipewire-pulse wireplumber` – should be active |
| Crackling / xruns with lowLatency.proAudio   | Quantum too low for CPU / hardware        | Set `lowLatency.proAudio = false;` or increase `default.clock.min-quantum` to 128–256 |
| Bluetooth headset connects but low quality   | Codec negotiation failure                 | Try `bluetooth.highQualityCodecs = false;` temporarily; check `pavucontrol` → Configuration tab |
| Bluetooth headset has no audio / disconnects | PipeWire Bluetooth module issue           | `systemctl --user restart pipewire pipewire-pulse wireplumber`                  |
| OBS / games have high latency                | Default quantum too high for your use case| Enable `lowLatency.proAudio = true;` (test carefully – may cause xruns)         |
| Volume widget missing in Hyprland            | pasystray not running                     | Add `pasystray &` to your Hyprland config or use Waybar volume module           |
| Plasma 6 has duplicate volume icons          | pasystray running unnecessarily           | Disable pasystray (it is conditional in this module) or kill it manually        |
| Ardour / JACK apps have no realtime priority | rtkit not working                         | Ensure `security.rtkit.enable = true;` is not overridden elsewhere             |
| High CPU usage with many Bluetooth devices   | Aggressive codec scanning                 | Set `bluetooth.highQualityCodecs = false;` or limit codecs manually            |
| libcamera warnings / errors in logs          | WirePlumber still trying to monitor cams  | Confirm `disableLibcameraMonitor = true;` is set                               |

### Quick diagnostic commands

```bash
# Check running services
systemctl --user status pipewire pipewire-pulse wireplumber

# See current latency / quantum
pw-dump | grep -i quantum -A 5

# List available codecs for a Bluetooth device
pactl list cards | grep -A 20 "bluez"

# Check for xruns / errors
journalctl --user -u pipewire -u wireplumber -u pipewire-pulse -f
```

If issues persist, capture logs with the above commands and share them (remove any sensitive info).

## License

BSD 3-Clause License

Copyright © 2025–2026 DeMoD LLC

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.
3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

---

Made with ♥ by DeMoD LLC – Pushing the boundaries of NixOS + high-performance audio.
