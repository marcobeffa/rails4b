---
layout: default
title: New nested branch
parent: Branch Tree
grand_parent:  Web app rails
nav_fold: true
has_children: true
nav_order: 7
---

# New nested branch


🧱 1. Vista edit.html.erb annidata
erb
Copia
Modifica
<h1>Modifica ramo: <%= @branch.name %></h1>

<!-- Genitore (livello 0) -->
<div class="tree-node">
  <strong><%= @branch.name %></strong>
</div>

<!-- Turbo Frame per figli -->
<turbo-frame id="branch_children_<%= @branch.id %>">
  <div class="children">
    <% @branch.children.each do |child| %>
      <div class="child-node">
        └── <%= link_to child.name, edit_branch_path(child) %>
      </div>
    <% end %>
  </div>
</turbo-frame>

<!-- Form per aggiungere figlio -->
<%= render "branches/form", branch: Branch.new(parent_id: @branch.id), parent: @branch %>
🧾 2. Form _form.html.erb
erb
Copia
Modifica
<%= form_with(model: branch, data: { turbo_frame: "branch_children_#{parent.id}" }) do |f| %>
  <%= f.hidden_field :parent_id, value: parent.id %>
  <%= f.text_field :name, placeholder: "Nuovo figlio..." %>
  <%= f.submit "Aggiungi" %>
<% end %>
⚙️ 3. Controller (branches_controller.rb)
ruby
Copia
Modifica
def create
  @branch = Branch.new(branch_params)
  if @branch.save
    @parent = @branch.parent
    respond_to do |format|
      format.turbo_stream do
        render turbo_stream: turbo_stream.update(
          "branch_children_#{@parent.id}",
          partial: "branches/children", locals: { branch: @parent }
        )
      end
      format.html { redirect_to edit_branch_path(@parent) }
    end
  else
    # gestione errori...
  end
end
🧩 4. Parziale branches/_children.html.erb
erb
Copia
Modifica
<div class="children">
  <% branch.children.each do |child| %>
    <div class="child-node">
      └── <%= link_to child.name, edit_branch_path(child) %>
    </div>
  <% end %>
</div>
