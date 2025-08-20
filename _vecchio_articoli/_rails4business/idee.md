---
layout: default
title: Idee
 
nav_fold: true
has_children: true
nav_order: 2
---

# Possibili Progetti


- skool.com


- mi servirebbe la scena dove lui parla con la barista che gli racconta di come il cuore e la pressione risolvano un problema abbassando la pressione e lui trova il modo per metterlo nella programmazione.

- CONOSCIUTO SCONOSCIUTO
- PERCHè COME COSA | CONTENUTO - COMUNICAZIONE INTERAZIONE - AZIONE | MOTIVO - SINTOMI - COSA SI FA SU COSA PUOI AGIRE |


- INTELLETTO - INFORMAZIONI - IDENTITA' MEMORIA

- PERCENTUALE DI COMPLETAMENTO

- STEINER MUSICA | RIPETIZIONE CONSAPEVOLE ANDAR IN BICI - MEMORIA AUTOMATICA - FARE UN ALTRA COSA - IMPROVVISAZIONE PROVARE PIACERE

- EVENTI A RETE 

- ROUTINE RITMO GIORNALIERO 
- IMPEGNO EVENTO ESPERIENZA

- VETTORE MATRICE - ALBERO - RETE - TAG 

- APPRENDIMENTO

- esercizi - schede - programma settimanale 

- attività - obiettivi progressione - costruire risultato - apprendimento


- The playlist il cuore e il suggerimento per il programmatore

- https://b12.io/client/iajtHzqR/site_builder/ 

- gamma

- https://web.descript.com/9c3dc533-2ce0-4203-9505-b0501569ff07/d168b?editorVariant=default

- sora

- open api from rails https://github.com/a-chacon/oas_rails

- https://www.skool.com/discovery

- eventi https://casticino.ch/?page_id=55

- https://ivanmartellato.com/ebook/

- impegno risorse tempo energia attenzione soldi energia relazioni - apprendere - comunicare - vendere

- seriale parallelo 




- memorizzazione vs apprendimento - https://www.youtube.com/watch?v=2tIdZ43Fchk
- generalizzazione 
- gradienti pesi concordi
- gradienti pesi discordi


#  2 maggio 08:40

Post instagram

# Ven 2 maggio 08:40

Video Youtube 



# Ven 2 maggio 08:01

rails new posturacorretta_app -d postgresql --css=tailwind
cd posturacorretta_app

# Aggiungi Flowbite
yarn add flowbite
# poi importa in app/javascript/application.js:
# import 'flowbite';

bundle add devise
rails generate devise:install
rails generate devise User
rails db:migrate


rails generate scaffold Course \
  title:string \
  description:text \
  url:string \
  image_copertina:text \
  controller:string
  action:string
rails db:migrate


rails generate scaffold Iscription course:references user:references
rails db:migrate

- bitbucket 
- hatchbox