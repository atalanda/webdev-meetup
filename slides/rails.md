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

Rails Version Marketplace

- 04.2017 - today **5.2.0**
- 09.2017 - 04.2017 **5.1.3**
- 08.2017 - 09.2017 **4.2.9**
- 12.2013 - 08.2017 **3.2.X**



## New Main Features
* Active Storage
* Redis Cache Store
* HTTP2 / Early Hints
* CSP
* Credentials

Note: Wir werden jetzt die neusten Main-Features durchgehen und Georg und ich werden euch erzählen ob und wie wir gedenken die Features in atalanda einzubauen.



## Active Storage


### File Uploads in Rails

Note: Super wichtiges Feature, allerdings hat man bis jetzt immer auf Plugins zurückgreifen müssen. Seit der neuen Version hat man die nun folgende Möglichkeiten. Optional für Vortrag: `rails active_storage:install` erstellt eine Migration mit 2 Tabellen (Blobs und Attachments).


### Model

```ruby
class User < ApplicationRecord
  has_one_attached :avatar
  has_many_attached :images
end
```
Note: Das neue has_one_attached Macro erstellt eine 1 zu 1 Beziehung zwischen einem User-Eintrag und einem File. Das has_many_attached Macro erstellt eine 1 zu N Beziehung zwischen einem User-Eintrag und mehreren Files.


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
Note: On-the-fly Generierung von Varianten


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
* Upload to Amazon’s S3, Googles Cloud Storage and Microsoft Azure Cloud File Storage
* Mirroring
* Asynchronous deletion
* ...


### atalanda

* Currently Paperclip and S3
* `has_attached_file` macro in model
* Variants get created on upload
* Switch to ActiveStorage does not have the highest priority



## Redis Cache Store

Note: Wer kennt Redis? Redis ist eine schnelle NoSQL In-Memory-Datenbank mit einer key-value Datenstruktur.


## Previous Cachespeicher (for Pages, Fragments, ...)
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
* Cache little, optimize performance
* Increase in users as before: Redis Cache Store could be interesting



## HTTP2/Early Hints


### HTTP2/Early Hints

* Assets send before request finished
* Early Hints still in draft
* HTTP/2 and TLS as requirement
* `rails s --early-hints`

Note: Puma ist der Standard-Server wenn `rails s` zum Starten ausgeführt wird


### Pull request

https://github.com/rails/rails/pull/30744/commits/59a02fb7bcbe68f26e1e7fdcec45c00c66e4a065


![early-hints](slides/images/eary-hints-in-rails-of-course-will-we-use-it.jpg)
