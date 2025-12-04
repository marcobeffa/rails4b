---
title: Gestire eventi ruoli journey cicli
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 15
---


# Gstire eventi ruoli journey cicli



rails g scaffold Journeypath \
  preparatory_journeypath:integer \
  lead:references \
  visibility:integer \ 
  status:integer \
  name:string \
  description:text \
  certification_id:integer \
  meta:jsonb
  role:boolana
  role_name:string
  role_description:text

  
  visibility:enum [public, private, unlisted, subscription]

rails g scaffold Planjourney \
  taxbranch:references \
  lead:references \
  duration:integer \
  priced_cents:integer \
  priced_currency:string \
  crypto_amount:decimal{18,8} \
  crypto_currency:string \
  crypto_address:string \
  crypto_txid:string 
  role_journeypath_id:integer \
  max_number_of_participants:integer \
  min_number_of_participants:integer \
  description:text \
  meta:jsonb
  




rails g scaffold Certificationjourney \

rails g scaffold Journeyticket \
rails g scaffold Event
  
  [enrolled, in_progress, completed, certified]

cer


  


