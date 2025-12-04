---
title: Gestire eventi ruoli journey cicli
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 16
---


# Taxbranch da vedere
module TaxbranchesHelper
  def taxbranch_label(node)
    node.description.presence || node.slug_label.presence || node.slug
  end

  def taxbranch_breadcrumb(node)
    trail = node.ancestors + [node]
    parts = []
    parts << link_to("Tassonomie", taxbranches_path)
    parts += trail.map { |n| link_to(taxbranch_label(n), taxbranch_path(n)) }
    safe_join(parts, content_tag(:span, " › ", class: "text-slate-400"))
  end
end


PosturaCorretta.org
 ├─ Stop al Dolore (Rosso)
 │   ├─ Articoli
 │   ├─ Video tematici
 │   └─ Pacchetto Primo Soccorso
 ├─ Benessere Integrato (Verde)
 │   ├─ Questionari salute
 │   ├─ Gruppi e consulenze
 │   └─ Kit salute
 ├─ Igiene Posturale (Blu)
 │   ├─ Mini-corso gratuito
 │   ├─ Corso completo
 │   └─ Percorso con tutor
 └─ Lavora con noi / Accademia
     ├─ Formazione operatori
     └─ Collaborazioni e affiliazioni
