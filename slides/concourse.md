# Pragmatic Continuous Delivery for the Web with Kanban and Concourse

---

# Kanban

---

<img src="/slides/concourse_images/concourse-logo.png">
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
# Pipeline

<img src="/slides/concourse_images/atalanda-pipeline.png" class="background">

>>>

# Resources

* git, slack-notification, S3, docker, time...
* base on docker images
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

<img src="/slides/concourse_images/circle-ci.svg">
<img src="/slides/concourse_images/codeship.png">
<img src="/slides/concourse_images/travisci.png">
<img src="/slides/concourse_images/teamcity.jpg">

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
