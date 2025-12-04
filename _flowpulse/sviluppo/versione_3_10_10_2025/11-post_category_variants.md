---
title: Post category variants
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 11
---

# Blog Creators

Se ho taxbranch con slug_category "blog_creators" e un @post.taxbranch_id @taxbranch has_one post associato .

taxbranch controller lo uso in superadmin
post in public per visualizzare show

così ho superadmin taxbranch e post associati public post show 

solo che vorrei dare la possibilità di cambiare al posto di show  

show.html.erb 

le slug_category che non hanno uno show associato: 
mettere render "show/default" 

@post.taxbranch.slug_category == "blog_creators" 

mettere render "show/blog_creators"  


come organizzo il tutto per tenere pulito il codice? che ne dici hai suggerimenti?




1) Variants per slug_category (consigliata)

Controller (public/posts_controller.rb)
Imposta una variant in base a @post.taxbranch.slug_category, con default.

def show
  @post = Post.find(params[:id])

  slug = @post.taxbranch&.slug_category.to_s
  request.variant = if slug.present?
    slug.parameterize.underscore.to_sym
  else
    :default
  end
end


Views (app/views/posts/)
Crea i template varianti solo dove ti servono:

app/views/posts/show.html.erb ← default (usato come fallback automatico)

app/views/posts/show.html+blog_creators.erb ← custom per blog_creators
(Rails userà questo quando request.variant = :blog_creators)

Se domani hai un’altra categoria (es. events), basta aggiungere:

app/views/posts/show.html+events.erb

Pro:

Nessun if/else nelle view.

Fallback automatico al show.html.erb senza scrivere codice extra.

Struttura super pulita e “convenzionale”