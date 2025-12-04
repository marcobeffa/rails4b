---
layout: default
title: Branch Tree
parent: Web app rails
nav_fold: true
has_children: true
nav_order: 4
---


## Tree branch 


## Create app

```sh
rails new flowpulse_branch -j importmap --database=postgresql
cd flowpulse_branch

bundle add css-zero

bin/rails generate css_zero:install

bin/rails generate css_zero:add --help

bin/rails g authentication

User.create(email_address: "mario@mario.it", password: "123456")



rails g controller Pages home 
```

```sh
rails g scaffold Category user:references Category name:string description:text icon

rails g scaffold Mycategory user:references category:references name:string description:text icon public:boolean

rails g scaffold Branch user:references slug parent_id:integer nav_order content_id:integer slug_content user_content_username child_id:integer mycategory:references 
```

```rb
#user.rb
class User < ApplicationRecord
  has_secure_password
  has_many :sessions, dependent: :destroy

  normalizes :email_address, with: ->(e) { e.strip.downcase }
  has_many :mycategories
  has_many :categories
  has_many :branches
end


```




```rb
class Branch < ApplicationRecord
  belongs_to :user
  belongs_to :mycategory, optional: true

  # Gerarchia dell'albero (Parent-Child)
  belongs_to :parent, class_name: "Branch", foreign_key: "parent_id", optional: true
  has_many :children, class_name: "Branch", foreign_key: "parent_id", dependent: :destroy

  # Collegamenti tra rami separati
  belongs_to :link_child, class_name: "Branch", foreign_key: "child_id", optional: true
  has_many :linked_parents, class_name: "Branch", foreign_key: "child_id"

  # Gestisce l'ordine tra i figli dello stesso parent
  acts_as_list scope: :parent 

  # Validazioni
  validates :slug, presence: true, uniqueness: true
  validates :slug_content, uniqueness: true, allow_nil: true
end

```

```rb
# branches_controller.rb
```

## Gem act as list

gem 'acts_as_list'

## helpers

```rb 
# helpers_branch.rb

def root(id)
  Branch.find(id)

end 


```


## nestable sortjs

branches/show.html.erb





```sh
bin/importmap pin sortablejs
bin/rails generate stimulus sortable

```




```html

<div data-controller="sortable">
  <div id="nestedDemo" class="list-group col nested-sortable">
    <div class="list-group-item nested-1">Item 1.1
      <div class="list-group nested-sortable">
        <div class="list-group-item nested-2">Item 2.1</div>
        <div class="list-group-item nested-2">Item 2.2</div>
      </div>
    </div>
    <div class="list-group-item nested-1">Item 1.2</div>
  </div>
</div>

```
sortable.css
```css
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


```rb
# branches_controller.rb
   def updatenav_order
    @branch_root = @branch.root
    new_nav_order = params[:nav_order].to_i
    parent_id = params[:parent_id]

    respond_to do |format|
      if @branch.update(parent_id: parent_id)
        # acts_as_list è 1-based, ma se il valore è 0, forziamo a 1
        @branch.insert_at(new_nav_order)

        format.html { redirect_to @branch_root } # , notice:  "Branch spostato con successo." }
        format.json { render :show, status: :ok, location: @branch_root }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @branch.errors, status: :unprocessable_entity }
      end
    end
  end

```
_sortable.html.erb
```html
<div data-controller="sortable">
  <ul class="nested-sortable" data-sortable-target="list" data-id="<%= @branch.id %>">
    <%= render partial: "child", collection: @branch.children, as: :branch, locals: { parent_id: @branch.id } %>
  </ul>
</div>
```

_child.html.erb
```html
<li class="list-group-item" data-id="<%= branch.id %>">
  <span class="drag-handle">☰</span> <%= branch.slug %>
  <% if branch.children.any? %>
    <ul class="nested-sortable" data-sortable-target="list">
      <%= render partial: "child", collection: branch.children, as: :branch %>
    </ul>
    <% else %>
    <ul class="nested-sortable" data-sortable-target="list">
    </ul>
  <% end %>
</li>
```


```js
// sortable_controller.js

