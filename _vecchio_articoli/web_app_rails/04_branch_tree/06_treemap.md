---
layout: default
title: Tree Map
parent: Branch Tree
grand_parent:  Web app rails
nav_fold: true
has_children: true
nav_order: 6
---

# tree map

Potrebbe essere usato sul lato manager o utente in associazione con un 1impegno, per visualizzare gli avanzamenti.

Insiema alla vista a rete che è da considerare se creare separatamente secondo l'articolo:

- Vettoriale sensazione
- Albero categorie
- rete 
- tag



# nuovo

ho un app rails con flowpulse_branch che ha un @branch.parent_id che genera un albero vorrei caricare l'albero magari trasformandolo in json e mi servirebbe un html un breadcrumb per vedere i livelli di profondita in branches/:id/treemap treemap.html.erb controller @branch.branch_root
ChatGPT ha detto:
Perfetto, sei su Rails e stai usando un modello chiamato Branch, che ha un campo parent_id per costruire un albero gerarchico. Vuoi:

Caricare i dati di quell’albero in formato JSON.

Visualizzarli come treemap.

Includere un breadcrumb per navigare i livelli.

Il tutto nella pagina branches/:id/treemap, in treemap.html.erb.

👨‍💻 Struttura suggerita
1. Modello Rails (branch.rb)
Assicurati di avere queste associazioni:

ruby
Copia
Modifica
class Branch < ApplicationRecord
  belongs_to :parent, class_name: 'Branch', optional: true
  has_many :children, class_name: 'Branch', foreign_key: 'parent_id'

  def subtree_json
    {
      id: id,
      name: name,
      tempo_tot: tempo_tot || 1, # fallback se mancante
      color: color,
      children: children.map(&:subtree_json)
    }
  end

  def breadcrumb_path
    parent ? parent.breadcrumb_path + [self] : [self]
  end

  def root
    parent ? parent.root : self
  end
end
2. Controller (branches_controller.rb)
ruby
Copia
Modifica
def treemap
  @branch = Branch.find(params[:id])
  @tree_json = @branch.root.subtree_json.to_json
  @breadcrumb = @branch.breadcrumb_path
end
3. View: treemap.html.erb
erb
Copia
Modifica
<h2>Treemap: <%= @branch.name %></h2>

<!-- Breadcrumb -->
<nav style="margin-bottom: 1em;">
  <% @breadcrumb.each_with_index do |b, i| %>
    <% if i < @breadcrumb.length - 1 %>
      <%= link_to b.name, treemap_branch_path(b) %> &raquo;
    <% else %>
      <strong><%= b.name %></strong>
    <% end %>
  <% end %>
</nav>

<!-- Treemap SVG -->
<svg id="treemap" width="100%" height="600"></svg>

<script src="https://d3js.org/d3.v7.min.js"></script>
<script>
  const data = <%= raw @tree_json %>;

  const width = document.getElementById('treemap').clientWidth;
  const height = 600;

  const svg = d3.select("#treemap");
  const treemap = d3.treemap().size([width, height]).paddingInner(2);

  const root = d3.hierarchy(data)
    .sum(d => d.tempo_tot || 1)
    .sort((a, b) => b.value - a.value);

  treemap(root);

  const nodes = svg.selectAll("g")
    .data(root.descendants())
    .join("g")
    .attr("transform", d => `translate(${d.x0},${d.y0})`);

  nodes.append("rect")
    .attr("width", d => d.x1 - d.x0)
    .attr("height", d => d.y1 - d.y0)
    .attr("fill", d => d.data.color || d3.interpolateCool(d.depth * 0.2))
    .attr("stroke", "#fff");

  nodes.append("text")
    .attr("x", 4)
    .attr("y", 14)
    .attr("fill", "white")
    .text(d => {
      const w = d.x1 - d.x0;
      return w > 60 ? d.data.name : "";
    });
</script>
✅ Risultato
Treemap completa con tutta la gerarchia visibile.

Breadcrumb cliccabile per salire di livello.

Colori gestibili da attributo color su Branch o interpolazione automatica.

🔄 Prossimi Step (facoltativi)
Aggiungere drilldown con onclick per navigare solo un sotto-ramo.

Tooltip su hover per vedere tempo_tot.

Bottone "torna alla radice".

Caricamento AJAX dell’albero per performance.

Vuoi che ti aiuti a rendere il tutto AJAX o trasformarlo in componente Vue/React?
