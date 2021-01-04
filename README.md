# Systems

Guides / documentation for CBL hardware


## cbl1

`cbl1` is currently the only CBL workstation. It is a high-end machine
that has been configured for batch-processing long-running simulation
jobs.


### Hardware

High-end (in 2020) consumer-grade hardware. Specs: Ryzen9 3950X
(16-core processor), 32 GB RAM (16 GB swap), GeForce2080 RTX, Samsung
NVMe 1 TB SSD hard drive.

More details can be found from a terminal (e.g. SSH):

```
cat /proc/cpuinfo  | head
processor	: 0
vendor_id	: AuthenticAMD
cpu family	: 23
model		: 113
model name	: AMD Ryzen 9 3950X 16-Core Processor
stepping	: 0
microcode	: 0x8701013
cpu MHz		: 2195.876
cache size	: 512 KB
physical id	: 0

lspci | grep -i vga
0b:00.0 VGA compatible controller: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti Rev. A] (rev a1)

cat /proc/meminfo
MemTotal:       32874652 kB
MemFree:        12174576 kB
MemAvailable:   22365668 kB
Buffers:         1182340 kB
Cached:          8620264 kB
SwapCached:       479168 kB
Active:         14064448 kB
Inactive:        4162512 kB
Active(anon):    7577676 kB
Inactive(anon):  1010192 kB
Active(file):    6486772 kB
Inactive(file):  3152320 kB
Unevictable:         208 kB
Mlocked:             208 kB
SwapTotal:      16383996 kB

lspci | grep -i ssd
01:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983

df -h | grep '/$'
/dev/nvme0n1p2  938G  164G  727G  19% /
```

### Software

Barebones Ubuntu 20.04 install. `python` is assumed to be the
scripting language that most people will use. For biomech-specific
software, `opensim`, `scone`, and `opensim-moco` are currently
installed.

CBL-specific scripts have also been put onto the system. These are
always prefixed with `cbl_` and are generally ad-hoc quality-of-life
scripts (e.g. reset VNC, reset password).

`cbl1`'s firewall has been configured to only allow inbound
connections to the SSH server. If you need to host anything (e.g. a
webserver, VNC server, etc.) clients will only be able to access those
services by tunneling via an SSH connection.

| command | result (tidied) |
| ------- | --------------- |
| `lsb_release -a` | Ubuntu 20.04 |
| `python --version` | 3.8.x |
| `python2` | (not available) |
| `sconecmd --version` | scone 1.6.0.3183 ALPHA |
| `opensim-cmd --version` | OpenSim 4.2 (custom build from `master` /w support for ScapulothoracicJoint) |
| `conda --version` | conda 4.8.3 |
| `opensim-moco --version` | 0.5.0-2020-09-09-f5dc979c |
| `cbl_*` | not versioned: these are typically just bash/python scripts that do something useful |


### Connecting to `cbl1`

### Quickstart (Windows)

This assumes you already went through the walthrough and just want a
tl;dr.

- Open windows terminal
- Run `ssh -L 59000:localhost:<your vpn port> username@cbl1`
- Open VNC viewer on your Windows machine
- Use your VNC password to connect
  - You can reset your VNC password from an SSH terminal with `cbl_vnc-passwd`

### Walkthrough

This walkthrough essentially sets up:

- Making windows forward `cbl1` to the server's IP address
- Using `ssh` to connect a terminal to `cbl1`
- Launching a personal VNC server on `cbl1`
- Opening an SSH tunnel to the VNC server
- Using VNC Viewer to connect to the tunneled VNC server

| details |
| ------- |
| Server address (IPv4) == 145.94.60.179 |
| ECDSA MD5:9c:0a:63:57:71:c6:c6:77:5e:66:d3:8b:d0:ac:9d:8f |
| ECDSA SHA256:/3U2CknH+ndkOEuwLnEjRH3lUyy6xWtXPQbmj9lojq4 |
| **Access via SSH only**: you *must* use an SSH tunnel to access anything |


#### 1. (optional) Setup the `cbl1` Hostname

