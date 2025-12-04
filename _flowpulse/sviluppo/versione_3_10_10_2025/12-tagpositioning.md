---
title: Tag Positioning
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 12
---


# Tag positioning


bin/rails g scaffold TagPositioning \
  taxbranch:references \
  post:references \
  name:string \
  category:string \
  metadata:jsonb

  # db/migrate/xxxx_create_tag_positionings.rb
class CreateTagPositionings < ActiveRecord::Migration[7.1]
  def change
    create_table :tag_positionings do |t|
      t.references :taxbranch, null: false, foreign_key: true
      t.references :post,      null: true,  foreign_key: true
      t.string :name,          null: false        # il “termine” (es. “dolori cervicali”)
      t.string :category,      null: false        # libera: “problema”, “target”, …
      t.jsonb  :metadata,      null: false, default: {}

      t.timestamps
    end

    add_index :tag_positionings, [:taxbranch_id, :category]
    add_index :tag_positionings, [:taxbranch_id, :category, :name], unique: true
    add_index :tag_positionings, :metadata, using: :gin
  end
end






1) Flag pubblico + nuvola “positioning”
Migration
bin/rails g migration AddPositioningTagPublicToTaxbranches positioning_tag_public:boolean

# db/migrate/XXXXXXXXXX_add_positioning_tag_public_to_taxbranches.rb
class AddPositioningTagPublicToTaxbranches < ActiveRecord::Migration[7.1]
  def change
    add_column :taxbranches, :positioning_tag_public, :boolean, default: false, null: false
    add_index  :taxbranches, :positioning_tag_public
  end
end

Model
# app/models/taxbranch.rb
class Taxbranch < ApplicationRecord
  has_ancestry
  acts_as_list scope: [:ancestry]

  has_many :tag_positionings, dependent: :destroy
  has_many :posts, dependent: :nullify

  scope :ordered,        -> { order(:position, :slug_label) }
  scope :home_nav,       -> { where(home_nav: true).ordered }
  scope :positioning_on, -> { where(positioning_tag_public: true) }

  # Vista etichetta leggibile (dato che hai la colonna slug_label)
  def display_label
    slug_label.presence || slug.to_s.titleize
  end

  # Conta i tag per nuvola (public o admin)
  def positioning_items
    counts = tag_positionings.group(:name, :category).count
    counts.map { |(name, cat), n| { text: name, count: n, cat: cat } }
  end
end

Mostrare la nuvola nella show pubblica del ramo (solo se pubblico)

