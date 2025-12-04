---
title: Taxonomy treebranch
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 08
---

# Taxonomy treebranch

bin/rails g scaffold Lead name surname username email phone token parent_id referral_lead_id meta


bin/rails g scaffold Taxbranch \
lead:references \
name \
slug \
slug_category \
slug_label \
ancestry:string \
position:integer \
meta:jsonb \


bin/rails g scaffold  Brand 

bin/rails g scaffold  Project 

euro_cents:integer \
dash_duffs:integer \
start_project:datetime \
end_project:datetime \
routine, activity, o project? 
e mettere anche i cicle? 





bin/rails g scaffold Brandprofile lead:references name color slug:index domains principal_domain taxbranch:references
bin/rails g scaffold Post lead:references title description slug img_square_url img_vertical_url img_orizontal_url published_at group, polymorphic: true, null: true meta:jsonb