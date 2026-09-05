# Custom Pwnagotchi Plugins

This repository contains all of my custom-made Pwnagotchi plugins.

Some of the older plugins were written quite a while ago, so the code may not follow current best practices or coding standards. I do not plan to refactor them purely for code quality, but I may fix bugs or compatibility issues when needed.

I may also add new plugins in the future, but there are no guarantees.

If you find a bug or have an issue, please open an issue. Pull requests are welcome, including fixes and improvements to existing plugins.

## Installation

### Network installation

Add the repository to `/etc/pwnagotchi/config.toml`:

```toml
main.custom_plugin_repos = [
    "https://github.com/pieterjvt/custom-pwnagotchi-plugins/archive/refs/heads/main.zip"
]
```

Then update the plugin list and install the plugin you want:

```bash
sudo pwnagotchi plugins update
sudo pwnagotchi plugins install <plugin>
```

### Manual installation

Clone or download the repository, then copy the plugins you want to the custom plugin directory:

```bash
sudo cp <filename> /usr/local/share/pwnagotchi/custom-plugins/<filename>
```

## Plugins

### Remote Cracking

Uploads captured handshakes to your own FTP server and cracks them using Hashcat.

#### Server requirements

* Python
* Hashcat
* OpenSSL
* `pyftpdlib`
* `pyopenssl`
* One or more wordlists

Install the Python dependencies:

```bash
python -m pip install pyftpdlib pyopenssl
```

On Debian-based Linux systems:

```bash
sudo apt-get install hashcat openssl
```

Generate a TLS certificate:

```bash
openssl req -new -x509 -days 365 -nodes -newkey rsa:2048 \
    -out cert.pem -keyout key.pem
```

Create directories for handshakes and wordlists:

```bash
mkdir handshakes wordlists
```

Place the following together in the same directory:

* [`remote_cracking_server.py`](./remote_cracking_server.py)
* `cert.pem`
* `key.pem`
* `handshakes/`
* `wordlists/`

Start the server:

```bash
python remote_cracking_server.py \
    -d handshakes \
    -c cert.pem \
    -k key.pem \
    -I /usr/bin/hashcat \
    -w wordlists \
    -P <password>
```

On Windows, change `-I` to the path of `hashcat.exe`.

For all available options:

```bash
python remote_cracking_server.py -h
```

#### Network setup

By default, the server uses:

* TCP port `8888`
* Passive FTP ports `49152-65534`

Allow these ports through your firewall. If the server needs to be accessible outside your local network, forward them on your router.

If your public IP changes regularly, you can use a Dynamic DNS service such as [DuckDNS](https://www.duckdns.org/).

#### Pwnagotchi configuration

The [`hashie-hcxpcapngtool`](https://github.com/PwnPeter/pwnagotchi-plugins#how-to-use) plugin is required.

```toml
main.plugins.hashie-hcxpcapngtool.enabled = true

main.plugins.remote_cracking.enabled = true
main.plugins.remote_cracking.server = "123.456.789.123" # or example.duckdns.org
main.plugins.remote_cracking.port = 8888
main.plugins.remote_cracking.user = "user"
main.plugins.remote_cracking.password = "yourpassword"
```

Optional:

```toml
main.plugins.remote_cracking.display_cracked = true
main.plugins.remote_cracking.potfile = "/root/remote_cracking.potfile"
main.plugins.remote_cracking.orientation = "vertical"
main.plugins.remote_cracking.position = "10,90"
```

### iPhone GPS

Saves GPS coordinates when a handshake is captured using your iPhone's GPS, Shortcuts and Bluetooth tethering.

#### Setup

1. Install the [iPhone GPS shortcut](https://routinehub.co/shortcut/19128/).
2. Follow the shortcut's setup instructions.
3. Create an iPhone automation that runs the shortcut when your Pwnagotchi connects over Bluetooth.

A stable `bt-tether` connection between the iPhone and Pwnagotchi is required.

#### Configuration

```toml
main.plugins.iphone_gps.enabled = true
```

Optional:

```toml
main.plugins.iphone_gps.use_last_loc = true
main.plugins.iphone_gps.compact_view = true
main.plugins.linespacing = 15
```

### Aftershake

Handles post-handshake processing in one plugin, including functionality from AircrackOnly, Hashie and Quickdic.

#### Requirements

Install Aircrack-ng:

```bash
sudo apt-get install aircrack-ng
```

If Hashie is enabled, also install hcxtools:

```bash
sudo apt-get install hcxtools
```

#### Configuration

Enable Aftershake and a supported GPS plugin:

```toml
main.plugins.gps.enabled = true
main.plugins.gps.device = "/dev/ttyUSB0"
main.plugins.gps.speed = 19200

main.plugins.aftershake.enabled = true
```

`gpsd` or `iphone_gps` can also be used instead of the standard GPS plugin.

Optional:

```toml
main.plugins.aftershake.wordlist_folder = "/root/custom_folder/"
main.plugins.aftershake.hashie = false
main.plugins.aftershake.face = "(>.O)"
main.plugins.aftershake.orientation = "vertical"
```

## Credits

* [PwnPeter](https://github.com/PwnPeter) for the easy plugin configuration system
* `junohea.mail@gmail.com` for `hashie-hcxpcapngtool`
* `pwnagotchi@rossmarks.uk` for `quickdic`
* `evilsocket@gmail.com` for `aircrackonly`
* `@nagy_craig` for `display-password`
* `33197631+dadav@users.noreply.github.com` for `wpa-sec`

## License

Licensed under the [MIT License](./LICENSE).
