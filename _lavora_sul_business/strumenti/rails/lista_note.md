---
layout: default
title: Lista note e Business
parent: Web app rails
grand_parent: Strumenti 
nav_fold: true
has_children: true
nav_order: 3
---

# Lista note blog e Business

rails g scaffold Note title description body visibility:integer data_pubblicazione:datetime pubblicato:references

rails g scaffold Business name url 


rails g scaffold Link 
