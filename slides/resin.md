# Manage a Flock of Rasperry Pi's with Docker and Resin.io

#### Easy deployment and fleet management for IoT devices

note:
spannende weil ganz andere Schwierigkeiten auftreten als beim Web Deploy
web vs iot systeme (ram, disk, networking)
services die diese Probleme lösen wollen, eines davon ist resin.io
zuerst produkt, dann weg zu resin, dann resin

---
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/atalanda_intro_1.jpg" -->
# The product


note: 
* we want to use the display windows of merchants as an additional way for engaging customers
zusätzlicher Weg Kunden ins Geschäft zu bringen / interessieren / auf atalanda zu bringen => ROPO/beidseitig
Datenanbindung an atalanda

>>>
<!-- .slide: data-background-image="/slides/images/atalanda_intro_1.jpg" -->

>>>
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/atalanda_slider_1.jpg" -->
* pure text
* image / text
* pure image
* quote
* special deals
* quizzes
* events

>>>
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/atalanda_slider_1.jpg" -->
## Different information layers
* city wide
* district
* neighbourhood
* vendor

>>>
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/atalanda_slider_1.jpg" -->
## Different owners of information
* city manager
* local newspaper
* advertising association
* atalanda
* vendor


note: more screenshots for events, quizzes, etc.

>>>
<!-- .slide: data-background-image="/slides/images/atalanda_slider_1.jpg" -->

---
<!-- .slide: data-background-color="#FFFFFF" -->
## Admin backend for Merchants

<img src="/slides/images/dashboard.png">

---
## The physical device: Proof of concept

* to see if there is demand for the product
* with internet capable TVs

note: configuration via tv remote, vendor instructions are easy but procedure is tedious


---
<!-- .slide: data-background-color="#FFFFFF" -->
# The problem(s)

>>>
<!-- .slide: data-background-color="#FFFFFF" -->
* Customizability
* Deployment
* System-Updates
* Visibility


note: eliminate one after the other

---
## First take
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/angel-sinigersky-512707-unsplash.jpg" -->

>>>
<img src="/slides/images/little_pi.png">
<img src="/slides/images/raspbian-logo.png" style="width:30%;height:30%">

note:
günstiges Device mit akzeptabler Performance / Debian basiertes OS
Photo by Angel Sinigersky on Unsplash

---
* Customizability ✔
* Deployment
* System-Updates
* Visibility

>>>
<!-- .slide: data-background-image="/slides/images/maintenance.png" -->
## Updating the device?

---
<!-- .slide: data-background-color="#FFFFFF" -->
## Automating updates and deployments

<img src="/slides/images/Ansible_logo.png", style="width:20%">

note:
ansible => cm tool, yml files, einfach, von uns auch für andere Infrastruktur verwendet
        => verwendbar auch für deployment

>>>
<!-- .slide: data-background-color="#FFFFFF" -->
```yaml
---
- hosts: all
  remote_user: pi
  become_user: root
  become_method: sudo

  tasks:
    - name: Install basic packages
      become: yes
      package: name={{ item }} state=present
      with_items: [ntpdate, ..., ruby]
    ... (some hundred lines)
    - name: set hdmi mode to normal
      become: true
      lineinfile: path=/boot/config.txt regexp="#?hdmi_drive=" line="hdmi_drive=2"
    ... (some more lines)
    - name: add wlan driver with dkms
      command: chdir=/usr/src/8812au-4.2.2 {{ item }}
      with_items:
      - dkms add -m 8812au -v 4.2.2
      - dkms build -m 8812au -v 4.2.2
      - dkms install -m 8812au -v 4.2.2
```

---
* Customizability ✔
* Deployment ✔
* System-Updates (partially ✔)
* Visibility

>>>
## Some new problems

* Configuration Drift
* and because of having the basic ability to deploy and update there are additional issues<!-- .element: class="fragment" -->
  * network accessibility
  * unreliable network

note:
* Configuration management tools can only spot drift in areas that we have  defined that the tools control. Configuration drift that occurs outside those areas doesn't get fixed.
* ansible pull (gut weil unabhängig von accessiblity/private network, schlecht weil weniger Einfluss und mehr Software am Pi)
* vpn

---
<!-- .slide: data-background-color="#FFFFFF" -->
## Approach immutable infrastructure

<img src="/slides/images/moby.png" style="width:50%">

* Minimize configuration drift with shifting as much work as possible to Docker

---
<!-- .slide: data-background-color="#FFFFFF" -->
<img src="/slides/images/Resin-io-logo.png">

>>>
<!-- .slide: data-background-color="#FFFFFF" -->

