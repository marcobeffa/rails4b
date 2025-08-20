---
layout: default
title: Treepage
parent: Posturacorretta
grand_parent: Web app rails
nav_fold: true
has_children: true
nav_order: 06
---


Book Webpage Survey


````rb
class CreateTreepages < ActiveRecord::Migration[8.0]
  def change
    create_table :treepages do |t|
      t.string  :slug
      t.string  :title
      t.string  :image
      t.string  :description
      t.text    :content_md  
      t.integer :branch_id
      t.boolean :published, default: false, null: false

      # Migliorati:
      t.integer :visibility
      t.integer :kind_path       # { stop_al_dolore: 0, benessere_integrato: 1, accademia: 2, lavora_con_noi: 3, investi_sostieni_sponsorizza: 4, il_giardino_del_corpo: 5, generale: 6 }
      t.integer :kind_format     # es: book, webpage, question, answer
      
      # Struttura e relazioni
      t.integer :number_levels
      t.jsonb   :parent_link_ids, default: []
      t.jsonb   :children_link_ids, default: [] 
      t.integer :root_id

      # Campi JSON
      t.jsonb   :json_tree, default: {}
      t.jsonb   :json_root_menu, default: {}
      t.jsonb   :json_menu, default: {}
      t.jsonb   :json_fields, default: {}
   


      t.timestamps
    end
    add_index :treepages, :branch_id, unique: true, where: "branch_id IS NOT NULL", name: "index_unique_branch_id_not_null"
  end
end
`````

````rb
# controller 
def import
  @branch_id = params[:branch_id]
end

def do_import
  branch_id = params[:branch_id]

  begin
    Rails.logger.debug "Avvio importazione da branch_id=#{branch_id}"
    @treepage = import_root_page_from_branch_id(branch_id)
    raise "Nessuna pagina creata" unless @treepage

    redirect_to edit_treepage_path(@treepage), notice: "Importazione completata da branch_id #{branch_id}"
  rescue => e
    Rails.logger.error "Errore durante l'import: #{e.message}"
    redirect_to import_treepages_path, alert: "Errore: #{e.message}"
  end
end

def import_root_page_from_branch_id(branch_id)
  response = Faraday.get("https://branch.flowpulse.net/api/v1/branches/#{branch_id}/treepage")
  return nil unless response.success?

  root = JSON.parse(response.body).deep_symbolize_keys

  @treepage = Treepage.find_or_initialize_by(branch_id: branch_id)
  @treepage.assign_attributes(
    slug: root[:content][:slug],
    title: root[:content][:title],
    description: root[:content][:description],
    image: root[:content][:image],
    content_md: root[:content][:body],
    published: root[:content][:published],
    visibility: Treepage.visibilities[root[:visibility]],
    # kind_path e kind_format li lasci da scegliere a mano dopo
    parent_link_ids: root[:parent_links].map { |pl| pl[:grand_parent_branch_id] },
    children_link_ids: root[:children].map { |ch| ch[:branch_id] },
    number_levels: root[:children].size,
    root_id: root[:branch_id],
    json_tree: root,
    json_menu: root[:children],
    json_fields: root[:fields]
  )

  @treepage.save!
  @treepage
end

````
````rb
# route
resources :treepages do
  collection do
    get  :import
    post :do_import
  end
end
````



````rb
# edit vecchio
 if @treepage.parent_id.nil?
      @treepage_root = @treepage
    else
      @treepage_root = Treepage.find_by(id: @treepage.parent_id)
    end
      if @treepage.hash_tree.nil?
        response = Faraday.get("https://branch.flowpulse.net/api/v1/branches/#{@treepage_root.branch_id}/treepage")
        if response.success?
          @tree_hash = response.body
        end
     
      else
        if  @treepage.hash_tree_menu.blank?
          response_menu = Faraday.get("https://branch.flowpulse.net/api/v1/branches/#{@treepage.branch_id}/menu")
          if response_menu.success?
            @tree_menu = response_menu.body

          end
        else
          @tree_menu =  @treepage.hash_tree_menu
        end

        if @treepage.hash_fields.blank?
          response_fields = Faraday.get("https://branch.flowpulse.net/api/v1/branches/#{@treepage.branch_id}/fields")
          if response_fields.success?
            @fields = response_fields.body
          end
        end
        if  @treepage.content_md.blank?
          response_menu = Faraday.get("https://branch.flowpulse.net/api/v1/branches/#{@treepage.branch_id}/content")
          if response_menu.success?
            @content_md = response_menu.body

          end
        end
      end
      ````


````html
_show_book.html.erb
-------------------
<div class="p-6 bg-white rounded-2xl shadow-md">
  <h1 class="text-2xl font-bold mb-4"><%= treepage.title %></h1>
  <p class="text-gray-500 mb-2"><%= treepage.description %></p>
  <div class="prose max-w-none">
    <%= markdown(treepage.content_md) %>
  </div>
</div>

_show_webpage.html.erb
----------------------
<div class="p-4 bg-gray-50 border rounded-xl">
  <header class="mb-3">
    <h2 class="text-xl font-semibold"><%= treepage.title %></h2>
    <small class="text-gray-400 italic"><%= treepage.slug %></small>
  </header>
  <div class="prose max-w-none">
    <%= markdown(treepage.content_md) %>
  </div>
</div>

_show_survey.html.erb
---------------------
<div class="bg-yellow-50 border-l-4 border-yellow-400 p-5 rounded">
  <h3 class="font-bold text-lg text-yellow-800 mb-3">Questionario: <%= treepage.title %></h3>
  <p class="text-sm text-yellow-700 mb-4"><%= treepage.description %></p>

  <div class="space-y-4">
    <% if treepage.json_fields.present? %>
      <% treepage.json_fields.each do |field| %>
        <div>
          <label class="block font-medium"><%= field["label"] %></label>
          <% case field["type"] %>
          <% when "string" %>
            <input type="text" class="border rounded px-2 py-1 w-full" />
          <% when "text" %>
            <textarea class="border rounded px-2 py-1 w-full"></textarea>
          <% when "boolean" %>
            <input type="checkbox" class="mr-2" /> Sì
          <% end %>
        </div>
      <% end %>
    <% else %>
      <p class="text-sm italic">Nessun campo disponibile.</p>
    <% end %>
  </div>
</div>

_show_default.html.erb
----------------------
<div class="bg-gray-100 border rounded p-5">
  <h2 class="text-lg font-semibold"><%= treepage.title %></h2>
  <div class="mt-3 prose">
    <%= markdown(treepage.content_md) %>
  </div>
</div>

````