---
layout: default
title: Sortablejs nested list
parent: Branch Tree
grand_parent:  Web app rails
nav_fold: true
has_children: true
nav_order: 1
---
```rb
#route.rb
 resources :branches do
    member do
      get :updatenav_order
      get :mappa
      get :ul
    end
  end
```

```rb
#controller_branches.rb
def updatenav_order
  @branch_root = @branch.root
  new_nav_order = params[:nav_order].to_i
  parent_id = params[:parent_id]

  respond_to do |format|
    if @branch.update(parent_id: parent_id)
      # acts_as_list è 1-based, ma se il valore è 0, forziamo a 1
      @branch.insert_at(new_nav_order)

      format.html { redirect_to ordinabile_branch_path(@branch_root) } # , notice:  "Branch spostato con successo." }
      format.json { render :show, status: :ok, location: ordinabile_branch_path(@branch_root) }
    else
      format.html { render :edit, status: :unprocessable_entity }
      format.json { render json: @branch.errors, status: :unprocessable_entity }
    end
  end
end
```

```rb
# branch.rb
# == Schema Information
#
# Table name: branches
#
#  id                 :integer          not null, primary key
#  user_id            :integer          not null
#  slug               :string
#  parent_id          :integer
#  nav_order           :integer
#  content_id         :integer
#  slug_note          :string
#  user_note_username :string
#  child_id           :integer
#  mycategory_id      :integer          not null
#  created_at         :datetime         not null
#  updated_at         :datetime         not null
#
# Indexes
#
#  index_branches_on_mycategory_id  (mycategory_id)
#  index_branches_on_user_id        (user_id)
#

class Branch < ApplicationRecord
  belongs_to :user
  belongs_to :mycategory, optional: true

  has_many :category, through: :mycategory

  # Gerarchia dell'albero (Parent-Child)
  belongs_to :parent, class_name: "Branch", foreign_key: "parent_id", optional: true
  has_many :children, class_name: "Branch", foreign_key: "parent_id"

  # Collegamenti tra rami separati
  belongs_to :link_child, class_name: "Branch", foreign_key: "child_id", optional: true
  has_many :linked_parents, class_name: "Branch", foreign_key: "child_id", dependent: :nullify

  # Gestisce l'ordine tra i figli dello stesso parent
  acts_as_list scope: :parent_id

  # Validazioni
  validates :slug, presence: true, uniqueness: true
  #  validates :slug_note, uniqueness: true, allow_nil: true
  def root
    self.class.where(id: self_and_ancestors_ids.first).first
  end

  def self_and_ancestors_ids
    ids = [ id ]
    current = self
    while current.parent_id.present?
      ids.unshift(current.parent_id)
      current = current.parent
    end
    ids
  end
end

```

```html
<!--branches/show.html.erb-->
<nav class="tabs__list" aria-label="Home and Settings">
  <%= link_to "Mappa", branch_path(@branch), class: "btn tabs__button", aria: { current: "page" } %>
  <%= link_to "Ordinabile", ordinabile_branch_path(@branch), class: "btn tabs__button" %>
  <%= link_to "Ul", ul_branch_path(@branch), class: "btn tabs__button" %>
</nav>
<%= render 'tree_ascii' %>
```
```html
<!--branches/mappa.html.erb-->
<nav class="tabs__list" aria-label="Home and Settings">
  <%= link_to "Mappa", branch_path(@branch), class: "btn tabs__button" %>
  <%= link_to "Ordinabile", ordinabile_branch_path(@branch), class: "btn tabs__button", aria: { current: "page" } %>
  <%= link_to "Ul", ul_branch_path(@branch), class: "btn tabs__button" %>
</nav>
<div class="flex rounded-lg border">
  <div class="flex items-center justify-center p-3 border-ie resizable-inline">
    <%= render 'sidebar' %>
  </div>
  <div class="flex  grow p-3">
    <div class="">
      <h1 class="text-xl font-bold"><%= @branch.slug %><%= link_to "New children", new_branch_path(parent_id: @branch.id), class: "btn m-4" %></h1>
      <div class="flex flex-wrap items-center gap mb-2" >
        <%= link_to "Edit this branch", edit_branch_path(@branch), class: "btn" %>
        <%= link_to "Back to branches", branches_path, class: "btn" %>
        <%= button_to "Destroy this branch", @branch, method: :delete, class: "btn" %>
      </div>
      <%= render "branches/sortable_simple", branches: @branch.children, branch: @branch %>
    </div>
  </div>
</div>
<%#= render 'tree_ascii' %>
<%#= render "ul_list", branches: @branch.children, parent_id: nil %>
</div>
<script src="https://cdn.jsdelivr.net/npm/sortablejs@1.15.6/Sortable.min.js"></script>
<script>
  function makeSortable(element) {
    new Sortable(element, {
      group: 'nested',
      animation: 150,
      fallbackOnBody: true,
      swapThreshold: 0.65,
      onAdd: function (evt) {
        // Apply sortable to new nested lists
        let nestedUl = evt.item.querySelector('ul');
        if (nestedUl) makeSortable(nestedUl);
      },
      onEnd: function (evt) {
        let item = evt.item;
        let itemId = item.dataset.id;
        let parentUl = item.closest('ul');
        let parentLi = parentUl.closest('li');
        let parentId = parentLi ? parentLi.dataset.id : null;

        // Calcola la nuova posizione (index all’interno della lista)
        let newnav_order = Array.from(parentUl.children).indexOf(item);

        // Esegui redirect alla nuova posizione
        if (itemId && parentId != null && newnav_order != null) {
          const correctednav_order = parseInt(newnav_order) + 1;

          const url = `/branches/${itemId}/updatenav_order?parent_id=${parentId}&nav_order=${correctednav_order}`;

          window.location.href = url;
        }
      }

    });

    // Rendere anche le sotto-liste sortable
    Array.from(element.querySelectorAll('ul')).forEach(ul => makeSortable(ul));
  }

  // Inizializza il sortable principale
  makeSortable(document.getElementById('nested-list'));
</script>


```

```html
<!--branches/sortable_simple.html.erb-->
<ul>
  <li class="list-group-item " data-id="<%= @branch.id %>" data-parent-id="<%= @branch.parent_id %>">
    <span class="drag-handle">☰</span> <%= @branch.slug %>
    <ul id="nested-list" class="nested-sortable" data-sortable-target="list">
      <%= render partial: "sortable_child_simple", collection: @branch.children.order(:nav_order), as: :branch %>
    </ul>
  </li>
</ul>


```
```html
<!--branches/sortable_simple_child.html.erb-->
<li class="list-group-item" data-id="<%= branch.id %>">
  <span class="drag-handle">☰</span>
  id: <%= branch.id %> - <%= branch.slug %>
  <% if branch.children.any? %>
    <ul class="nested-sortable" data-sortable-target="list">
      <%= render partial: "sortable_child_simple", collection: branch.children.order(:nav_order), as: :branch %>
    </ul>
  <% else %>
    <ul class="nested-sortable" data-sortable-target="list">
    </ul>
  <% end %>
</li>

```
```css
/* sortable.css */
 .list-group {
    display: block;
    padding: 10px;
    border: 1px solid #ddd;
    background-color: #f8f8f8;
    margin-bottom: 10px;
  }

  .list-group-item {
    padding: 10px;
    background: white;
    border: 1px solid #ccc;
    margin: 5px 0;
    cursor: grab;
    border-radius: 5px;
  }

  /* Quando l'elemento viene trascinato */
  .sortable-ghost {
    opacity: 0.5;
    background: #ddd;
  }
```