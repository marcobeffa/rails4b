---
title: 1 First commit 
parent: Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 1
---
# creata l'app
rails new flowpulse \
  -d postgresql \
  -j importmap \
  -c tailwind

cd flowpulse
bin/setup
bin/rails db:create
bin/rails s


# brands e services routes

- :roles 
- :domain -> brand
- :folder-trees
- :folder -> folder attuale
- :key -> services di servizio legato a un subdomain
- :slug -> nome del file yml

- :events
- :posts
- :courses
- :lessons