import { Controller } from "@hotwired/stimulus";
import Sortable from "sortablejs";

export default class extends Controller {
  static targets = ["list"];

  connect() {
    console.log("SortableJS initializing...");
    console.log(this.listTargets); // Debug

    this.listTargets.forEach(container => {
      new Sortable(container, {
        group: {
          name: "nested",
          pull: true, 
          put: true
        },
        animation: 150,
        fallbackOnBody: true,
        swapThreshold: 0.65,
        handle: ".drag-handle",
        onEnd: (event) => this.updatenav_order(event)
      });
    });
  }

  async updatenav_order(event) {
    const item = event.item;
    const parent = item.closest(".nested-sortable")?.closest(".list-group-item");
    const newParentId = parent ? parent.dataset.id : null;
    const newnav_order = event.newIndex + 1;
  
    console.log("Item ID:", item.dataset.id);
    console.log("New Parent ID:", newParentId);
    console.log("New nav_order:", newnav_order);
  
    try {
      const response = await fetch(`/branches/${item.dataset.id}/update_nav_order`, {
        method: "PATCH",
        headers: { 
          "Content-Type": "application/json", 
          "X-CSRF-Token": document.querySelector('meta[name="csrf-token"]').content 
        },
        body: JSON.stringify({ 
          nav_order: newnav_order, 
          parent_id: newParentId 
        }),
      });
  
      if (response.ok) {
        const data = await response.json();
        console.log("Updated successfully:", data);
  
        if (data.root_id) {
          window.location.href = `/branches/${data.root_id}`;  // Redirect alla root
        }
      } else {
        console.error("Update failed:", response);
      }
    } catch (error) {
      console.error("Request failed:", error);
    }
  }
} 


```




gem 'annotate'

```html
<!--_child.html.erb-->
<li class="list-group-item" data-id="<%= branch.id %>">
   <%= branch.nav_order %> - <span class="drag-handle">☰</span> 
  - <%= branch.slug %> 
  <% if branch.children.any? %>
    <ul class="nested-sortable" data-sortable-target="list">
      <%= render partial: "child", collection: branch.children.order(:nav_order), as: :branch %>
    </ul>
    <% else %>
    <ul class="nested-sortable" data-sortable-target="list">
    </ul>
  <% end %>
</li>

```

```html
<!--_sortable.html.erb-->
<div data-controller="sortable">
  <li class="list-group-item" data-id="<%= @branch.id %>" data-parent-id="<%= @branch.parent_id %>">
    <span class="drag-handle">☰</span> <%= @branch.slug %>
    <ul class="nested-sortable" data-sortable-target="list">
      <%= render partial: "child", collection: @branch.children.order(:nav_order), as: :branch %>
    </ul>
  </li>
</div>

```

```rb
# branches_controller.rb
  def update_nav_order
    parent_id = params[:parent_id].presence || nil
    new_nav_order = params[:nav_order].to_i
  
    if @branch.update(parent_id: parent_id, nav_order: new_nav_order)
      @branch.insert_at(new_nav_order)  # Forza la posizione
      
      # Trova la radice della gerarchia dopo l'aggiornamento
      root_branch = @branch.root
  
      respond_to do |format|
        format.json { render json: { success: true, nav_order: @branch.nav_order, parent_id: @branch.parent_id, root_id: root_branch.id } }
        format.html { redirect_to branch_path(root_branch), notice: "Branch aggiornato con successo!" }
      end
    else
      respond_to do |format|
        format.json { render json: { success: false, errors: @branch.errors.full_messages }, status: :unprocessable_entity }
        format.html { redirect_back fallback_location: branches_path, alert: "Errore nell'aggiornamento della posizione." }
      end
    end
  end

```

```rb
# branch.rb
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

## Controllare perchè non fa il redirect

## Mettere la vista ad albero e la vista d3j 

## Vedere per il new children

## Per la vista edit vs show dove mettere i link

## vedere per le categorie
Contatto
Progetti

Capitoli 


Gli Eventi lasciarli su 1 impegno o spostarli ?
Transazioni 
Contatti ecc.

## branch_helper.rb

