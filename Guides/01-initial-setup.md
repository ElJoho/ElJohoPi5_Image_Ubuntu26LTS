# Guide 1 — Initial setup: Ubuntu Desktop 26.04.1 on the Pi 5, direct Ethernet, headless remote desktop (RDP) and fan at 100%

This guide takes a Raspberry Pi 5 from an empty microSD card to a running **Ubuntu Desktop 26.04.1 LTS** that:

1. boots and is fully updated,
2. is reachable from the Ubuntu PC over a **direct Ethernet cable** at `192.168.10.2`, the lab address from the UNPi5 image and the KIT guides,
3. shows its **GNOME (Wayland) desktop over RDP with no monitor attached**, through GNOME Remote Desktop, to Windows and Ubuntu PCs,
4. runs the **fan at 100% from boot**.

It stops there. ROS 2 Lyrical and the KIT Phantom X Pincher dependencies come in later guides.

> Checked on 2026-11-10 against the published image `ubuntu-26.04.1-preinstalled-desktop-arm64+raspi.img.xz` (its boot partition and package manifest), Ubuntu's source for the kernel it ships and Raspberry Pi Imager's OS list. Part 3 was checked on 2026-11-11 against the source of GNOME Remote Desktop 50.2, the version in the image. On 2026-11-26 its headless screen (3.6) was changed to the one in the published lab image `v2026.11.26` and in annex A.4 of the KIT remote-desktop guide ([`conexionEscritorioRemoto.md`](conexionEscritorioRemoto.md)): a 1328×748, 15 Hz virtual screen on HDMI1. **The full procedure hasn't been run on the Pi yet**: each part ends with the checks that prove it worked.
> Based on the Ubuntu Server guides ([ElJoho_Pi_Ros2_UbuntuServer26](https://github.com/ElJoho/ElJoho_Pi_Ros2_UbuntuServer26)) and the previous Desktop image ([UNPi5_Image_Ubuntu24_04](https://github.com/labsir-un/UNPi5_Image_Ubuntu24_04)).

---

## Values used in this guide

| Setting | Value | Notes |
|---|---|---|
| Board | Raspberry Pi 5 with the Active Cooler | 27 W USB-C power supply |
| Image | **Ubuntu Desktop 26.04.1 (64-bit)**, `ubuntu-26.04.1-preinstalled-desktop-arm64+raspi.img.xz` | 3,078,337,648 bytes to download; it expands to 9.55 GB. SHA-256 `8216f54cc1df2fcf7de37530a7edccdd200abd7dcfbc5d4f0407df7513544d6a` |
| Kernel in the image | `7.0.0-1017-raspi` | Newer after the update in Part 1 |
| Desktop | GNOME 50 on **Wayland only** (`gdm3` 50.1, `gnome-shell` 50.1) | GNOME 50 has no X11 session |
| Hostname / user / password | **`unpi`** / **`unpi`** / **`unpi`** | The user and password are the ones the KIT guides use. The hostname matches them; on the network the Pi is also `unpi.local`. The password is public on purpose, so see the warning below. |
| Pi cable address | `192.168.10.2/24`, gateway and DNS `192.168.10.1` | The same address as the old 24.04 lab image ([UNPi5](https://github.com/labsir-un/UNPi5_Image_Ubuntu24_04)) and the [KIT NoMachine guide](https://github.com/labsir-un/KIT_Phantom_X_Pincher_ROS2/blob/main/guias/Setup/conexionNoMachine.md). Students' Windows PCs set to `192.168.10.1` work unchanged. |
| PC cable address | `192.168.10.1` with the new `pi-lab-shared` profile (internet sharing, for setup), or `192.168.10.3` with your existing `pi-direct` profile (no sharing) | Both on `enp4s0`; see 1.5. The method is the same as in [Server Guide 3](https://github.com/ElJoho/ElJoho_Pi_Ros2_UbuntuServer26/blob/main/Guides/03-ubuntu-pc-ethernet-connection.md). |
| Remote desktop on the Pi | **GNOME Remote Desktop 50.2** (`gnome-remote-desktop`, already in the image), RDP on port **3389**, *Desktop Sharing* mode | Free and open source (GPL-2.0 or later). RDP login `unpi` / `unpi`. See Part 3. |
| Remote desktop clients | Windows: **Remote Desktop Connection** (`mstsc`, built in). Ubuntu PC: **Remmina** 1.4.43, installed in 3.5 | |
| Headless display | `HDMI-A-2` (the **HDMI1** port, the micro-HDMI port farther from the USB-C power) forced on with an EDID file that offers only **1328×748 at 15 Hz** | `HDMI-A-1` (**HDMI0**, next to the USB-C power) is left as it is, so a monitor there keeps working |

> [!WARNING]
> The password is public. **Never forward port 22 (SSH) or 3389 (RDP) to this Pi**, and never put it on a network that can be reached from the internet. Keep it on the direct cable or a private LAN.

### Where to type each command

| Where | Used for |
|---|---|
| **PC**: your Ubuntu PC's terminal (`johotan@…`) | Imager, `nmcli` on `enp4s0`, `ssh`, the RDP client (Remmina) |
| **Pi**: GNOME Terminal on the Pi's monitor (`Ctrl+Alt+T`) in Parts 1–3.6, then an SSH login from the PC | `apt`, `nmcli` on `eth0`, `/boot/firmware`, `/sys/...` |

`sudo` on the Pi asks for the Pi's password. Nothing appears while you type it.

---

## Part 1 — Write the card and do the first boot

### 1.1 Identify the card (PC)

```bash
lsblk -o NAME,SIZE,MODEL,TRAN,MOUNTPOINTS
```

Find the card by its size: a 64 GB card shows as about 58–60 GB. Your `nvme0n1` (Ubuntu and Windows) must never be chosen.

### 1.2 Write the card with Raspberry Pi Imager (PC)

1. Open Imager with `rpi-imager`. Version 1.8.5 is installed.
2. **CHOOSE DEVICE** → **Raspberry Pi 5**.
3. **CHOOSE OS** → **Other general-purpose OS** → **Ubuntu** → **Ubuntu Desktop 26.04.1 (64-bit)**. Don't pick Desktop 26.10, Server 26.04.1 or Desktop 24.04.5.
4. **CHOOSE STORAGE** → the card from 1.1. Leave **Exclude system drives** ticked.
5. **NEXT.** Imager goes straight to the erase warning. **It doesn't offer OS customisation for Ubuntu Desktop**, because its OS list marks this image as `"init_format": "none"`. The hostname, user, Wi-Fi and SSH are therefore set at the Pi in 1.6–2.1. If a customisation dialog appears anyway, answer **NO**; those settings wouldn't be applied.
6. **YES** → enter your PC password. Imager downloads, writes and verifies the image, which takes about 10–30 minutes depending on the card.

**Optional route B, a file you downloaded yourself.** In step 3, choose **Use custom** and pick the `.img.xz` file; don't unpack it. To download and check it:

```bash
mkdir -p ~/Documents/images && cd ~/Documents/images
wget https://cdimage.ubuntu.com/releases/26.04.1/release/ubuntu-26.04.1-preinstalled-desktop-arm64+raspi.img.xz
wget https://cdimage.ubuntu.com/releases/26.04.1/release/SHA256SUMS
sha256sum -c SHA256SUMS --ignore-missing     # must print: ...desktop-arm64+raspi.img.xz: OK
```

### 1.3 Look at the boot partition before the first boot (PC, read-only)

Take the card out and put it back in. Ubuntu mounts its two partitions, **`system-boot`** (FAT) and **`writable`** (the system).

```bash
ls /media/$USER/system-boot/
ls /media/$USER/system-boot/cmdline.txt               # expected: No such file or directory
grep -n os_prefix /media/$USER/system-boot/config.txt
cat /media/$USER/system-boot/current/cmdline.txt; echo
cat /media/$USER/system-boot/current/state
```

The 26.04.1 image contains this:

```
2:os_prefix=current/
5:os_prefix=new/
zswap.enabled=1 zswap.compressor=zstd multipath=off dwc_otg.lpm_enable=0 console=tty1 root=LABEL=writable rootfstype=ext4 panic=10 rootwait fixrtc quiet splash
good
```

- **What it means:** the bootloader reads `config.txt` from the root of the partition. `os_prefix=current/` makes it load the kernel, initramfs, device trees, overlays **and `cmdline.txt`** from `current/`. Before the first boot, the kernel command line is therefore in **`/media/$USER/system-boot/current/cmdline.txt`**. There is no `cmdline.txt` at the root.
- **Don't edit it yet.** The headless settings point to an EDID file that only exists on the Pi after 3.6. You add them in Part 3, after setup.

Eject the card (the ⏏ next to `system-boot` in Files) and take it out.

### 1.4 The first boot needs a monitor, keyboard and mouse

**Yes, this one time.** There are three reasons:

- Imager can't pre-configure Ubuntu Desktop (1.2, step 5).
- The image creates your account with GNOME's interactive first-run setup (`gnome-initial-setup` 50).
- **The image has no SSH server.** `openssh-server` isn't in its package list, so nothing can log in remotely until you install it in 2.1.

You need:

- A monitor on **HDMI0**, the micro-HDMI port closest to the USB-C power socket, through a micro-HDMI to HDMI cable.
- A USB keyboard and a USB mouse.

You can unplug them in **3.7**. From then on, nothing needs a monitor.

> The image's `user-data` file notes that cloud-init could create the user instead of the wizard. That route isn't covered or tested here.

### 1.5 Get the PC's cable ready (PC)

This Pi uses the lab subnet `192.168.10.x`. Your PC already has a profile for it, `pi-direct` (`192.168.10.3`), but that profile doesn't share the PC's internet. During setup, the Pi needs internet through the cable for updates and package installs.

So create one more profile, in the same way as `pi-shared` in Server Guide 3, but on the lab subnet. Run this **once**:

```bash
nmcli connection add type ethernet ifname enp4s0 con-name pi-lab-shared \
  ipv4.method shared ipv4.addresses 192.168.10.1/24 connection.autoconnect no
```

Expected: `Connection 'pi-lab-shared' (…) successfully added.`

Your PC then has three profiles on `enp4s0`:

| Profile | PC address | Pi gets internet? | Used for |
|---|---|---|---|
| **`pi-lab-shared`** (new) | `192.168.10.1/24`, internet sharing | ✅ Yes | **This Pi, during setup** and whenever it needs `apt` |
| `pi-direct` (existing, unchanged) | `192.168.10.3/24` | ❌ No | This Pi and the old lab Pis, for SSH and the remote desktop only |
| `pi-shared` (existing, unchanged) | `192.168.77.1/24`, internet sharing | — | The Ubuntu Server card at `192.168.77.2` |

- **`connection.autoconnect no`:** the new profile only starts when you run `nmcli connection up pi-lab-shared`. It never starts by itself, which matters on a port that can be plugged into another network, because it hands out addresses.
- **`pi-direct` and `pi-shared` both still connect automatically.** Either one can win when the cable gets a link, so always switch on the one you want yourself.
- **The Pi's addresses:** the Pi keeps its fixed `192.168.10.2` in every case. With `pi-lab-shared` it also gets an extra address from `192.168.10.10–254`.

### 1.6 First boot (at the Pi)

1. Put the card in the Pi. Connect the Ethernet cable to the PC, the monitor to HDMI0, and the keyboard and mouse. **Connect the power last.**
2. On the PC, once the Pi is booting:

   ```bash
   cat /sys/class/net/enp4s0/carrier      # 1 = the cable has a link
   nmcli connection up pi-lab-shared      # "Connection successfully activated"
   ```

3. Wait. The first boot takes a few minutes: the system partition is resized and a swap file is created. Don't unplug the Pi.
4. Fill in the first-run setup on the monitor:

   | Page | Value |
   |---|---|
   | Language | Your choice |
   | Keyboard / typing | Spanish (Latin American), the same as `latam` in the Server guide |
   | Network / Wi-Fi | Skip it: the cable already gives internet through `pi-lab-shared`. Join Wi-Fi only if you want it. |
   | Privacy | Your choice |
   | Time zone | America/Bogota |
   | About you | Full name `unpi`, username **`unpi`** |
   | Password | **`unpi`**. If the wizard rejects it as too weak, enter a temporary stronger one and set `unpi` in 1.7. |

5. The wizard logs you in to GNOME.

### 1.7 First checks (Pi, GNOME Terminal: `Ctrl+Alt+T`)

```bash
grep PRETTY_NAME /etc/os-release       # PRETTY_NAME="Ubuntu 26.04.1 LTS"
uname -r                               # 7.0.0-1017-raspi
echo $XDG_SESSION_TYPE                 # wayland
id -nG | grep -w sudo                  # your account is an administrator
nmcli device status                    # eth0  ethernet  connected  (a 192.168.10.x DHCP address from pi-lab-shared for now)
ping -c 3 ubuntu.com                   # replies = the Pi is online
```

Set the hostname, because the wizard doesn't ask for one. Set the public password too if 1.6 needed a temporary one:

```bash
sudo hostnamectl set-hostname unpi
hostnamectl hostname                        # unpi
echo 'unpi:unpi' | sudo chpasswd    # only if the wizard refused the public password
```

### 1.8 Update and reboot (Pi)

```bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```

- **"Waiting for cache lock … (unattended-upgr)"** means Ubuntu's automatic updates are running. Let `apt` wait; this can take 10–30 minutes. Never kill the updater or delete lock files.
- **After a kernel update the Pi boots twice.** Ubuntu test-boots the new kernel from `new/` before keeping it (its A/B boot). Don't cut the power.

When the desktop is back, open a terminal:

```bash
uname -r                               # the new kernel version
cat /boot/firmware/current/state       # good
ls /boot/firmware/                     # current/ and, after an update, old/. No new/ left behind.
```

If a `new/` folder is still there with `bad` in `new/state`, the new kernel failed its test boot and the Pi fell back to `current/`. Run `sudo apt full-upgrade` and reboot again.

✅ **Part 1 done:** Ubuntu Desktop 26.04.1 boots and is up to date.

---

## Part 2 — Direct Ethernet between the PC and the Pi

The method is the same as in the Server guide: one fixed address on the Pi's cable port, and a profile on the PC. Two things differ:

- **The address** is the lab one, `192.168.10.2`.
- **The Pi uses NetworkManager**, not `systemd-networkd`, because that's how Ubuntu Desktop manages the network. The Server guide's `/etc/netplan/90-direct.yaml` is replaced by a NetworkManager profile.

| | Server guide | This image |
|---|---|---|
| Network service on the Pi | `systemd-networkd` (netplan file) | NetworkManager (`nmcli` profile `pi-cable`) |
| Pi address | `192.168.77.2/24` plus DHCP | `192.168.10.2/24` fixed, with `192.168.10.1` as gateway and DNS |
| PC profile | `pi-shared` (`192.168.77.1`) | `pi-lab-shared` (`192.168.10.1`, with internet) or `pi-direct` (`192.168.10.3`, without) |
| Internet through the cable | Yes, with `pi-shared` | Yes with `pi-lab-shared`. No with `pi-direct`, or with a Windows PC set up as in the KIT guide. |
| SSH server | Already in the image | Must be installed (2.1) |

**Why a fixed address instead of DHCP plus a fixed one:**

- **The problem with DHCP:** on a PC that doesn't share its connection, nothing answers DHCP. NetworkManager can then give up on the whole profile after its timeout, and the fixed address goes down with it.
- **What the fixed address gives you:** it always comes up, whichever PC profile is active. With `pi-lab-shared` on the PC, the Pi also gets internet through the PC.
- **Wi-Fi stays first:** `route-metric 700` ranks the cable after Wi-Fi (600), so if you later add Wi-Fi, the Pi uses Wi-Fi for internet first.

### 2.1 Install the SSH server (Pi)

```bash
sudo apt install -y openssh-server
ss -tln 'sport = :22'                                # LISTEN on 0.0.0.0:22 and [::]:22
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub     # write down this fingerprint for 2.4
```

Ubuntu starts SSH on demand through `ssh.socket`, so `ssh.service` may show as inactive until the first login. That's normal.

### 2.2 Give `eth0` its fixed address (Pi)

```bash
nmcli device status | grep ethernet       # the cable port, normally eth0. If it's different, use that name below.
sudo nmcli connection add type ethernet ifname eth0 con-name pi-cable \
  ipv4.method manual ipv4.addresses 192.168.10.2/24 \
  ipv4.gateway 192.168.10.1 ipv4.dns 192.168.10.1 ipv4.route-metric 700 \
  ipv6.method auto ipv6.addr-gen-mode eui64 \
  connection.autoconnect yes connection.autoconnect-priority 10
sudo nmcli connection up pi-cable
```

| Setting | What it does |
|---|---|
| `ipv4.method manual`, `ipv4.addresses 192.168.10.2/24` | The fixed lab address, the same one as the old 24.04 image and the KIT guides |
| `ipv4.gateway`, `ipv4.dns 192.168.10.1` | Internet and name lookups go through the PC when `pi-lab-shared` is on. With `pi-direct` or a Windows PC, nobody answers there; that's harmless, and the Pi simply has no internet over the cable. |
| `ipv4.route-metric 700` | Wi-Fi (600) is preferred for internet whenever it's connected |
| `ipv6.addr-gen-mode eui64` | Keeps the hardware-based link-local address the Server guide used for rescue (`fe80::2ecf:67ff:fe6c:4e6e` on that board). Without it, NetworkManager may pick a different, hash-based one. |
| `autoconnect-priority 10` | Wins over the automatic "Wired connection 1" profile at every boot |

> Run the `add` command **only once**. Running it twice creates two profiles with the same name. To check and clean up: `nmcli -f NAME,UUID connection show | grep pi-cable`, then `sudo nmcli connection delete <UUID>` for the extra one.

### 2.3 Check on the Pi

```bash
nmcli -f NAME,DEVICE,AUTOCONNECT,AUTOCONNECT-PRIORITY connection show
ip -br addr show eth0        # eth0  UP  192.168.10.2/24 fe80::2ecf:67ff:fe6c:4e6e/64
ip route | grep default      # default via 192.168.10.1 dev eth0 proto static metric 700
ping -c 3 192.168.10.1       # the PC answers (pi-lab-shared is on)
ping -c 3 ubuntu.com         # internet through pi-lab-shared
```

### 2.4 Check from the PC

```bash
ssh-keygen -R 192.168.10.2          # forget the old 24.04 lab card's identity: same address, different card
ping -c 3 192.168.10.2              # three replies, each under 1 ms
ssh unpi@192.168.10.2           # compare the fingerprint with 2.1, answer yes, password unpi
```

Inside the SSH session, `hostname` must print `unpi`. Type `exit` to leave.

**Check the everyday profile too.** It doesn't share the PC's internet, but it must reach the Pi:

```bash
nmcli connection up pi-direct       # PC becomes 192.168.10.3
ping -c 3 192.168.10.2
ssh unpi@192.168.10.2 hostname  # prints unpi
nmcli connection up pi-lab-shared   # back to sharing, so the Pi has internet again
```

**Rescue path, if `192.168.10.2` doesn't answer:**

```bash
ping -6 -c 3 ff02::1%enp4s0                       # every device on the cable answers; the one that isn't the PC is the Pi
ssh unpi@fe80::2ecf:67ff:fe6c:4e6e%enp4s0     # the link-local address (same board as the Server guide)
```

✅ **Part 2 done:** `ssh unpi@192.168.10.2` works over the cable with both PC profiles, and with `pi-lab-shared` the Pi has internet through the PC.

---

## Part 3 — Remote desktop (RDP) on Wayland with no monitor

### How it works on 26.04

- **Why GNOME Remote Desktop and not NoMachine:**
  - **NoMachine 10 isn't free on the Pi side.** It needs a Personal Edition subscription (US$24.50 per Pi) or a 14-day evaluation.
  - **GNOME Remote Desktop 50.2 is already in the image** and is free and open source.
  - **It speaks RDP**, so a Windows PC connects with its built-in Remote Desktop Connection, and the Ubuntu PC uses Remmina.
- **The mode used here is *Desktop Sharing*.** The upstream README calls it "remote assistance". The client sees, and controls, the session that is already running on the Pi. GNOME Remote Desktop has two other modes that this guide doesn't use:
  - ***Remote Login*** gives each client its own session through GDM's login screen. Each client then chooses the screen size, so the Pi may have to encode a much larger screen.
  - ***Headless (single user)*** needs a separately started headless GNOME session, which Ubuntu doesn't set up.
- **Desktop Sharing still needs a real screen output:**
  - **What it shares:** the session drawn on a real output, as NoMachine did.
  - **The problem without a monitor:** GNOME has nothing to draw on, so there's nothing to share.
  - **The fix:** two kernel settings make HDMI1 (`HDMI-A-2`) look connected to a small monitor. One gives that port an **EDID file** (the data a monitor normally sends to say which modes it accepts) that offers only 1328×748 at 15 Hz. The other forces the port on. GNOME then keeps drawing the desktop there. Together they do in software what a dummy HDMI plug does.
  - **Side benefit for the CPU:** the shared screen is that fixed output. Whatever window size a client uses, the Pi never encodes more than 1328×748, and at most 15 frames per second.
  - **Why 1328×748:** it's the smallest almost-16:9 screen that fits the whole **DYNAMIXEL Wizard 2.0** window. That window can't be shorter than 678 pixels, and 748 = 678 + GNOME's top bar + the title bar. RViz also fits with normal-size text.
  - **Why 15 Hz:** it's the lowest rate at which typing and moving windows still feel smooth. The client draws the mouse pointer itself, so the pointer moves smoothly anyway.
  - **Why HDMI1:** HDMI0 isn't touched, so a monitor plugged into HDMI0 still works normally.
- **How the screen is compressed:** the Pi 5 has no hardware H.264 encoder. GNOME Remote Desktop only uses H.264 through VA-API or NVIDIA hardware, so on the Pi it uses its software **RFX Progressive** codec, on the CPU. That codec re-encodes only the parts of the screen that changed. 3.10 measures the load.
- **Two things Desktop Sharing depends on:**
  - **The login keyring:** Desktop Sharing keeps its RDP username and password in GNOME's login keyring, which automatic login doesn't unlock. 3.4 handles this.
  - **No screen lock:** locking the screen closes the connection, so 3.4 also turns the lock off.

### 3.1 Confirm where the kernel command line lives (Pi)

```bash
findmnt -no SOURCE,TARGET,FSTYPE /boot/firmware   # /dev/mmcblk0p1 /boot/firmware vfat
ls /boot/firmware/cmdline.txt                     # No such file or directory  ← expected
grep -n os_prefix /boot/firmware/config.txt       # 2:os_prefix=current/  and  5:os_prefix=new/
ls -d /boot/firmware/{current,new,old} 2>/dev/null
cat /boot/firmware/current/state                  # good
cat /boot/firmware/current/cmdline.txt; echo
cat /proc/cmdline
```

| When | Path of the kernel command line |
|---|---|
| Before the first boot (card in the PC) | `/media/$USER/system-boot/current/cmdline.txt` |
| On the running Pi | **`/boot/firmware/current/cmdline.txt`** |
| `/boot/firmware/cmdline.txt` | **Doesn't exist on 26.04.** Since Ubuntu 25.10 the file lives in `current/`. Creating it does nothing, because the bootloader only reads `current/cmdline.txt` (`os_prefix=current/`). |

- **Kernel updates keep your edit.** `flash-kernel` writes a new kernel to `new/` and copies `current/cmdline.txt` into `new/cmdline.txt`. After the test boot, `new/` becomes `current/`. An edit made to `current/` is therefore carried forward.
- **If `new/` exists right now** (a kernel was installed and the Pi hasn't rebooted yet), its copy must be edited too. The loop in 3.6 does both.
- **`/proc/cmdline`** is what the kernel actually received. The bootloader puts its own settings in front, so your `cmdline.txt` appears at the end.

### 3.2 Check that the kernel supports the headless settings (Pi)

```bash
ls /sys/class/drm/ | grep HDMI                                # cardN-HDMI-A-1 and cardN-HDMI-A-2
grep CONFIG_DRM_LOAD_EDID_FIRMWARE /boot/config-$(uname -r)   # CONFIG_DRM_LOAD_EDID_FIRMWARE=y
```

| Setting | Works on 26.04.1? | Evidence |
|---|---|---|
| `drm.edid_firmware=HDMI-A-2:edid/1328x748-15.bin` | **Yes** | Ubuntu's raspi kernel is built with `CONFIG_DRM_LOAD_EDID_FIRMWARE=y` (checked in `7.0.0-1023` and `7.0.0-1024`). The kernel then reads that port's EDID from `/lib/firmware/edid/1328x748-15.bin` instead of asking the port. The Pi 5 driver names the HDMI1 port `HDMI-A-2` (check with the `ls` above). |
| `video=HDMI-A-2:1328x748@15D` | **Yes** | A standard kernel display parameter: 1328×748 at 15 Hz, and `D` = force this output on even with nothing plugged in. |
| The file | **Changed** | `/boot/firmware/current/cmdline.txt`, not `/boot/firmware/cmdline.txt` |

Both settings are in the published lab image (`v2026.11.26`), where they give `HDMI-A-2: connected` with the single mode `1328x748`.

**Why the EDID file is needed:** without a monitor, Linux doesn't know which modes the port accepts, so it uses a standard list (1024×768, 848×480, 800×600, all at about 60 Hz). With the file, the port offers exactly 1328×748 at 15 Hz, and nothing else.

> **Side effect:** a monitor plugged into **HDMI1** gets only 1328×748 at 15 Hz, which most monitors show badly or not at all. Use **HDMI0** for a monitor; it isn't changed. With a monitor on HDMI0, GNOME sees two screens, the monitor and the virtual one, and **Settings → Displays** chooses which to use.

### 3.3 Set up GNOME Remote Desktop (Pi, terminal at the monitor, as `unpi`)

Run every command in this step as `unpi`, **without `sudo`**. Desktop Sharing belongs to your user session: its settings, its TLS files and its login all live in your account.

```bash
dpkg-query -W gnome-remote-desktop          # gnome-remote-desktop  50.2-0ubuntu0.1, or newer after 1.8
```

**1. Create the TLS certificate.** RDP connections are encrypted, and GNOME Remote Desktop doesn't create a certificate by itself. `openssl` is already in the image.

```bash
mkdir -p ~/.local/share/gnome-remote-desktop
openssl req -new -newkey rsa:4096 -days 3650 -noenc -x509 -subj /CN=unpi \
  -keyout ~/.local/share/gnome-remote-desktop/tls.key \
  -out ~/.local/share/gnome-remote-desktop/tls.crt
chmod 600 ~/.local/share/gnome-remote-desktop/tls.key
openssl x509 -in ~/.local/share/gnome-remote-desktop/tls.crt -noout -subject -enddate
```

Expected: a few lines of dots and `+` while the key is generated, then `subject=CN=unpi` and a `notAfter=` date ten years from today.

**2. Configure Desktop Sharing and switch it on:**

```bash
grdctl rdp set-tls-key ~/.local/share/gnome-remote-desktop/tls.key
grdctl rdp set-tls-cert ~/.local/share/gnome-remote-desktop/tls.crt
grdctl rdp set-credentials unpi unpi
grdctl rdp disable-view-only
grdctl rdp disable-port-negotiation
grdctl rdp enable
```

No error means success. `rdp enable` also enables and starts your user service, `gnome-remote-desktop.service`.

| Command | Default | Why |
|---|---|---|
| `set-tls-key`, `set-tls-cert` | none | Without them the service doesn't accept connections. Its log says "RDP TLS certificate and key not yet configured properly". |
| `set-credentials unpi unpi` | none | The **RDP login**. It's separate from the Linux account; the lab login is reused so there's one less password to remember. It's stored in your login keyring. |
| `disable-view-only` | view-only **on** | Lets the client use the mouse and keyboard, not just watch |
| `disable-port-negotiation` | **on** | With negotiation on, the service quietly moves to another port when 3389 is busy, and clients that only type the address can't find it. Off, it always uses 3389. |
| Screen-share mode (not changed) | `mirror-primary` | The client sees the primary screen: once headless (3.6), that's HDMI1's 1328×748 virtual screen. The other mode, `extend`, adds a virtual monitor, which means more pixels to encode. |

**3. Check:**

```bash
grdctl status
systemctl --user is-enabled gnome-remote-desktop.service      # enabled
systemctl --user is-active gnome-remote-desktop.service       # active
ss -tln 'sport = :3389'                                       # a LISTEN line for port 3389
```

In the `grdctl status` output, look for these lines. It also prints a `VNC:` block, which stays `disabled`:

```
Overall:
	Unit status: active
RDP:
	Status: enabled
	Port: 3389
	TLS fingerprint: …            ← write this down for 3.5
	View-only: no
	Negotiate port: no
	Username: (hidden)
	Password: (hidden)
```

- **Settings shows the same configuration.** **Settings → System → Remote Desktop → Desktop Sharing** shows *Desktop Sharing* and *Remote Control* turned on. **Leave the *Remote Login* tab off**: it's the other mode, and it would compete for port 3389.
- **Firewall:** `ufw` is installed but off (`sudo ufw status` → `inactive`). If you ever turn it on, allow RDP only on the cable: `sudo ufw allow in on eth0 to any port 3389 proto tcp`.

### 3.4 Automatic login, no lock, no blanking, no suspend, an open keyring (Pi, at the monitor)

Desktop Sharing shows the session running on the Pi's screen (the monitor now, HDMI1's virtual screen after 3.6), so these four things must hold:

- A user must be logged in.
- The screen must never lock, because locking closes the connection.
- The screen must never sleep.
- The login keyring must be readable after an automatic login.

1. **Settings → System → Users →** Unlock → turn on **Automatic Login** for `unpi`.

   If the Users panel doesn't offer the switch, edit `/etc/gdm3/custom.conf` with `sudo nano` so the `[daemon]` section contains:

   ```ini
   [daemon]
   AutomaticLoginEnable=true
   AutomaticLogin=unpi
   ```

2. In a terminal on the Pi, as `unpi` (no `sudo`):

   ```bash
   gsettings set org.gnome.desktop.session idle-delay 0
   gsettings set org.gnome.desktop.screensaver lock-enabled false
   gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
   ```

3. Check:

   ```bash
   grep -iE '^\s*AutomaticLogin' /etc/gdm3/custom.conf                        # AutomaticLoginEnable=True and AutomaticLogin=unpi
   gsettings get org.gnome.desktop.session idle-delay                         # uint32 0
   gsettings get org.gnome.desktop.screensaver lock-enabled                   # false
   gsettings get org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type   # 'nothing'
   ```

4. **Give the login keyring an empty password.**
   - **The problem:** automatic login doesn't unlock GNOME's login keyring, and that's where Desktop Sharing keeps its RDP login (3.3). After a reboot the service can't read the login, so nobody can connect. Meanwhile an "unlock keyring" prompt waits on the HDMI output, where nobody sees it.
   - **The fix:** a keyring with an empty password is always open.

   Do this at the monitor:
   1. Open **Passwords and Keys** (Activities → type `passwords`).
   2. Under **Passwords**, right-click **Login** (or open its ⋮ menu) → **Change Password**.
   3. As the old password, type the one you used in the first-run wizard. That's `unpi`, or the temporary password if 1.6 needed one, because `chpasswd` in 1.7 doesn't change the keyring.
   4. Leave the new password **empty**, confirm, and choose **Continue** when it warns that the passwords will be stored unencrypted.

   Check, in the terminal:

   ```bash
   head -n 1 ~/.local/share/keyrings/login.keyring      # [keyring]  ← stored unencrypted, so no unlock is needed
   ```

   If it prints `GnomeKeyring` and some binary characters instead, the keyring is still encrypted. Repeat step 4.

   > **The trade-off:** every password saved in that keyring is stored unencrypted on the card. That includes the RDP login, Wi-Fi passwords and browser passwords. The lab password is public anyway, but don't save personal passwords on this image.

### 3.5 First RDP test, with the monitor still attached

This proves that the remote desktop works before the monitor comes off. The monitor still shows its own resolution at this point; the 1328×748 virtual screen comes in 3.6.

**From the Ubuntu PC (Remmina):**

```bash
sudo apt install -y remmina remmina-plugin-rdp
remmina -c rdp://unpi@192.168.10.2
```

1. **Certificate:** Remmina shows the Pi's certificate. Its fingerprint should match the `TLS fingerprint` from 3.3. Accept it.
2. **Password:** `unpi`.
3. You should see the same desktop as on the monitor, and the mouse and keyboard should work.

**From a Windows PC (Remote Desktop Connection, built in).** The Windows PC's cable port must be on `192.168.10.x`, for example `192.168.10.1` as in the KIT guide.

1. Press `Win+R`, type `mstsc` and press Enter.
2. **Computer:** `192.168.10.2`. The size of the remote desktop is set by the Pi's screen, so the **Display** tab can stay as it is.
3. **Connect.** When it asks for credentials, choose **More choices → Use a different account**, then enter user `unpi` and password `unpi`.
4. **Certificate:** "The identity of the remote computer cannot be verified" is expected, because the certificate is self-signed. Choose **Yes**.

### 3.6 Add the virtual screen on HDMI1 (Pi)

**1. Copy the EDID file.** It's the same file as in the KIT repository (`guias/Setup/scripts/1328x748-15.bin`): 128 bytes, one mode, 1328×748 at 15.000 Hz.

```bash
sudo mkdir -p /lib/firmware/edid
sudo wget -O /lib/firmware/edid/1328x748-15.bin \
  https://raw.githubusercontent.com/ElJoho/ElJoho_KIT_Phantom_X_Pincher_ROS2/main/guias/Setup/scripts/1328x748-15.bin
sha256sum /lib/firmware/edid/1328x748-15.bin
```

Expected: `725660a918e68522fad4503d3b3bdf2dda27f6fd8f80e5fa77b680d0e3b3e03b  /lib/firmware/edid/1328x748-15.bin`. If the KIT repository is already cloned on the Pi, `sudo cp ~/ros2/KIT_Phantom_X_Pincher_ROS2/guias/Setup/scripts/1328x748-15.bin /lib/firmware/edid/` does the same.

**2. Append the two settings to the single line:**

```bash
cp /boot/firmware/current/cmdline.txt ~/cmdline.txt.orig
for f in /boot/firmware/current/cmdline.txt /boot/firmware/new/cmdline.txt; do
  [ -f "$f" ] || continue
  grep -q 'drm.edid_firmware=' "$f" || sudo sed -i '1 s#$# drm.edid_firmware=HDMI-A-2:edid/1328x748-15.bin video=HDMI-A-2:1328x748@15D#' "$f"
  echo "== $f"; cat "$f"; echo
  awk 'END { print NR " line(s)" }' "$f"
done
```

Expected (plus a `new/` block if that folder exists):

```
== /boot/firmware/current/cmdline.txt
zswap.enabled=1 zswap.compressor=zstd multipath=off dwc_otg.lpm_enable=0 console=tty1 root=LABEL=writable rootfstype=ext4 panic=10 rootwait fixrtc quiet splash drm.edid_firmware=HDMI-A-2:edid/1328x748-15.bin video=HDMI-A-2:1328x748@15D
1 line(s)
```

| Part | What it does |
|---|---|
| `drm.edid_firmware=HDMI-A-2:edid/1328x748-15.bin` | HDMI1 reads its EDID from the file of step 1 (the path is relative to `/lib/firmware/`) |
| `video=HDMI-A-2:1328x748@15D` | Uses that mode, and `D` forces the port on with nothing plugged in |
| `1 s#$# …#` | Adds the two settings, separated by spaces, at the **end of line 1**. The bootloader ignores anything after the first line. |
| `grep -q … \|\|` | Running the block a second time doesn't add them twice |
| `1 line(s)` | Proves the file is still a single line. It ships without a final newline, and `sed` keeps it that way. |

**To undo:** `sudo cp ~/cmdline.txt.orig /boot/firmware/current/cmdline.txt`. With no SSH, put the card in the PC and edit `/media/$USER/system-boot/current/cmdline.txt` instead. The EDID file can stay; without the settings nothing uses it.

### 3.7 Go headless (Pi)

```bash
sudo poweroff
```

Wait for the **red** LED. Unplug the monitor, keyboard and mouse, and **leave the Ethernet cable connected**. Power the Pi back on (press its power button, or unplug and replug the power) and wait about 2 minutes.

### 3.8 Verify over SSH (PC → Pi)

```bash
ssh unpi@192.168.10.2
```

Then, on the Pi:

```bash
tr ' ' '\n' < /proc/cmdline | grep -E '^(drm\.edid_firmware=|video=)'
for c in /sys/class/drm/card*-HDMI-A-*; do
  printf '%s: %s, %s, modes: %s\n' "${c##*/}" "$(cat "$c/status")" "$(cat "$c/enabled")" "$(paste -sd' ' "$c/modes")"
done
for s in $(loginctl list-sessions --no-legend | awk '{print $1}'); do
  loginctl show-session "$s" -p Name -p Type -p Class -p Active | paste -sd' '
done
systemctl --user is-active gnome-remote-desktop.service
ss -tln 'sport = :3389' | grep -c LISTEN
grdctl status | grep -E 'Unit status|Username'
```

Expected (the card number may differ):

```
drm.edid_firmware=HDMI-A-2:edid/1328x748-15.bin
video=HDMI-A-2:1328x748@15D
card1-HDMI-A-1: disconnected, disabled, modes:
card1-HDMI-A-2: connected, enabled, modes: 1328x748
Name=unpi Type=wayland Class=user Active=yes
Name=unpi Type=tty Class=user Active=…      ← your SSH login; normal
active
1                                           ← at least 1 LISTEN line for port 3389
	Unit status: active
	Username: (hidden)
```

- **`connected`** on `HDMI-A-2` means the fake connection works.
- **`modes: 1328x748`**, and nothing else, means the EDID file was read. If it lists `1024x768 848x480 …` instead, see Troubleshooting.
- **`enabled`** means GNOME is actually drawing on that output. That's the part the remote desktop needs.
- **`Type=wayland … Active=yes`** means automatic login worked.
- **`active`, a `LISTEN` line and `Username: (hidden)`** mean Desktop Sharing started with the session and could read its login from the keyring. If `grdctl` prints `Failed to read credentials` instead, the keyring is still locked; see 3.4 step 4.

### 3.9 Connect over RDP without a monitor (PC or Windows)

Connect as in 3.5, then check:

- [ ] The desktop appears, and it isn't black.
- [ ] **Settings → Displays** shows one display at **1328 × 748** with a refresh rate of **15.00 Hz**. It's the only size offered, because the EDID file has a single mode.
- [ ] In a terminal inside the RDP window, `echo $XDG_SESSION_TYPE` prints `wayland`.
- [ ] **Reboot test:** disconnect the client, run `ssh -t unpi@192.168.10.2 sudo reboot` on the PC, wait about 2 minutes, and connect again without touching the Pi.

**If the desktop doesn't appear:** first check that 3.8 shows `connected, enabled`, `Type=wayland` and the three Desktop Sharing lines. If it does and the window is still black, plug a **dummy HDMI plug** into HDMI0 and reboot. Set the plug to a small size in **Settings → Displays**, because many plugs report 1920 × 1080 or 4K, which is 2 to 8 times the pixels of 1328 × 748 to encode.

### 3.10 Measure the load on the Pi (PC → Pi over SSH, RDP client connected)

This records what the remote desktop really costs on this Pi. Keep the RDP window open, and in a **second** PC terminal, log in with `ssh unpi@192.168.10.2`.

First find the two processes that do the work: GNOME Shell captures the screen, and the remote desktop service compresses it.

```bash
PIDS="$(pgrep -d, -x gnome-shell),$(pgrep -d, -f gnome-remote-desktop-daemon)"; echo "$PIDS"   # two numbers, e.g. 1834,2210
```

For each scene below, start the scene, then run:

```bash
top -b -d 5 -n 7 -p "$PIDS" | grep -E '^%Cpu|gnome-'
```

It prints 7 samples, 5 seconds apart. **Ignore the first sample**, because it doesn't cover a full 5-second interval. In each of the others, read these values:

| Value in `top` | Meaning |
|---|---|
| `%CPU` on the `gnome-remote-de` line | The compression. 100 = one full core; the Pi 5 has four, so 400 = all of them. |
| `%CPU` on the `gnome-shell` line | Drawing and capturing the screen |
| `id` on the `%Cpu(s)` line | Idle share of the **whole** CPU. The total load is 100 − `id`. |

| Scene | What to do in the RDP window during the 35 seconds | `gnome-remote-de` %CPU | `gnome-shell` %CPU | Total (100 − `id`) |
|---|---|---|---|---|
| A. Baseline | **Disconnect** the client first | | | |
| B. Still desktop | Connect, then touch nothing | | | |
| C. Terminal output | In a terminal in the RDP window: `ls -lR /usr` (stop it with `Ctrl+C` afterwards) | | | |
| D. Constant motion | Drag a window around the screen without stopping | | | |

- **Fill in the table on the Pi and keep it.** It's the real answer to "how much does the remote desktop load the Pi". RViz and Gazebo, the heavy ROS 2 tools, are measured the same way in the ROS 2 guide.
- **If B already uses a lot of CPU:** something on the screen is moving even when you don't touch it, such as a clock with seconds or an animation. Close it and measure again.

✅ **Part 3 done:** an RDP client on Windows or on the PC shows the Wayland desktop with no monitor, also after a reboot, and the load table from 3.10 is filled in.

---

## Part 4 — Fan at 100% from boot

**How it works:**

- **Who drives the fan:** the Linux `pwm-fan` driver, at four "cooling levels" tied to CPU temperature points.
- **What Ubuntu's device tree sets:** the levels switch on at 50, 60, 67.5 and 75 °C, at speeds 75, 125, 175 and 250 out of 255.
- **What this part changes:** `config.txt` settings set **every level to 255** and **move the first switch-on point down to 10 °C**. A running Pi 5 is always warmer than that, so the fan runs at 255 from the moment Linux loads the fan driver, in the first seconds of boot.
- **Checked on the image:** its Pi 5 device tree (`current/bcm2712-rpi-5-b.dtb`, and the same for `bcm2712d0-rpi-5-b.dtb`) defines `cooling_fan` and `fan_temp0…3` with `_hyst` and `_speed`. The `cooling_fan` device is disabled in the base tree, so `dtparam=cooling_fan=on` switches it on explicitly.

### 4.1 Before (Pi, over SSH)

```bash
F=$(ls -d /sys/devices/platform/cooling_fan/hwmon/hwmon* 2>/dev/null); echo "${F:-no fan device}"
cat $F/pwm1 $F/fan1_input                     # current PWM (0-255) and RPM
cat /sys/class/thermal/thermal_zone0/temp     # CPU temperature in thousandths of °C
cd /sys/class/thermal/thermal_zone0 && for t in trip_point_*_temp; do n=${t%_temp}; echo "$n $(cat ${n}_type) $(cat $t)"; done; cd
```

With the default settings, `pwm1` is between 0 and 250 depending on the temperature. The switch-on points are `critical 110000` and `active 50000 / 60000 / 67500 / 75000`.

If it prints `no fan device`, check the fan cable on the FAN connector. Step 4.2 also switches the device on.

### 4.2 Edit `config.txt` (Pi)

The file is **`/boot/firmware/config.txt`**, at the root of the boot partition, **not** in `current/`.

- **It's never replaced:** the bootloader reads it before `os_prefix` applies, and kernel updates don't rewrite it.
- **Position matters:** settings for the base device tree must come **before the first `dtoverlay=` line** (Ubuntu's own `config.txt` says so). So the block goes right after `dtparam=watchdog=on`.

```bash
cp /boot/firmware/config.txt ~/config.txt.orig
grep -q '^dtparam=fan_temp0_speed=' /boot/firmware/config.txt || sudo sed -i '/^dtparam=watchdog=on$/a\
# Pi 5 fan: 100% (255) at every cooling level, from early boot\
dtparam=cooling_fan=on\
dtparam=fan_temp0=10000\
dtparam=fan_temp0_hyst=5000\
dtparam=fan_temp0_speed=255\
dtparam=fan_temp1_speed=255\
dtparam=fan_temp2_speed=255\
dtparam=fan_temp3_speed=255' /boot/firmware/config.txt
grep -n -E '^(dtparam|dtoverlay)=' /boot/firmware/config.txt
```

The last command must list, in this order: `audio`, `i2c_arm`, `spi`, `watchdog`, **the seven fan lines**, and only then `dtoverlay=vc4-kms-v3d`.

| Line | Default | New | Effect |
|---|---|---|---|
| `cooling_fan=on` | off in the base tree | on | Enables the fan device and the PWM output it uses |
| `fan_temp0=10000` | 50000 (50 °C) | 10 °C | The first level switches on almost immediately |
| `fan_temp0_hyst=5000` | 5000 | 5000 | The first level would switch off only below 5 °C |
| `fan_temp0_speed` … `fan_temp3_speed` | 75 / 125 / 175 / 250 | 255 | Every level means full speed, so the fan never slows down when it's hot |

**To undo:** `sudo cp ~/config.txt.orig /boot/firmware/config.txt`, then reboot.

### 4.3 Reboot and verify (Pi)

```bash
sudo reboot
```

Log in again with `ssh unpi@192.168.10.2`, then:

```bash
F=$(ls -d /sys/devices/platform/cooling_fan/hwmon/hwmon*)
cat $F/pwm1                                   # 255
cat $F/fan1_input                             # several thousand RPM, clearly more than in 4.1
cd /sys/class/thermal/thermal_zone0 && for t in trip_point_*_temp; do n=${t%_temp}; echo "$n $(cat ${n}_type) $(cat $t)"; done; cd
for d in /sys/class/thermal/cooling_device*; do [ "$(cat $d/type)" = pwm-fan ] && echo "$d: state $(cat $d/cur_state) of $(cat $d/max_state)"; done
```

Expected:

- `pwm1` = **255**.
- The first `active` switch-on point is **10000**.
- The `pwm-fan` cooling device is at state 1 or higher, out of 4. Every state now means 255.

**From power-on:** run `sudo poweroff`, wait for the red LED, unplug the power for 10 seconds and plug it back in. The fan should go to full speed within the first seconds of boot (you'll hear it) and stay there. Check `pwm1` again after logging in.

✅ **Part 4 done:** the fan runs at 255 at every boot.

---

## Final check (after one more cold boot, no monitor)

- [ ] `ping -c 3 192.168.10.2` and `ssh unpi@192.168.10.2` work from the PC, with `pi-lab-shared` and with `pi-direct`.
- [ ] The 3.8 block shows the two settings in `/proc/cmdline`, `HDMI-A-2: connected, enabled, modes: 1328x748`, and a `Type=wayland … Active=yes` session.
- [ ] An RDP client (Remmina on the PC, or Remote Desktop Connection on Windows) shows the desktop at 1328 × 748.
- [ ] `pwm1` = 255 and `fan1_input` is at full RPM.

---

## Before you capture this as the reusable image

- **Remote desktop identity:** every card flashed from the image shares the same TLS certificate and the same RDP login, `unpi` / `unpi`. That's acceptable on the direct cable. To give one Pi its own certificate, repeat 3.3 steps 1 and 2 on it; clients then warn once about the new certificate.
- **The login keyring is unencrypted (3.4 step 4).** Don't save personal passwords on the Pi before capturing the image, because they would be copied to every card in plain text.
- **Names:** the address (`192.168.10.2`) and the login (`unpi` / `unpi`) match the KIT Phantom X Pincher guides (`guias/Setup/conexionNoMachine.md`, `guiaSamba.md`). The hostname is `unpi` as well.

---

## Troubleshooting

| What you see | Why | What to do |
|---|---|---|
| Imager never asks for OS customisation | Normal for Ubuntu Desktop (`init_format: none`) | Set everything at the first boot (1.6–1.7) |
| The wizard rejects the password `unpi` | Its password-strength check | Use a temporary password, then run `echo 'unpi:unpi' \| sudo chpasswd` (1.7) |
| The Pi gets no internet over the cable | The PC is on `pi-direct` (`192.168.10.3`) or `pi-shared` (`192.168.77.1`), neither of which shares internet on this subnet | On the PC: `nmcli connection up pi-lab-shared`, then `ip -br addr show enp4s0` must show `192.168.10.1/24` |
| `nmcli connection up pi-lab-shared` fails | The cable has no link yet, or `dnsmasq-base` (the sharing helper) is missing | Check `cat /sys/class/net/enp4s0/carrier` = 1; `sudo apt install dnsmasq-base` |
| `REMOTE HOST IDENTIFICATION HAS CHANGED` | The old 24.04 lab card used the same address | On the PC: `ssh-keygen -R 192.168.10.2` |
| `ssh: … port 22: Connection refused` | `openssh-server` isn't installed | 2.1 |
| `/proc/cmdline` doesn't contain `video=` | The edit went to `/boot/firmware/cmdline.txt`, which is ignored, or to a `current/` that a pending `new/` replaced at boot | Redo 3.6; it edits `current/` and `new/` |
| `HDMI-A-2: connected, disabled` | Nobody is logged in, or GNOME didn't switch the output on | Check automatic login (3.4) and the `loginctl` output in 3.8, then reboot |
| `HDMI-A-2` lists `1024x768 848x480 …` instead of only `1328x748` | The kernel couldn't read the EDID file: it's missing, or the name in `cmdline.txt` has a typo | `ls -l /lib/firmware/edid/1328x748-15.bin` (128 bytes) and `sudo dmesg \| grep -i edid`. Redo 3.6 step 1, or fix the name in `current/cmdline.txt`, then reboot. |
| The RDP client can't connect after a reboot ("connection refused", or it times out) | The login keyring is locked, so Desktop Sharing can't read its login; or the service didn't start | Over SSH: `grdctl status`. `Failed to read credentials` means the keyring: see 3.4 step 4. Otherwise read the log: `journalctl --user -u gnome-remote-desktop -b --no-pager \| tail -n 30` |
| The log says "RDP TLS certificate and key not yet configured properly" | Step 1 or the `set-tls-*` commands of 3.3 were skipped | Repeat 3.3 steps 1 and 2 |
| The client connects, then disconnects at once | The screen locked (locking ends Desktop Sharing), or there's no active output | Check the `lock-enabled false` setting (3.4) and the 3.8 output |
| The client shows a black window | No active output to capture | Check 3.8. If it's fine and the window is still black, use a dummy HDMI plug on HDMI0 set to a small size (3.9). |
| The login is rejected | The RDP login (`grdctl rdp set-credentials`) is separate from the Linux account; or Windows sent your Windows account instead | Run `grdctl rdp set-credentials unpi unpi` again. In Windows, choose **More choices → Use a different account** and type `unpi`. |
| The client can watch but not click or type | View-only is still on, which is the default | `grdctl rdp disable-view-only` |
| Nothing listens on port 3389, and the log mentions the address being in use | *Remote Login* (the system service) is also on and holds 3389 | Turn off **Settings → System → Remote Desktop → Remote Login**, then reboot |
| A monitor shows "no signal" or a bad picture after 3.6 | It's on **HDMI1**, which now only offers 1328×748 at 15 Hz | Move it to **HDMI0**, next to the USB-C power. To give HDMI1 back to monitors, undo 3.6. |
| No `cooling_fan/hwmon` folder | The fan isn't detected or switched on | Check the fan cable; check that `dtparam=cooling_fan=on` is before the first `dtoverlay=` line (4.2) |
| `pwm1` isn't 255 after 4.3 | The fan lines are after a `dtoverlay=` line, or have a typo | Check the order with the `grep -n` in 4.2 |

---

## Sources

- [Raspberry Pi Imager OS list](https://downloads.raspberrypi.com/os_list_imagingutility_v4.json): the "Ubuntu Desktop 26.04.1 (64-bit)" entry, `init_format: none`
- [Ubuntu 26.04.1 release images](https://cdimage.ubuntu.com/releases/26.04.1/release/) and the [Desktop raspi image manifest](https://cdimage.ubuntu.com/releases/26.04.1/release/ubuntu-26.04.1-preinstalled-desktop-arm64+raspi.manifest)
- [A/B boot on Ubuntu for Raspberry Pi](https://canonical-ubuntu-hardware-support.readthedocs-hosted.com/boards/explanations/piboot-ab/): the `current/`, `new/` and `old/` folders, and `cmdline.txt` under them
- [Edit the Raspberry Pi boot configuration (Ubuntu)](https://documentation.ubuntu.com/hardware-support/boards/how-to/special_hardware/rpi-config-txt/)
- [Call for testing: A/B boot on Raspberry Pi](https://discourse.ubuntu.com/t/call-for-testing-a-b-boot-on-raspberry-pi/64173)
- [Linux kernel: EDID](https://docs.kernel.org/admin-guide/edid.html) (`drm.edid_firmware=` and `/lib/firmware/edid/`) and [modedb](https://docs.kernel.org/fb/modedb.html) (the `video=` setting and its `D` flag)
- [KIT remote-desktop guide, annex A.4](conexionEscritorioRemoto.md#a4-sin-monitor-conectado-pantalla-virtual-de-1328x748-a-15-fps): the 1328×748, 15 Hz virtual screen and why that size and rate were chosen
- [Raspberry Pi `config.txt` documentation](https://www.raspberrypi.com/documentation/computers/config_txt.html)
- [GNOME Remote Desktop 50.2 README](https://gitlab.gnome.org/GNOME/gnome-remote-desktop/-/blob/50.2/README.md): the three modes, `grdctl` setup, TLS certificate generation, and "locking the screen also closes the remote desktop connection"
- GNOME Remote Desktop 50.2 source: [`grd-settings.c`](https://gitlab.gnome.org/GNOME/gnome-remote-desktop/-/blob/50.2/src/grd-settings.c) (Desktop Sharing keeps its login in the keyring through libsecret), [`grd-ctl.c`](https://gitlab.gnome.org/GNOME/gnome-remote-desktop/-/blob/50.2/src/grd-ctl.c) (`grdctl` commands and `status` output), [`org.gnome.desktop.remote-desktop.gschema.xml.in`](https://gitlab.gnome.org/GNOME/gnome-remote-desktop/-/blob/50.2/src/org.gnome.desktop.remote-desktop.gschema.xml.in) (defaults: port 3389, view-only on, port negotiation on, `mirror-primary`), [`grd-rdp-sw-encoder-ca.c`](https://gitlab.gnome.org/GNOME/gnome-remote-desktop/-/blob/50.2/src/grd-rdp-sw-encoder-ca.c) (the software RFX Progressive encoder) and [`NEWS`](https://gitlab.gnome.org/GNOME/gnome-remote-desktop/-/blob/50.2/NEWS)
- [Raspberry Pi white paper: H.264 encoding performance on Raspberry Pi 5 series computers](https://pip-assets.raspberrypi.com/categories/685-app-notes-guides-whitepapers/documents/RP-010033-WP-1-H.264%20encoding%20performance%20on%20Raspberry%20Pi%205_series%20computers.pdf): no hardware H.264 encoder on the Pi 5
- [Remmina in Ubuntu 26.04](https://packages.ubuntu.com/resolute/remmina-plugin-rdp)
- [NoMachine 10: what's new](https://www.nomachine.com/what-s-new-in-nomachine): the reason NoMachine isn't used (no free edition for the Pi)
