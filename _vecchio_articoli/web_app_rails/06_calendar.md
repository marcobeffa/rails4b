---
layout: default
title: Flowpulse calendar
parent: Web app rails
nav_fold: true
has_children: true
nav_order: 6
---

```sh
rails new flowpulse_calendar -j importmap --database=postgresql --css=tailwind
cd flowpulse_branch

rails g controller Pages home about

```

senza npm solo importmap
https://flowbite.com/docs/getting-started/rails/

``` 
pin "flowbite", to: "https://cdn.jsdelivr.net/npm/flowbite@3.1.2/dist/flowbite.turbo.min.js"
import 'flowbite';

bin/rails g authentication

User.create(email_address: "mario@mario.it", password: "123456")

rails g scaffold Datecalendar user:references title:string description:text start_time:datetime end_time:datetime visibility:string routine_type:integer datecalendar_type:integer recurring:boolean parent_id:integer color:string 

```ruby
enum :routine_type, { day, week, month, quadrimetrale, trimestral, bimestrale, year, 7years }
enum :datecalendar_type,{ Business: 0, Project: 1, Target: 2, Task: 3, Event: 4, attività: 5 } 
  
```
imnportanza urgenza?

rails g Calendar mese settimana giorno agenda