module BranchesHelper
  def parent_chain(id)
    parent_ids = []
    branch = Branch.find_by(id: id)
    
    while branch&.parent_id
      parent_ids << branch.parent_id
      branch = Branch.find_by(id: branch.parent_id)
    end

    parent_ids.reverse
  end
  def tree_to_hash(branch)
    return {} if branch.nil?

    {
      id: branch.id,  # <-- Aggiunto ID per il link corretto
      name: branch.slug,
      icon: branch.mycategory&.icon || '➖',
      children: branch.children.map { |child| tree_to_hash(child) }
    }
  end
  def hash_to_ascii(tree, prefix = "", parent_prefix = "")
    return "" if tree.nil? || tree.empty?

    # Creiamo un link con l'ID corretto del nodo
    node_link = "<a href='/branches/#{tree[:id]}'>#{tree[:icon]} #{tree[:name]}</a>"
    tree_content = "#{prefix}#{node_link}\n"

    tree[:children].each_with_index do |child, index|
      is_last = index == tree[:children].length - 1
      child_prefix = parent_prefix + (is_last ? "└── " : "├── ")
      new_parent_prefix = parent_prefix + (is_last ? "    " : "│   ")

      tree_content += hash_to_ascii(child, child_prefix, new_parent_prefix)
    end

    tree_content
  end
  
  def markdown(text)
    renderer = Redcarpet::Render::HTML.new(hard_wrap: true, filter_html: true)
    md = Redcarpet::Markdown.new(renderer, autolink: true, tables: true)
    md.render(text).html_safe
  end
end

## 27 mar 2025 
```sh
rails generate migration RemoveMycategoryFromBranches mycategory:references

rails generate migration DropCategoriesAndMycategories
rails destroy scaffold Mycategory
rails destroy scaffold Category
```
```rb
class DropCategoriesAndMycategories < ActiveRecord::Migration[8.0]
  def change
    remove_foreign_key :mycategories, :categories
    drop_table :categories
    drop_table :mycategories
  end
end
```

Api 
```sh 

rails generate scaffold ExternalPost branch:references  api_variabile:string content:json
rails g migration RemoveIconFromBranches icon:string

rails g migration AddRootToBranches index_branch:boolean



```sh
rails g controller api/v1/trees  --skip-template-engine --skip-assets --skip-helper

rails generate migration AddDefaultsToBranches
rails generate migration AddStructureToBranches structure:boolean:default:false
```

branch_root_id → radice della struttura

branch_id → nodo a cui è collegato (es. progetto, modulo)

actor_ids → array JSON di user o ruoli coinvolti

📌 Task (impegno operativo)
bash
Copia
Modifica
rails g model Task content_id:integer user_id:integer status:string start_date:datetime due_date:datetime
user_id → assegnatario

content_id → cosa deve fare (la compilazione o blocco informativo)

🔁 Request (richiesta o passaggio)
bash
Copia
Modifica
rails g model Request task_id:integer status:string payload:jsonb
può essere usata per workflow tra task (es. approvazioni, richieste info)

🗓️ DataEvent (evento reale)
bash
Copia
Modifica
rails g model DataEvent task_id:integer user_id:integer data:datetime place_id:integer contact_id:integer transaction_id:integer body:jsonb
Quando un Task è completato → si registra qui

Collegabile a:

Place

Contact

Transaction

💶 Transaction
bash
Copia
Modifica
rails g model Transaction data:datetime amount:decimal currency:string description:string
📍 Place
bash
Copia
Modifica
rails g model Place name:string address:string geolocation:jsonb
👤 Contact
bash
Copia
Modifica


rails g scaffold Datecalendar user:references title:string description:text start_time:datetime end_time:datetime area:string event_type:string visibility:string completed:boolean


rails g migration RemoveStructureFromBranches structure:boolean
rails g migration AddContentToBranch content:boolean 


def change
  add_column :branches, :content, :boolean, default: true, null: false
end


# Hash completo 
# Tree Menu attivo 
# External Post
# Label


rails generate migration ChangeContentToUpdatedContentInBranches

rails g migration RemoveLabelFromBranch label:boolean

rails g migration AddExtrafieldsToBranch field:boolean field_type:integer