* Docker based, ready made base images
* everything is in container(s)<!-- .element: class="fragment" -->
* We do nothing in the host system (no conf drift!)<!-- .element: class="fragment" -->
* Host and Kernel Updates<!-- .element: class="fragment" -->
* Development Workflow<!-- .element: class="fragment" -->
* Visibility (Dashboard)<!-- .element: class="fragment" -->
* container deltas<!-- .element: class="fragment" -->

note:
* OS-Optimierung für möglichst wenige Schreibvorgänge auf SD-Card, Toolkit für einfaches Setup neuer vorkonfigurierter Geräte

>>>
<img src="/slides/images/resin_references.png" style="width: 1400px">

---
<!-- .slide: data-background-color="#FFFFFF" -->
<img src="/slides/images/architecture.png" style="width: 50%">

note:
* supervisor koordinert container deployment

---
## Docker => Moby => Balena

<img src="/slides/images/balena.svg">

note:
* https://www.balena.io/

>>>
* from https://resin.io/blog/announcing-balena-a-moby-based-container-engine-for-iot/
  * 10-70 times more bandwidth efficient
  * atomic and durable image pulling (power & network failures)<!-- .element: class="fragment" -->
  * conservative about how much it writes to the filesystem (not writing compressed layers, on the fly extraction)<!-- .element: class="fragment" -->
  * low-memory<!-- .element: class="fragment" -->
  * build-time volumes<!-- .element: class="fragment" -->
  * small binary (3.5x smaller)<!-- .element: class="fragment" -->

note:
- Docker Swarm, support for plugins, cloud logging drivers, overlay networking drivers, and stores that are not backed by boltdb, such as etcd, consul, zookeeper, etc

