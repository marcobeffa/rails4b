---
layout: default
title: Trees
parent: Branch Tree
grand_parent:  Web app rails
nav_fold: true
has_children: true
nav_order: 5
---

## Pubblicazione 

rails g migration AddFieldtoBranch visibility:integer published: boolean stato:integer


## 
```yml
it:
  activerecord:
    attributes:
      branch:
        visibility:
          privato: "Privato"
          iscritti: "Solo iscritti"
          pubblico: "Pubblico"
        stato:
          bozza: "Bozza"
          organizzata: "Organizzata"
          corretto: "Corretto"
```

```html
<%= f.select :visibility, Branch.visibilities.keys.map { |v| [t("activerecord.attributes.branch.visibility.#{v}"), v] } %>
```

```html
<h1><%= @branch.title %></h1>

<!-- PUBLISHED -->
<%= turbo_frame_tag "branch_published_#{@branch.id}" do %>
  <p>
    <strong>Pubblicato:</strong>
    <%= link_to (@branch.published? ? "✅ Sì" : "❌ No"),
        "#",
        onclick: "document.getElementById('edit_published_form').style.display = 'inline'; this.style.display = 'none'; return false;" %>

    <%= form_with model: @branch, url: branch_path(@branch), method: :patch,
        data: { turbo_frame: "branch_published_#{@branch.id}" },
        html: { id: "edit_published_form", style: "display:none;" } do |f| %>
      <%= f.check_box :published, {}, true, false, onchange: "this.form.requestSubmit()" %> Pubblica
    <% end %>
  </p>
<% end %>

<!-- VISIBILITY -->
<%= turbo_frame_tag "branch_visibility_#{@branch.id}" do %>
  <p>
    <strong>Visibilità:</strong>
    <%= link_to t("activerecord.attributes.branch.visibility.#{@branch.visibility}"),
        "#",
        onclick: "document.getElementById('edit_visibility_form').style.display = 'inline'; this.style.display = 'none'; return false;" %>

    <%= form_with model: @branch, url: branch_path(@branch), method: :patch,
        data: { turbo_frame: "branch_visibility_#{@branch.id}" },
        html: { id: "edit_visibility_form", style: "display:none;" } do |f| %>
      <%= f.select :visibility,
            Branch.visibilities.keys.map { |v| [t("activerecord.attributes.branch.visibility.#{v}"), v] },
            {}, onchange: "this.form.requestSubmit()" %>
    <% end %>
  </p>
<% end %>

<!-- STATO -->
<%= turbo_frame_tag "branch_stato_#{@branch.id}" do %>
  <p>
    <strong>Stato:</strong>
    <%= link_to t("activerecord.attributes.branch.stato.#{@branch.stato}"),
        "#",
        onclick: "document.getElementById('edit_stato_form').style.display = 'inline'; this.style.display = 'none'; return false;" %>

    <%= form_with model: @branch, url: branch_path(@branch), method: :patch,
        data: { turbo_frame: "branch_stato_#{@branch.id}" },
        html: { id: "edit_stato_form", style: "display:none;" } do |f| %>
      <%= f.select :stato,
            Branch.statos.keys.map { |s| [t("activerecord.attributes.branch.stato.#{s}"), s] },
            {}, onchange: "this.form.requestSubmit()" %>
    <% end %>
  </p>
<% end %>

```

```sh
rails g controller Trees index show
rails g migration AddUsernameToUser username:string
```



## recuperare content_id