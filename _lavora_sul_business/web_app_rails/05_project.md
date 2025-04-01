---
layout: default
title: Flowpulse project
parent: Web app rails
nav_fold: true
has_children: true
nav_order: 5
---


## Flowpulse login

Aggiungere tutti i moduli e il login unico per far comunicare i moduli

[aggiornamenti flowpulse](/rails4business/idee/sul_business/flowpulse_aggiornamenti/#flowpulse-istruzioni-per-sviluppo-e-integrazione-mvp)


```sh
rails new flowpulse_project -j importmap --database=postgresql
cd flowpulse_branch

bundle add css-zero

bin/rails generate css_zero:install

bin/rails generate css_zero:add --help

bin/rails g authentication

User.create(email_address: "mario@mario.it", password: "123456")


rails g controller Pages home 


rails g scaffold Structure user:references branch_id:integer parent_id:integer structure_branch_link_child_id:integer tree_hash:jsonb name description


rails generate stimulus tree_loader


rails g scaffold Record user:references structure:references structure_branch_id:integer name description body:jsonb parent_id:integer


```
