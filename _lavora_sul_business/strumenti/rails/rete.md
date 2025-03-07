---
layout: default
title: Rete
parent: Web app rails
grand_parent: Strumenti 
nav_fold: true
has_children: true
nav_order: 5
---

# Struttura a rete

rails g scaffold Node name:string
rails g scaffold Connection node_1:references node_2:references