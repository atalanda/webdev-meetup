![rails-logo](http://rubyonrails.org/images/rails-logo.svg)
# 5.2

Note: Kurz vorstellen (Namen, Werdegang, Aufgabenbereich). Rails: Quelloffenes Ruby Webframework. "Model View Controller" Architektur. Zwei Grundprinzipien "Don’t repeat yourself" und "Convention over Configuration".



## Release 8 Days ago


## RubyConf

Pittsburgh (USA)
17.-19.04.2018
![pitsburgh](slides/images/pitsburgh.jpg)

Note: Die größte und längste Konferenz für Ruby und Rails Enthusiasten


## Kick-off event in Freilassing!


## ![atalanda-logo](slides/images/atalanda.png) Anecdote

|          |                   |
|----------|-------------------|
|**5.2.0** | since 04/2018 	   |
|**5.1.3** | 04/2018 - 09/2017 |
|**4.2.9** | 09/2017 - 08/2017 |
|**3.2.X** | 08/2017 - 12/2013 |




## New Main Features
* Active Storage

* Redis Cache Store

* HTTP2 / Early Hints

* CSP

* Credentials

Note: Wir werden jetzt die neusten Main-Features durchgehen und Georg und ich werden euch erzählen ob und wie wir gedenken die Features in atalanda einzubauen.



## Active Storage


### File Uploads in Rails

Note: Super wichtiges Feature, allerdings hat man bis jetzt immer auf Plugins zurückgreifen müssen. Jetzt gibt es eine Implementierung im Rails Core, welche ein paar neue Features mit sich bringt, welche durch die Plugins nicht möglich waren.
Optional für Vortrag: `rails active_storage:install` erstellt eine Migration mit 2 Tabellen (Blobs und Attachments). Neue Features: FK und Direct-Upload.


### Model

```ruby
class User < ApplicationRecord
  has_one_attached :avatar
  has_many_attached :images
end
```
Note: Das neue has_one_attached Macro erstellt eine 1 zu 1 Beziehung zwischen einem User-Eintrag und einem File. Das has_many_attached Macro erstellt eine 1 zu N Beziehung zwischen einem User-Eintrag und mehreren Files. Das ganze funktioniert ohne Migration, da der Foreign Key nicht auf der User-Tabelle gespeichert wird, sondern auf der vom Active-Storage.


### View
```erb
<%= form.file_field :avatar %>
<%= form.file_field :images, multiple: true %>
```
```erb
<%= image_tag @user.avatar %>
<% @user.images.each do |image| %>
  <%= image_tag image %>
<% end %>
```


### Image Transformation

```ruby
gem 'mini_magick'
```
```erb
<%= image_tag @user.avatar.variant(resize: "100x100") %>
```
Note: On-the-fly Generierung von Varianten. Lazy Erstellen von Varianten, signierte permanente Referenz, expiring Service URL für den Download.


### Preview for PDFs and Videos
```ruby
gem "ffmpeg"
gem "mutool"
```
```erb
<%= image_tag file.preview(resize: "100x100>") %>
```
Note: On-the-fly Generierung einer Vorschau für PDFs und Videos. Braucht zwei zusätzliche Gems (ffmpeg und mutool)


### Many more additional features

* Direct-Upload (from client/browser to cloud)

* Progressbar

* Upload to Amazons S3, Googles Cloud Storage and Microsoft Azure Cloud File Storage

* Mirroring

* Asynchronous deletion

* ...


### atalanda

* Currently Paperclip and S3 (39 Million images)

* `has_attached_file` macro in model

* Variants get created on upload

* Switch to ActiveStorage doesn't have the highest priority



## Redis Cache Store

Note: Wer kennt Redis? Redis ist eine schnelle NoSQL In-Memory-Datenbank mit einer key-value Datenstruktur.


### Previous Cachespeicher (for Pages, Fragments, ...)

* MemoryStore (RAM)

* FileStore (file system)

* MemCacheStore (Memory object caching system)

* NullStore (Never saves anything)


`config/environments/*.rb`:

```ruby
config.cache_store = :redis_cache_store
```


### Redis Cache Store

* Fault-tolerant (doesn't throw exceptions)

* Automatically deletes least-recently or -frequently used keys on memory shortage

* Distributed redis servers possible

* Hot in-memory primary cache


### atalanda

* Currently using default cache store (FileStore)

* Increase in users as before: Redis Cache Store could be interesting



## HTTP2/Early Hints


### HTTP2/Early Hints

* Assets send before request finished

* Early Hints still in draft

* HTTP/2 and TLS as requirement

* `rails s --early-hints`

Note: Puma ist der Standard-Server wenn `rails s` zum Starten ausgeführt wird


### Process
<!-- .slide: data-background-color="#FFFFFF" -->
![early-hints-process](slides/images/early_hints.png)


### Pull request

[https://github.com/rails/rails/pull/30744](https://github.com/rails/rails/pull/30744/commits/59a02fb7bcbe68f26e1e7fdcec45c00c66e4a065)


![early-hints-meme](slides/images/early-hints-in-rails-of-course-will-we-use-it.jpg)



## Content Security Policy

### What is it about?

HTTP response header which allows to control resources the user agent is allowed to load for a given page. With a few exceptions, policies mostly involve specifying server origins and script endpoints. This helps guard against cross-site scripting attacks (XSS).

### Configuring CSP globally

```ruby
# config/initializers/content_security_policy.rb
Rails.application.config.content_security_policy do |policy|
  policy.default_src :self, :https
  policy.font_src    :self, :https, :data
  policy.img_src     :self, :https, :data
  policy.object_src  :none
  policy.script_src  :self, :https
  policy.style_src   :self, :https

  # Specify URI for violation reports
  policy.report_uri "/csp-violation-report-endpoint"
end

```

<!-- Content-Security-Policy: default-src 'self' *.trusted.com -->

###