Nella pagina che mostra un ramo (es. TreepagesController#show o la tua page equivalente), sotto al titolo:

<% if @taxbranch.positioning_tag_public? %>
  <section class="mt-10">
    <h2 class="text-xl font-semibold mb-2">Concetti collegati</h2>

    <div class="flex flex-wrap gap-2 mb-4">
      <% %w[all problema target keyword messaggio servizio].each do |cat| %>
        <button data-cat="<%= cat %>" class="px-3 py-1 rounded-full bg-slate-100 text-slate-700 text-sm">
          <%= cat == "all" ? "Tutti" : cat.capitalize %>
        </button>
      <% end %>
    </div>

    <div id="cloudWrap" class="bg-white rounded-2xl shadow p-4">
      <canvas id="wordCloud"></canvas>
    </div>
  </section>

  <script>
    const items = <%= raw @taxbranch.positioning_items.to_json %>;
  </script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-chart-wordcloud"></script>
  <script>
    function normalizeWeights(arr, minFont=12, maxFont=64){
      const cs = arr.map(a=>a.count), min=Math.min(...cs), max=Math.max(...cs);
      const scale=v => (max===min) ? (minFont+maxFont)/2 : Math.round(minFont + ((v-min)/(max-min))*(maxFont-minFont));
      return arr.map(a => ({...a, weight: scale(a.count)}));
    }
    const catColor = { problema:"#ef4444", target:"#d97706", keyword:"#10b981", messaggio:"#6366f1", servizio:"#0ea5e9" };
    let currentCat="all", chart;

    function renderCloud(){
      const filtered = (currentCat==="all") ? items : items.filter(i => i.cat===currentCat);
      const dataArr = normalizeWeights(filtered);
      const labels  = dataArr.map(d=>d.text);
      const data    = dataArr.map(d=>d.weight);

      const wrap = document.getElementById('cloudWrap');
      const canvas = document.getElementById('wordCloud');
      canvas.width  = wrap.clientWidth;
      canvas.height = Math.max(320, Math.round(wrap.clientWidth*0.55));

      if(chart) chart.destroy();
      chart = new Chart(canvas.getContext('2d'), {
        type: 'wordCloud',
        data: { labels, datasets: [{ data, color: (ctx)=>catColor[dataArr[ctx.dataIndex]?.cat] || "#334155" }] },
        options: { responsive:false, plugins:{legend:{display:false}}, elements:{ word:{ rotate:0, padding:3, spiral:"rectangular" } } }
      });
    }

    document.querySelectorAll('button[data-cat]').forEach(btn=>{
      btn.addEventListener('click', ()=>{
        currentCat = btn.dataset.cat;
        renderCloud();
      });
    });
    window.addEventListener('resize', ()=>{ clearTimeout(window.__wc); window.__wc=setTimeout(renderCloud,120); });
    renderCloud();
  </script>
<% end %>



User.create!(
  email_address: "markpostura@gmail.com",
  password: "ChiComin67?",
  password_confirmation: "ChiComin67?",
  superadmin: true
)


------- Versione nuova ------ 

# app/models/tag_positioning.rb
class TagPositioning < ApplicationRecord
  belongs_to :post
  has_one :taxbranch, through: :post
  belongs_to :lead, optional: true

  validates :name, :category, presence: true

  # Evita doppioni sullo stesso post
  validates :name, uniqueness: { scope: [:post_id, :category] }

  # Conteggio utile per nuvole o ranking
  scope :by_category, ->(cat) { where(category: cat) }
  scope :for_branch,  ->(tb)  { joins(:post).where(posts: { taxbranch_id: tb.id }) }

  def self.cloud_for(branch, category = nil)
    query = joins(:post).where(posts: { taxbranch_id: branch.id })
    query = query.where(category: category) if category
    query.group(:name).order("count_all desc").count
  end
end
🌥️ Come costruire la “nuvola di tag”
Esempio in un helper:

ruby
Copia codice
# app/helpers/tags_helper.rb
module TagsHelper
  def render_tag_cloud(branch, category: nil)
    tags = TagPositioning.cloud_for(branch, category)
    return content_tag(:p, "Nessun tag disponibile") if tags.empty?

    max = tags.values.max
    min = tags.values.min
    range = [max - min, 1].max

    safe_join(
      tags.map do |name, count|
        weight = ((count - min) / range.to_f * 4).round + 1
        link_to(
          name,
          "#", # in futuro potrai mettere il link SEO
          class: "text-slate-600 hover:text-blue-700 text-sm font-medium tag-size-#{weight}"
        )
      end,
      " "
    )
  end
end
E CSS/Tailwind per scalare i pesi:

html
Copia codice
<style>
  .tag-size-1 { font-size: 0.8rem; opacity: 0.6; }
  .tag-size-2 { font-size: 0.9rem; opacity: 0.7; }
  .tag-size-3 { font-size: 1rem;   opacity: 0.8; }
  .tag-size-4 { font-size: 1.1rem; opacity: 0.9; }
  .tag-size-5 { font-size: 1.2rem; opacity: 1;   }
</style>
E nel template:

erb
Copia codice
<!-- app/views/posts/index.html.erb -->
<div class="my-6">
  <h2 class="text-lg font-semibold text-slate-800 mb-2">Temi più frequenti</h2>
  <%= render_tag_cloud(@taxbranch) %>
</div>