This step sets up the hostname `cbl1` to point to the server's IP
address (above) on your machine. It's mostly a quality-of-life step
that enables referring to the machine as `cbl1`, rather than some
harder-to-remember (and potentially changing) IP address.

- Open Notepad as an administrator

  - Find it Notepad in the start bar
  - Right click the icon, click "Run as administrator"

- In Notepad, open C:\Windows\system32\drivers\etc\hosts

- Add this line to the bottom of `hosts`:

```
145.94.60.179 cbl1
```

- All connections from your computer to `cbl1` (e.g. in a browser,
  SSH, software) will now connect to 145.94.60.179


#### 2. Connect to the server with SSH

> Note: this assumes you have been given SSH credentials by (e.g.) an
> admin.

- Open a Windows PowerShell window

  - SHIFT + right-click on your Windows desktop

  - Click "Open PowerShell Window Here"

  - SSH into the server with your login credentials by typing `ssh
    username@cbl1`

  - The server should prompt that you're using an unknown connection
    with some ECDSA fingerprint. This fingerprint should match the
    server's (table, above)

  - If the ECDSA fingerprint looks ok, enter your password

  - You should now have an SSH connection to `cbl1`. If you only need
    terminal access, you can stop reading the walkthrough.


#### 3. Boot a VNC server on `cbl1`

A VNC client is used to "remote desktop" connect to a VNC server on
`cbl1`. Each user runs their own VNC server on `cbl1`.

- In your SSH session (established above) run `cbl_vnc-start`. If
  necessary, type a new VNC password. You can change the password
  later with `cbl_vnc-passwd`.

- The `cbl_vnc-start` command should, before completing, print some
  help documentation. From this, **you need to rememeber**:

  - The VNC password you set
  - The VNC server's port, printed in the terminal (e.g. `6901`)

- Although the VNC server is running, you cannot connect to the VNC
  server from your desktop yet. `cbl1` only allows external
  connections via SSH tunnels (the next step).

#### 4. Setup SSH tunnel to the VNC server

`cbl1` only allows external connections via SSH. All services
(e.g. VNC) must connect via an SSH tunnel. This step sets up a tunnel
to the VNC server you booted in the previous step.

- Exit the SSH connection to `cbl1` that was used in previous steps by
  either:

  - Typing `exit` in the terminal (to `exit` the SSH session)
  - Pressing CTRL+D to interrupt+exit the terminal
  - Closing the Powershell window

- Open a new SSH session to `cbl1` with tunelling enabled by running
  the command below (note: `<your VNC server port>` is printed during
  the previous step):

```bash
ssh -L 59000:localhost:<your VNC server port> username@cbl1

# example: ssh -L 59000:localhost:6901 adam@cbl1
```

- You should now have a new SSH session open that looks exactly like
  the previous one. It is essentially the same, but now *also* tunnels
  any connections to `localhost:59000` on your machine to `<your VNC
  server port>` on `cbl1`.


#### 5. Install + setup VNC client on your machine

- Download a VNC client. I used VNC Viewer, from [here](https://www.realvnc.com/en/connect/download/viewer/)

- Other VNC clients (e.g. Remmina) also seem to work, but I only
  tested with VNC Viewer on Windows

#### 6. Use the client to connect to the VNC server via the tunnel

- In your VNC client, connect to `localhost:59000`, which is the
  local-side of your (opened in the previous step) SSH tunnel to your
  VNC server (also a previous step) on `cbl1`

- If you receive warnings about connection security (e.g. encyption)
  you can ignore these. The connection between the VNC client and
  `localhost:59000` is insecure, but local (to your machine). The
  actual data is transported via the (encrypted) SSH tunnel, which is
  secure - the VNC client doesn't "know" that's whats happening.

- Use your VNC password when connecting to the machine with the VNC
  client.

  - Note: you can change your password in the SSH session with
    `cbl_vnc-passwd`

  - Note #2: VNC passwords are usually max. 6 chars, so your usual
    password might've been truncated

- Once connected, you should see a (somewhat rough-looking) Linux
  desktop
