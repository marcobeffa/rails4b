---
title: 2 Controller pages
parent: Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 2
---



rails g controller pages home about contact

rails g controller dashboard home superadmin 


Crea un constraint che guarda la sessione impostata dal generator:
````rb
# app/constraints/authenticated_constraint.rb
class AuthenticatedConstraint
  def matches?(req)
    req.session[:current_user_id].present?
  end
end
````
````rb
# config/routes.rb
constraints AuthenticatedConstraint.new do
  root to: "dashboard#show", as: :authenticated_root
end

# fallback per chi NON è autenticato
root to: "landing#show", as: :unauthenticated_root
````
````sh
rails g authentication
````
````rb
superadmin: boolean
````

````html
<% folder = request.host %>
<% begin %>
  <%= render "#{folder}/home" %>
  <% rescue ActionView::MissingTemplate %>
  <%= render "pages/flowpulse/home" %>
<% end %>
````