https://resin.io/blog/announcing-balena-a-moby-based-container-engine-for-iot/#technical_comparison
TODO: how does balena deltas relate to the delta feature RESIN_SUPERVISOR_DELTA (https://docs.resin.io/learn/deploy/delta/)

>>>

<img src="/slides/images/Balena-Blogpost.jpg">



---
<!-- .slide: data-background-color="#FFFFFF" -->
## Device Support

* https://docs.resin.io/reference/hardware/devices/
* https://docs.resin.io/reference/base-images/resin-base-images/

<img src="/slides/images/resin_supported.png" class="background">

---
<!-- .slide: data-background-color="#FFFFFF" -->
## Deployment

<img src="/slides/images/how_deploy_works.jpg" style="width: 100%">

note:
heroku style
build grid

---
* Customizability ✔
* Deployment ✔
* System-Updates ✔
* Visibility ✔
* Configuration Drift ✔
* network accessibility ✔
* unreliable network ✔

---
<!-- .slide: data-background-image="/slides/images/20171110_150023.jpg" -->

>>>
<!-- .slide: data-background-image="/slides/images/resin-deploy-full.png" -->

---
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/3055-06.jpg" -->
## Show me the code!

note:
* Resin-Dashboard
* neue App
* download iso und flash (vorbereitet, resin local flash)
* unwrap raspberry

>>>
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/3055-06.jpg" -->
```bash
npm i -g resin-cli
resin version
  7.3.1
```


>>>
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/3055-06.jpg" -->
```
resin help --verbose
  Primary commands:                                    
    app <name>                          list a single application
    apps                                list all applications
    build [source]                      Build a single image or a multicontainer project locally
    config generate                     generate a config.json file                                     
    config inject <file>                inject a device configuration file                              
    deploy <appName> [image]            Deploy a single image or a multicontainer project to a resin.io application
    device <uuid>                       list a single device
    device move <uuid>                  move a device to another application                            
    device reboot <uuid>                restart a device                                                
    device register <application>       register a device                                               
    device rename <uuid> [newName]      rename a resin device                                           
    device shutdown <uuid>              shutdown a device                                               
    devices                             list all devices
    local configure <target>            (Re)configure a resinOS drive or image                          
    local flash <image>                 Flash an image to a drive                                       
    local logs [deviceIp]               Get or attach to logs of a running container on a resinOS device
    local push [deviceIp]               Push your changes to a container on local resinOS device
    local scan                          Scan for resinOS devices in your local network
    local ssh [deviceIp]                Get a shell into a resinOS device                               
    login                               login to resin.io
    logs <uuid>                         show device logs
    preload <image>                     (beta) preload an app on a disk image (or Edison zip archive)
    ssh [uuid]                          (beta) get a shell into the running app container of a device
    sync [uuid]                         (beta) sync your changes to a device
```

>>>
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/3055-06.jpg" -->

## Flash the Image to the SD Card

```bash
resin local flash resin-webdev-2.12.5+rev2-v7.1.18.img
? Select drive /dev/mmcblk0 (15.5 GB) - SL16G
? This will erase the selected drive. Are you sure? Yes
Flashing [========================] 100% eta 0s
Validating [========================] 100% eta 0s
```

>>>
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/3055-06.jpg" -->

## Preconfigure new Device

```bash
resin device register APP_NAME
resin device rename UUID
resin config generate --device UUID --output FILE
resin config inject FILE --type raspberrypi3
```


note:
* Online nehmen / Device in Web-Interface zeigen

---
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/3055-06.jpg" -->

```bash
git push resin master

[Info]  Starting build for webdev, user it_team                           
[Info]  Dashboard link: https://dashboard.resin.io/apps/1031903/devices   
[Info]  Building on arm03                                                 
[Info]  Pulling previous images for caching purposes...             
[Success]  Successfully pulled cache images                                 
[main]  Step 1/11 : FROM resin/raspberrypi3-debian
[main]   ---> b6da9ef4a1bd
[main]  Step 2/11 : RUN apt-get update && apt-get install -y                 cec-utils                 chromium-browser                 curl                 feh                 git                 libgl1-mesa-dri
                 matchbox-window-manager                 python-dbus                 vim                 wget                 x11-xserver-utils                 xinit                 xserver-xorg-core
    xserver-xorg-video-fbdev                 xterm      && rm -rf /var/lib/apt/lists/*
[main]   ---> Running in b83329abfe28
....
```

note:
* app erstellen und pushen (dann mit public device url herzeigen)
* dev mode erwähnen


>>>
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/3055-06.jpg" -->

```
[Success]  Release successfully created!                                
[Info]  ┌─────────┬────────────┬───────────────────────┐      
[Info]  │ Service │ Image Size │ Build Time            │                                           
[Info]  ├─────────┼────────────┼───────────────────────┤                              
[Info]  │ main    │ 627.85 MB  │ 2 minutes, 43 seconds │
[Info]  └─────────┴────────────┴───────────────────────┘
[Info]  Build finished in 4 minutes, 13 seconds                   
                            \             
                             \                            
                              \\      
                               \\               
                                >\/7                      
                            _.-(6'  \
                           (=___._/` \
                                )  \ |
                               /   / |
                              /    > /
                             j    < _\
                         _.-' :      ``.
                         \ r=._\        `.
                        <`\\_  \         .`-.
                         \ r-7  `-. ._  ' .  `\
                          \`,      `-.`7  7)   )
                           \/         \|  \'  / `-._
                                      ||    .'
                                       \\  (
                                        >\  >
                                    ,.-' >.'
                                   <.'_.''
                                     <'

To git.resin.io:it_team/webdev.git
   b829dda..51162ec  master -> master
```

---
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/3055-06.jpg" -->

```bash
resin ssh 537... --host
root@537:~# balena ps
CONTAINER ID        IMAGE                                     CREATED
9d010ac4757e        registry2.resin.io/shopscreenesa/f31...
5fdabad3c75b        resin/armv7hf-supervisor:v7.1.18
```

>>>
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/3055-06.jpg" -->

```bash
resin ssh c94419882ac82bbaa9f6a8d2a4bd62bc 
Connecting to: c94419882ac82bbaa9f6a8d2a4bd62bc
root@c944198:/#
```

---
<!-- .slide: data-background-image="/slides/images/3055-06.jpg" -->

note:
ev. noch erwähnen: local mode (ssh / docker socket), multicontainer, dbus, privileged mode (gpio / i2c), wifi connect, cec, staged releases

---
<!-- .slide: data-state="dimbg" data-background-image="/slides/images/foobar.jpg" -->

# Thanks a lot!

## Next Up

* Q&A
* Announcements
* opening of the fooBar!

note:

MÖGLICHE WEITERE THEMEN:
https://resin.io/blog/filters-and-tags-a-fleet-management-primer-for-resin-io/
https://resin.io/blog/multicontainer-on-resin-io-is-here/
WLAN setup (USB, WifiConnect)
dbus (network einstellen herzeigen)
AWS IoT, Azure, IBM, Artik Cloud
dev mode: ssh access, docker socket

SOME CLI COMMANDS:
resin ssh UUID
resin devices -a webdev
resin device rename UUID
resin os versions raspberrypi3
resin os download raspberrypi3 -o out.img
resin logs UUID
resin preload  os.img --app xxx --commit xxx --splash-image logo.png
resin apps

echo "on 0" | cec-client -s -o webdev -d 1
echo "as" | cec-client -s -o webdev -d 1

https://github.com/resin-io-playground/staged-releases
https://docs.google.com/presentation/d/1sxlxuslPXxJPW390MmmJTfBcvP9r9PXc4UX-Miv73o0/edit?ts=5acfa110#slide=id.g3816b9d468_3_129
curl -XPOST https://c94419882ac82bbaa9f6a8d2a4bd62bc.resindevice.io/play
