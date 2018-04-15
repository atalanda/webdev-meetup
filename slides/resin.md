# Manage a Flock of Rasperry Pi's with Docker and Resin.io

## Easy deployment and fleet management for IoT devices

note: about / the product / the problem
willkommen / es geht um deployment / nicht wie gewohnt auf server / auf schwächere systeme / weniger leistung, netzwerk, unzuverlässig, verteilt
im web gut gelöstes Problem / siehe voriger Talk / IoT schwieriger, aber viele neue Lösungen / wir stellen Lösung für Raspberries vor / eigenes Produkt siehe Wall


---

# The product

<img src="/slides/images/atalanda_slider_1.jpg" style="width:1000px">

note: 
* we want to use the display windows of merchants as an additional way for engaging customers
zusätzlicher Weg Kunden ins Geschäft zu bringen / interessieren / auf atalanda.com zu bringen => ROPO/beidseitig
muss einfach / günstig sein

>>>
* special deals
* quizzes
* ad network with city, other vendors and others
* events

<img src="/slides/images/atalanda_intro_1.jpg" class="background">

note: more screenshots for events, quizzes, etc.

>>>
## This is not the way we want to update an IoT device

<img src="/slides/images/maintenance.png">

note: erster Teil zum Vermeiden ist das Admin-Backend /zweiter Teil später

---
## Therefore the content can be updated with an admin backend

<img src="/slides/images/schaufenster_digital_screenshot.png">

---
## The physical device: Proof of concept

* to see if there is demand for the product
* with internet capable TVs

note: configuration via tv remote, vendor instructions are easy but procedure is tedious


---
# The problem(s)

>>>
* Customizability
* Deployment
* System-Updates
* Visibility


note: eliminate one after the other

---
## First take

<img src="/slides/images/angel-sinigersky-512707-unsplash.jpg" class="background">

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

---
## Updates and Deployments (still first take)

<img src="/slides/images/Ansible_logo.png", style="width:20%">

note:
ansible => cm tool, yml files, einfach, von uns auch für andere Infrastruktur verwendet
        => verwendbar auch für deployment

>>>
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
* and because of having the basic ability to deploy and update there are additional issues
  * network accessibility
  * unreliable network

note:
* Configuration management tools can only spot drift in areas that we have  defined that the tools control. Configuration drift that occurs outside those areas doesn't get fixed.
* ansible pull (gut weil unabhängig von accessiblity/private network, schlecht weil weniger Einfluss und mehr Software am Pi)
* vpn

---
## Approach immutable infrastructure

<img src="/slides/images/moby.png" style="width:50%">

* Minimize configuration drift with shifting as much work as possible to Docker

---
<img src="/slides/images/Resin-io-logo.png">

note:
* docker based / ready made base images / supervisor koordinert container deployment
* we do nothing in the host system (no conf drift!)
* everything is in container(s)
* host and kernel updates
* development workflow
* visibility (dashboard)
* OS-Optimierung für möglichst wenige Schreibvorgänge auf SD-Card, Toolkit für einfaches Setup neuer vorkonfigurierter Geräte
* container deltas

>>>
<img src="/slides/images/architecture.png" style="width: 50%">

---
## Docker => Moby => Balena

* https://www.balena.io/

<img src="/slides/images/balena.svg">

note:
* 10-70 times more bandwidth efficient
* atomic and durable image pulling (power & network failures)
* conservative about how much it writes to the filesystem (not writing compressed layers, on the fly extraction)
* low-memory
* build-time volumes
* small binary (3.5x smaller)
- Docker Swarm, support for plugins, cloud logging drivers, overlay networking drivers, and stores that are not backed by boltdb, such as etcd, consul, zookeeper, etc

https://resin.io/blog/announcing-balena-a-moby-based-container-engine-for-iot/#technical_comparison
TODO: how does balena deltas relate to the delta feature RESIN_SUPERVISOR_DELTA (https://docs.resin.io/learn/deploy/delta/)

>>>

<img src="/slides/images/Balena-Blogpost.jpg">



---
https://docs.resin.io/reference/hardware/devices/
<img src="/slides/images/resin_supported.png">

---
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
<img src="/slides/images/20171110_150023.jpg">


---
## Show me the code!

<img src="/slides/images/caspar-rubin-224229-unsplash.jpg" class="background">

note:
Photo by Caspar Rubin on Unsplash

* unwrap raspberry
* Resin-Dashboard
* neue App
* download iso und flash (vorbereitet, resin local flash)
* Online nehmen
* TODO: hello world app erstellen und pushen (ev. sinatra auf port 80, dann mit public device url herzeigen)
* Im Dashboard herzeigen, mit env variablen, und actions
* TODO: dev mode herzeigen (mit zweitem Pi)
* TODO: kleine app mit Mitmachmöglichkeit

---
# Thanks a lot!

## Next Up: Announcements and opening of the Foo-Bar!

note:
TODO (Stichworte extrahieren, überlegen wo in Präsentation einbauen):
* https://resin.io/blog/filters-and-tags-a-fleet-management-primer-for-resin-io/

Andere Inhalte, nicht sicher ob ausreichend Zeit in Präsentation vorhanden:
https://resin.io/blog/multicontainer-on-resin-io-is-here/


```bash
resin ssh 537... --host
root@537:~# balena ps
CONTAINER ID        IMAGE                                     CREATED
9d010ac4757e        registry2.resin.io/shopscreenesa/f31...
5fdabad3c75b        resin/armv7hf-supervisor:v7.1.18
```


MÖGLICHE WEITERE THEMEN:
WLAN setup (USB, WifiConnect)
Videoplayer Performance
dbus (network einstellen herzeigen)
AWS IoT, Azure, IBM, Artik Cloud
dev mode: ssh access, docker socket
* ev: Concourse-Pipeline für Resin-Deployments?

SOME CLI COMMANDS:
resin help --verbose
resin devices -a webdev
resin device rename UUID
resin ssh UUID
resin os versions raspberrypi3
resin os download raspberrypi3 -o out.img
resin logs UUID
resin preload  os.img --app xxx --commit xxx --splash-image logo.png
resin local flash os.img
resin apps
resin device register APP_NAME
resin device rename UUID
resin config generate --device UUID --output FILE
resin config inject FILE --type raspberrypi3
