---
layout: default
title: Business model
parent: Branch Tree
grand_parent: Web app rails
nav_fold: true
has_children: true
nav_order: 4
---

## Per aggiungere modelli di business
[aggiornamenti flowpulse](/rails4business/idee/sul_business/flowpulse_aggiornamenti/#flowpulse-istruzioni-per-sviluppo-e-integrazione-mvp)


```rb
# routes.rb
   resources :businesses, only: %i[ show index ]
  resources :projects, only: %i[ show index ]
```

```sh
rails g controller Businesses index show
rails g controller Projects index show
rails g migration  AddAdminToUser admin:boolean
```

```rb
# mycategories_controller.rb

def create_from_category
  category = Category.find(params[:id])

  # Cerca se già esiste per quell'utente
  existing = Mycategory.find_by(user: Current.user, category: category)

  if existing
    redirect_to mycategory_path(existing), alert: "Hai già una copia di questa categoria."
    return
  end

  mycategory = Mycategory.new(
    name: category.name.parameterize(separator: "_"),
    description: category.description,
    category: category,
    user: Current.user
  )

  if mycategory.save
    redirect_to mycategory_path(mycategory), notice: "Categoria copiata con successo!"
  else
    redirect_back fallback_location: categories_path, alert: "Errore nella creazione della copia."
  end
end

```

```rb
# routes.rb
resources :categories do
  post 'copy_to_mycategory', on: :member, to: 'mycategories#create_from_category'
end
```



