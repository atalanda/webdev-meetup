# Manage a Flock of Rasperry Pi's with Docker and Resin.io

## Easy deployment and fleet management for IoT devices

<img src="/slides/images/atalanda_slider_1.jpg">

note: about / the product / the problem
willkommen / es geht um deployment / nicht wie gewohnt auf server / auf schwächere systeme / weniger leistung, netzwerk, unzuverlässig, verteilt
im web gut gelöstes Problem / siehe voriger Talk / IoT schwieriger, aber viele neue Lösungen / wir stellen Lösung für Raspberries vor / eigenes Produkt siehe Wall


---

# The product

<img src="/slides/images/atalanda_slider_1.jpg">

note: 
* we want to use the display windows of merchants as an additional way for engaging customers
zusätzlicher Weg Kunden ins Geschäft zu bringen / interessieren / auf atalanda.com zu bringen => ROPO/beidseitig
muss einfach / günstig sein

>>>
* special deals
* quizzes
* ad network with city, other vendors and others
* events
<img src="/slides/images/atalanda_intro_1.jpg">

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

<img src="/slides/images/angel-sinigersky-512707-unsplash.jpg">

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
* balena

>>>
<img src="/slides/images/architecture.png" style="width: 50%">

note:

>>>
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
## Show me the code!

<img src="/slides/images/caspar-rubin-224229-unsplash.jpg">

note:
Photo by Caspar Rubin on Unsplash

* Resin-Dashboard
* neue App
* config.json download
* auf vorbereitete SD-Karte schreiben (flash-Kommando nur erwähnen)
* Online nehmen
* hello world pushen
* kleine app mit Mitmachmöglichkeit


```bash
resin ssh 537... --host
root@537:~# docker ps
CONTAINER ID        IMAGE                                     CREATED
9d010ac4757e        registry2.resin.io/shopscreenesa/f31...   About an hour ago
7fc3a763df40        resin/armv7hf-supervisor:v6.3.6           5 months ago
```

---
# Thanks a lot!

## The Foo-Bar is open!

note:
WLAN setup (USB, WifiConnect)
Videoplayer Performance
flash deployment localdev dbus
AWS IoT, Azure, IBM, Artik Cloud
links, books, pointers
! Konfiguration / Zuordnung 
dev mode: ssh access, docker socketo
fleet management script herzeigen
resin api
