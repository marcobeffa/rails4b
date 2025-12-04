---
title: List Engine
parent: Componenti engine gem
layout: default
nav_fold: true 
has_children: true
nav_order: 1
---

ActiveRecord::Base.transaction do
  @list.list_items.destroy_all
  build_nodes(@list, nodes, nil)
end

def build_nodes(list, arr, parent)
  arr.each_with_index do |node, idx|
    item = list.list_items.create!(
      parent: parent,
      title:  node["title"].to_s,
      slug:   (node["slug"].presence || node["title"].to_s.parameterize),
      nav_order: idx,
      data:   node.except("title","slug","children","client_id") # ignora client_id
    )
    build_nodes(list, node["children"] || [], item)
  end
end
Nell’editor: esporta così (niente id rails, solo client):

js
Copia codice
return { client_id: id, title, slug, children: kids };
Strategia 2 — Upsert/Merge (quando vuoi aggiornare senza perdere gli id DB)
Aggiungi un identificatore stabile esterno:

bash
Copia codice
rails g migration AddExternalUidToListItems external_uid:string
Migrazione:

ruby
Copia codice
add_index :list_items, [:list_id, :external_uid], unique: true
Model:

ruby
Copia codice
class ListItem < ApplicationRecord
  validates :slug, uniqueness: { scope: [:list_id, :parent_id] }
end
Editor: genera un UUID per ogni nodo e fallo persistere nel localStorage (non cambiarlo ai rename). Esporta:

json
Copia codice
{ "external_uid":"a2f1-...", "title":"...", "slug":"...", "children":[...] }
Controller (upsert):

ruby
Copia codice
def build_nodes(list, arr, parent)
  arr.each_with_index do |node, idx|
    finder =
      if node["external_uid"].present?
        list.list_items.find_or_initialize_by(external_uid: node["external_uid"])
      else
        list.list_items.new
      end

    finder.assign_attributes(
      parent: parent,
      title:  node["title"].to_s,
      slug:   (node["slug"].presence || node["title"].to_s.parameterize),
      nav_order: idx,
      data:   node.except("title","slug","children","external_uid")
    )
    finder.save!

    build_nodes(list, node["children"] || [], finder)
  end
end
Vantaggi: re-importi senza perdere gli ID DB e puoi fare diff/merge.
Nota: se cambi il parent, l’upsert la gestisce perché assegni parent ad ogni passaggio.

Quando ha senso tenere un ID lato client
Per gestire focus, shortcut, drag/drop, link temporanei fra nodi.

Ma non confonderlo con l’ID DB: chiamalo client_id o cid.

Consiglio pratico per te ora
Parti con Strategia 1 (replace-all): è la più veloce e robusta.

Nel JSON rinomina id → client_id e ignora lato Rails.

Quando GeneraImpresa richiederà import incrementale, aggiungiamo external_uid e passiamo alla Strategia 2.

Se vuoi, ti passo un mini-patch per l’editor in canvas che:

sostituisce data-id → data-cid

rimuove id dall’export (o lo rinomina in client_id)

opzione (commentata) per generare external_uid UUID.




