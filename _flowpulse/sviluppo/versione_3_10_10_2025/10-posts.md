---
title: Posts
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 10
---



# Posts
bin/rails g scaffold Post \
  lead:references \
  title:string \
  slug:string:uniq \
  description:text \
  thumb_url:string \
  cover_url:string \
  banner_url:string \
  content:text \
  published_at:datetime \
  taxbranch:references \
  status:integer \
  meta:jsonb

    add_index :posts, :slug, unique: true
    add_index :posts, :published_at
    add_index :posts, :meta, using: :gin


    