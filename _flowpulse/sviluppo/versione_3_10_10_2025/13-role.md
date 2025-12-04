---
title: Role
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 13
---


# Role



rails g scaffold Role \
  work_journeymap_id:integer \
  creator_lead:integer \
  name:string \
  description:text \
  img_url:string \
  meta:jsonb \
  parent_id:integer 

rails g scaffold Journeymap
  creator_lead_id:integer \
  name:string \
  description:text \
  started_at:datetime \
  ended_at:datetime \
  meta:jsonb \
  taxbranch:references \
  certified_name:string \
  certified_description:text \
  certified_img_url:string \
  granted_role_id:integer 
 
rails g scaffold journeystep 
  name:string \
  description:text \
  img_url:string \
  meta:jsonb \
  order:integer \
  journeymap:references

rails g model certification 
  journeymap:references \
  lead:references \
  received_role_id:references \
  name:string \
  description:text \
  img_url:string \
  meta:jsonb
  rilasciata_datetime:datetime \
  rilasciata_da_role_id:references \



rails g scaffold Journey
  journeymap:references \
  lead:references \
  started_at:datetime \
  ended_at:datetime \
  status:integer \
  meta:jsonb


rails g scaffold Event 
  journey:references \
  journeymap:references \
  role:references \
  lead:references \
  kind:integer \
  status:integer \
  scheduled_at:datetime \
  started_at:datetime \
  completed_at:datetime \
  occurred_at:datetime \
  metadata:jsonb \
  priced_cents:integer \
  priced_currency:string \
  crypto_amount:decimal{18,8} \
  crypto_currency:string \
  crypto_address:string \
  crypto_txid:string 