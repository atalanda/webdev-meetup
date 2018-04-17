# Pragmatic Continuous Delivery for the Web with Kanban and Concourse

---

# Kanban
![kanban](slides/images/kanban/kanban.png)

>>>

## Where does it come from?

Systematic method for waste minimization within a value stream without sacrificing productivity

Note:
- Lean Manufacturing / Lean Production
- Waste definition: Ueberbelastung des Teams bzw Ungleichheit der Workloads
- Materialien bewegen, die zur Zeit nicht bewegt werden muessen
- Materialien, die nicht verwendet werden (Ueberproduktion)
- Warten auf den naechsten Arbeitsschritt, Unterbrechung der Produktion waehrend eines Schichtwechsels
- Produktion vor Gebrauch
-

>>>

## Basic Kanban
  - Kanban (= japanese term for signboard)

  - Method to uncover waste within a value stream

  - Buckets with a sign and filled with material

>>>

## Kanban rules
  - An incoming request provides the quantity of the items needed

  - Production and Transport of an item is always associated with a request

  - The request is always physically attached to an item

  - A finished item is always flawless

  - The number of pending requests is limited

>>>

<!-- .slide: data-background-image="/slides/images/kanban/tps.jpg" -->

Note:
We have a bucket with material we use
we use all the material till the bucket is empty
the (Kanban) card attached to the bucket is moved to the source of supply
the source fills the bucket with flawless material and sends it back to the supplier

>>>

## Most basic example in software development

![Kanban Softwaredevelopment](https://conceptboard.com/wp-content/uploads/Conceptboard-Kanban-Board-opt.png)

>>>

## Kanban in atalanda

![Atalanda Jira](slides/images/kanban/ata_jira.png)

>>>

## Kanban vs. Pipeline
![Atalanda Pipeline Kanban](slides/images_concourse/ata_pipeline_kanban.png)

>>>

## Literature

- Toyota Production System: Beyond Large-scale Production - Taiichi Ohno (1988);
- Schlanker Materialfluss mit Lean Production - Philipp Dickmann (2009);
- Kanban: Successful Evolutionary Change for Your Technology Business - David Anderson (2010)

---

<img src="/slides/images_concourse/concourse-logo.png">
<h4 style="color: white">an open-source continuous thing-doer</h4>

---

# Continous Integration

> Civilization advances by extending the number of important operations we can perform without thinking.

> **Alfred North Whitehead**

note: Es sollte alles automatisiert werden, was automatisiert werden kann.

>>>

* automate everything
* **If it hurts, do it more often**, Martin Fowler

>>>

* docker builds
* linting (ruby, scss, js)
* security checks (brakeman)
* Common Vulnerabilities and Exposures (CVE)
* tests
* deployments
* git merges
* atomic deploys: secrets within builds

---
<!-- .slide: data-background-image="/slides/images_concourse/atalanda-pipeline.png" -->

>>>

# Resources

* git, slack-notification, S3, docker, time...
* based on docker images
* language agnostic

```yaml
resources:
- name: atalanda-master
  type: git
  source:
    uri: git@bitbucket.org:atalanda/atalanda.git
    branch: master
    private_key: {{private-key}}
- name: slack-alert
  type: slack-notification
  source:
    url: {{slack-webhook-uri}}
```

>>>

# Jobs & Plans

```yaml
jobs:
- name: hello-world
  plan:
  - task: say-hello
    config:
      platform: linux
      image_resource:
        type: docker-image
        source: {repository: alpine}
      run:
        path: echo
        args: ["Hello, world!"]
```

>>>

# Dependencies

* `passed: [tests]` only run job when tests are passed
* Auto-trigger for new versions in a resource
* `serial: true` don't run in parallel
* Callbacks `on_success`, `on_failure`

notes: ich sag concourse nicht wie die Pipeline aussieht, sondern definiere nur meine Abhängigkeiten (resources) und concourse baut mir die Pipeline (ergibt das Sinn?)

---

# Demo

<a href="http://internal-ci/teams/main/pipelines/atalanda" target="_blank">
  http://internal-ci/teams/main/pipelines/atalanda
</a>

---

# Why Concourse?

* Configuration As Code: Config & CI under source control
* docker based
* language agnostic
* open source

---

# Other Projects

<div style="display: flex; align-items: center; justify-content: space-around"><img src="/slides/images_concourse/circle-ci.svg" width="300px">
  <img src="/slides/images_concourse/codeship.png" width="300px">
  <img src="/slides/images_concourse/travisci.png" width="300px">
  <img src="/slides/images_concourse/teamcity.jpg" height="150px">
</div>

notes: wir sind fan von hostet solutions, sodass wir uns auf unsere core competences konzentrieren können

---

```yaml
resources:
- name: Concourse CI
  type: open-source-project
  source: https://concourse-ci.org

- name: Continuous Delivery
  type: book
  source: https://martinfowler.com/books/continuousDelivery.html

- name: The Pragmatic Programmer
  type: book
  source: https://pragprog.com/book/tpp/the-pragmatic-programmer

- name: Pragmatic Thinking and Learning
  type: book
  source: https://pragprog.com/book/ahptl/pragmatic-thinking-and-learning


jobs:
- name: learn
  plan:
  - put: brain
```
