---
title:  Deploy hatchbox
has_children: true
---

# Come si crea un app con rails?

per prima cosa vai sul sito: [https://gorails.com/setup/macos/15-sequoia](https://gorails.com/setup/macos/15-sequoia) seleziona il tuo sistema operativo! E installa l'occorrente:

poi


rails new appname \
  -d postgresql \
  -j importmap \
  -c tailwind

cd appname
bin/setup
bin/rails db:create
bin/rails s

