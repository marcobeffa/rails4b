---
layout: default
title: Record variabili
parent: Posturacorretta
grand_parent:  Web app rails
nav_fold: true
has_children: true
nav_order: 05
---


rails g scaffold Record treepage:references branch:references all_field_json:json current_field_json:json all_version:json current_version:integer

````html
<%= form_with(model: record) do |form| %>
  <%= form.label :branch_id %>
  <%= form.number_field :branch_id %>

  <%= form.label :treepage_id %>
  <%= form.number_field :treepage_id %>

  <h3>Campi dinamici</h3>
  <div id="dynamic-fields">
    <% record.current_field_json&.each do |key, value| %>
      <div class="field">
        <%= label_tag key.humanize %>
        <%= text_field_tag "record[current_field_json][#{key}]", value %>
      </div>
    <% end %>
  </div>

  <%= form.submit %>
<% end %>
````

def record_params
  params.require(:record).permit(:branch_id, :treepage_id, :all_version, :current_version, current_field_json: {})
end

Bisogna sistemare l'acquisizione dei nested Treepages