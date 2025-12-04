---
layout: default
title: Accesso utente base
parent: Posturacorretta
grand_parent:  Web app rails
nav_fold: true
has_children: true
nav_order: 02
---

````rb
get 'user_dashboard/:id', to: 'dashboards#user', as: :user_dashboard


````
resources :tutors do
  member do
   get "admin"
  end
  
  resources :assigneds
  resources :appointments
  resources :pathprofessionals
  resources :professionals
  resources :conversations

end


Invite user -> g user -> profile -> appointment -> Assigned 
