> # FORK NOTICE
> I have forked the steam-headless docker to adapt it to my crappy bleeding edge AMD GPU. Here are most of the hacks I use to reduce the amount of crashes and GPU-resets I get with my own hardware :
> 
> **AMD Ryzen 7 8845HS with Radeon 780M** (gfx1103_r1) (wouldn't recommend)
>
> If you are still experiencing GPU crashes, upgrade your host kernel. My GPU was completely broken and unusable until kernel 6.15.
> 
> If you are using non-AMD hardware, you should stick with [the original project](https://github.com/Steam-Headless/docker-steam-headless). This fork breaks non-amd gpu support and might break features I don't use.
>
> It is expected that you edit `/etc/default/grub` to include `amdgpu.virtual_display=0000:c6:00.0,1` in `GRUB_CMDLINE_LINUX_DEFAULT` (after replacing the PCI address with your own). Otherwise, you will get a black screen.
>
> Once sunshine is up and running, you might want to go to its settings and force it to use the VA-API encoder. Otherwise, the stream will be so laggy it's unplayable.
> 
> <details>
> <summary>Here is my docker-compose for reference</summary>
>
> ```yaml
> services:
>   steam-headless:
>     image: mubelotix/docker-steam-headless-amdgpu:latest
>     restart: unless-stopped
>     shm_size: 2G
>     ipc: host # Could also be set to 'shareable'
>     ulimits:
>       nofile:
>         soft: 1024
>         hard: 524288
>     cap_add:
>       - NET_ADMIN
>       - SYS_ADMIN
>       - SYS_NICE
>     security_opt:
>       - seccomp:unconfined
>       - apparmor:unconfined
> 
>     # This is required (see https://github.com/Steam-Headless/docker-steam-headless/issues/157)
>     network_mode: host
>     # ports:
>     #   - "{{ sunshine_inner_port }}:47990"
>     #   - "47984:47984/tcp"
>     #   - "47989:47989/tcp"
>     #   - "48010:48010/tcp"
>     #   - "47998:47998/udp"
>     #   - "47999:47999/udp"
>     #   - "48000:48000/udp"
>     #   - "48002:48002/udp"
>     #   - "48010:48010/udp"
> 
>     environment:
>       # System
>       - TZ=Europe/Paris
>       - USER_LOCALES=en_US.UTF-8 UTF-8
>       - SHM_SIZE=2G
>       # User
>       - PUID={{ user_id }}
>       - PGID={{ group_id }}
>       - UMASK=000
>       - USER_PASSWORD={{ steam_user_password }}
>       # Mode
>       - MODE=primary
>       # Web UI
>       - WEB_UI_MODE=none
>       # Steam
>       - ENABLE_STEAM=true
>       - STEAM_ARGS=-silent
>       # Sunshine
>       - ENABLE_SUNSHINE=true
>       - SUNSHINE_USER=mubelotix
>       - SUNSHINE_PASS={{ sunshine_password }}
>       # Xorg
>       - ENABLE_EVDEV_INPUTS=true
>       - FORCE_X11_DUMMY_CONFIG=true
> 
>     devices:
>       - /dev/fuse
>       - /dev/uinput
>       # Add AMD/Intel HW accelerated video encoding/decoding devices
>       # Obtained from `lspci | grep -iE "vga|3d|display"` and `ls /dev/dri/`
>       - /dev/dri/card0
>       - /dev/dri/renderD128
>       # - /dev/kfd  # Kernel Fusion Driver for AMD GPUs (for compute tasks like OpenCL, ROCm, etc.)
>       # - /dev/snd  # Sound card devices
>     
>     device_cgroup_rules:
>       - 'c 13:* rmw' # Ensure container access to devices 13:*
> 
>     volumes:
>       # The location of your home directory.
>       - /data/steam/home/:/home/default/:rw
> ```
> </details>

# Headless Steam Service

![](./images/banner.jpg)

Remote Game Streaming Server.

Play your games either in the browser with audio or via Steam Link or Moonlight. Play from another Steam Client with Steam Remote Play.

Easily deploy a Steam Docker instance in seconds.

## Features:
- Steam Client configured for running on Linux with Proton
- Moonlight compatible server for easy remote desktop streaming
- One click installation of EmeDeck, Heroic and Lutris
- Full video/audio noVNC web access to a Xfce4 Desktop
- NVIDIA, AMD and Intel GPU support
- Full controller support
- Support for Flatpak and Appimage installation
- Root access
- Based on Debian Bookworm

---
## Notes:

### ADDITIONAL SOFTWARE:
If you wish to install additional applications, you can generate a script inside the `~/init.d` directory ending with ".sh".
This will be executed on the container startup.

### STORAGE PATHS:
Everything that you wish to save in this container should be stored in the home directory or a docker container mount that you have specified. 
All files that are store outside your home directory are not persistent and will be wiped if there is an update of the container or you change something in the template.

### GAMES LIBRARY:
It is recommended that you mount your games library to `/mnt/games` and configure Steam to add that path.

### AUTO START APPLICATIONS:
In this container, Steam is configured to automatically start. If you wish to add additional services to automatically start, 
add them under **Applications > Settings > Session and Startup** in the WebUI.

### NETWORK MODE:
If you want to use the container as a Steam Remote Play (previously "In Home Streaming") host device you should create a custom network and assign this container it's own IP, if you don't do this the traffic will be routed through the internet since Steam thinks you are on a different network.

### USING HOST X SERVER:
If your host is already running X, you can just use that. To do this, be sure to configure:
  - DISPLAY=:0    
    **(Variable)** - *Configures the sceen to use the primary display. Set this to whatever your host is using*
  - MODE=secondary    
    **(Variable)** - *Configures the container to not start an X server of its own*
  - HOST_DBUS=true    
    **(Variable)** - *Optional - Configures the container to use the host dbus process*
  - /run/dbus:/run/dbus:ro    
    **(Mount)**  - *Optional - Configures the container to use the host dbus process*


---
## Installation:
- [Docker Compose](./docs/docker-compose.md)
- [Unraid](./docs/unraid.md)
- [Ubuntu Server](./docs/ubuntu-server.md)


---
## Running locally:

For a development environment, I have created a script in the devops directory.


---
## TODO:
- Remove SSH
- Require user to enter password for sudo
- Document how to run this container:
    - Other server OS
    - TrueNAS Scale 
