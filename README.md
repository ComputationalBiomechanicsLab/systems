# Systems

Guides / documentation for CBL hardware

## `cbl1` (or `cbl1.3me.tudelft.nl`)

`cbl1` is currently the only CBL workstation. It is a high-end machine
that has been configured for batch-processing long-running simulation
jobs.

This guide uses `cbl1` (the short name) and `cbl1.3me.tudelft.nl` (the
internet name) interchangeably: they mean the same thing.

| server detail | value | comment |
| ------------- | ----- | ------- |
| Short name | `cbl1` | Mostly only used in this guide |
| Internet Domain Name | `cbl1.3me.tudelft.nl` | Globally true |
| Internet IP address | `130.161.252.14` | May change over time |
| Local IP address | `172.19.126.14` | May change over time |
| Firewall | SSH (TCP/22) only | |
| SSH ECDSA (MD5) | `MD5:9c:0a:63:57:71:c6:c6:77:5e:66:d3:8b:d0:ac:9d:8f` | |
| SSH ECDSA (SHA256) | `SHA256:/3U2CknH+ndkOEuwLnEjRH3lUyy6xWtXPQbmj9lojq4` | |


### Quickstart (Windows)

This assumes you already went through the extensive walthrough below and 
just want to reconnect:

- Open windows terminal
- Run `ssh -L 59000:localhost:<your vpn port> username@cbl1.3me.tudelft.nl`
- Type in your password (**remember, you can't see it being typed in**)
- Open VNC viewer on your Windows machine
- Use your VNC password to connect
  - You can reset your VNC password from an SSH terminal with `cbl_vnc-passwd`


### Walkthrough (Windows)

This walkthrough essentially sets up:

- Using `ssh` to connect a terminal to `cbl1`
- Launching a personal VNC server on `cbl1`
- Opening an SSH tunnel to the VNC server
- Using VNC Viewer to connect to the tunneled VNC server


#### 1. Connect to the server with SSH

> **Note**: this assumes you have been given SSH credentials by (e.g.) an
> admin. Your credentials will be a **username** + **password** combo. This combo
> differs from your general TU Delft credentials. Once you have logged into
> `cbl1.3me.tudelft.nl`, you can change your password in the terminal with `passwd`

All connections to `cbl1.3me.tudelft.nl` must go via an SSH tunnel. SSH is a
standard method for creating a secure connection to a remote machine. By "SSHing"
into a server, you gain access to a terminal that can be used to run things on
the server (via a command prompt).

You always need an SSH connection to connect to `cbl1` - even 
if you actually want a GUI (e.g. a remote desktop). Later steps in this guide
describe setting up a remote desktop *via* the SSH connection.

- Open a Windows PowerShell window

  - SHIFT + right-click on your Windows desktop

  - Click "Open PowerShell Window Here"

- SSH into the server with your login credentials by typing `ssh
  username@cbl1.3me.tudelft.nl`

- Enter your password. **Note: you can't see any changes while typing
  the password in (it's a security feature)**

- You should now have an SSH connection to `cbl1.3me.tudelft.nl`. It will print a
  welcome message, etc.

- If you only need  terminal access, you can stop reading this
  walkthrough - you're done :smile:


#### 2. Boot a VNC server on `cbl1`

A VNC client is used to "remote desktop" connect to a VNC server on
`cbl1.3me.tudelft.nl`. Each user runs their own VNC server on `cbl1`.

- In your SSH session (established above) run `cbl_vnc-start`. If
  necessary, type a new VNC password. You can change the password
  later with `cbl_vnc-passwd`.

- The `cbl_vnc-start` command should, before completing, print some
  help documentation. From this, **you need to rememeber**:

  - The VNC password you set
    - **this is different from your SSH password**
    - It is limited to 6 characters
    - It does not need to be secure, because your VNC connection
      always goes via your (secure) SSH connection
  - The VNC server's port, printed in the terminal (e.g. `6901`)
    - The `cbl_vnc-start` should print this in a large banner, or some
      explanation text

- A VNC server is now running on `cbl1`. However, it is hidden behind
  `cbl1`'s firewall, which only permits SSH connections.


#### 3. Setup a SSH tunnel to the VNC server

`cbl1` only allows external connections via SSH. All services
(e.g. VNC) must connect via an SSH tunnel. This step sets up a tunnel
through the SSH connection through to the VNC server you booted in the
previous step.

- Exit the SSH connection to `cbl1` that was used in previous steps by
  either:

  - Typing `exit` in the terminal (to `exit` the SSH session)
  - Pressing CTRL+D to interrupt+exit the terminal
  - Closing the Powershell window

- Open a new SSH session to `cbl1` with tunelling enabled by running
  the command below (note: `<your VNC server port>` is printed during
  the previous step):

```bash
ssh -L 59000:localhost:<your VNC server port> username@cbl1.3me.tudelft.nl

# example: ssh -L 59000:localhost:6901 adam@cbl1.3me.tudelft.nl
```

- You should now have a new SSH session open that looks exactly like
  the previous one. It is essentially the same, but now *also* tunnels
  any connections to `localhost:59000` on your machine to `<your VNC
  server port>` on `cbl1`.


#### 4. Install + setup VNC client on your machine

- Download a VNC client. I used VNC Viewer, from [here](https://www.realvnc.com/en/connect/download/viewer/)

- Other VNC clients (e.g. Remmina) also seem to work, but I only
  tested with VNC Viewer on Windows


#### 5. Use the client to connect to the VNC server via the tunnel

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
  
You're done! woohoo! :smile:


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
