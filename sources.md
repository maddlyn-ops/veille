# Sources de veille — sélection curée

Cette liste est le **point de départ** pour ton workflow. Tu pourras ajouter / retirer des sources au fil de l'eau dans le nœud "Sources RSS" de n8n.

Elle mélange :
- tes sources actuelles (Eric Djavid, UX Pilot, The Product Crew, Figma Plugin Weekly, The Ticket)
- des références incontournables en product design, discovery et IA
- un mix FR / EN, avec un léger biais vers l'EN (c'est là que le volume est)

---

## Tes sources actuelles

| Source | URL RSS (à vérifier après inscription) | Note |
|---|---|---|
| Eric Djavid | `https://ericdjavid.substack.com/feed` | Newsletter FR product/design |
| UX Pilot | *via Gmail* | Newsletter — pas de RSS public, on lira depuis Gmail en Phase 2 |
| The Product Crew | *via Gmail* | Idem |
| Figma Plugin Weekly | `https://figmapluginweekly.substack.com/feed` | RSS probable, à confirmer |
| The Ticket (Intercom) | `https://www.intercom.com/blog/feed/` | Flux du blog Intercom |

> 💡 Certaines newsletters n'ont pas de flux RSS. En **Phase 2** on ajoutera un nœud qui lit ta boîte Gmail et intègre ces newsletters automatiquement. Pour le MVP, on commence avec les RSS.

---

## Product Design & UX

| Source | URL RSS | Pourquoi |
|---|---|---|
| Nielsen Norman Group | `https://www.nngroup.com/feed/articles/` | La référence recherche UX |
| Smashing Magazine | `https://www.smashingmagazine.com/feed/` | Design & front-end, très concret |
| UX Collective | `https://uxdesign.cc/feed` | Agrégateur Medium, gros volume |
| Josh W. Comeau | `https://www.joshwcomeau.com/rss.xml` | Design d'interaction haut niveau |
| Growth.Design | `https://growth.design/feed` | Case studies design produit |
| Jakob Nielsen (Substack) | `https://jakobnielsenphd.substack.com/feed` | Le "vrai" Jakob, post-NNG |

## Discovery & Product Management

| Source | URL RSS | Pourquoi |
|---|---|---|
| Lenny's Newsletter | `https://www.lennysnewsletter.com/feed` | La bible du product |
| Product Talk (Teresa Torres) | `https://www.producttalk.org/feed/` | **LA** ref en continuous discovery |
| SVPG (Marty Cagan) | `https://www.svpg.com/feed/` | Stratégie produit |
| Reforge | `https://www.reforge.com/blog/rss.xml` | Frameworks produit avancés |
| First Round Review | `https://review.firstround.com/rss` | Articles de fond |
| Mind the Product | `https://www.mindtheproduct.com/feed/` | Communauté PM |

## IA & Automatisation

| Source | URL RSS | Pourquoi |
|---|---|---|
| Simon Willison | `https://simonwillison.net/atom/everything/` | Le meilleur observateur LLM |
| Anthropic News | `https://www.anthropic.com/news/rss.xml` | Source officielle Claude |
| Every.to | `https://every.to/feed.xml` | IA appliquée au produit |
| The Batch (Andrew Ng) | `https://www.deeplearning.ai/the-batch/feed/` | Hebdo IA vulgarisée |
| Ben's Bites | `https://bensbites.beehiiv.com/feed` | IA quotidien, grand public |
| Stratechery (free posts) | `https://stratechery.com/feed/` | Analyse stratégique tech |

## Sources françaises

| Source | URL RSS | Pourquoi |
|---|---|---|
| Snowball | `https://snowball.substack.com/feed` | Tech/produit FR |
| Design Brief (Dan Mall) | `https://danmall.com/feed/` | Design ops |
| Tech Trash | *via Gmail* | Satirique tech FR |

---

## Comment ajouter une source plus tard

1. Tu trouves une newsletter / blog qui te plaît
2. Tu regardes si elle a un RSS : ajoute `/feed`, `/rss`, `/feed.xml` à l'URL du site, ou cherche "rss" dans le pied de page
3. Si pas de RSS : utilise [kill-the-newsletter.com](https://kill-the-newsletter.com) qui convertit une newsletter email en RSS (gratuit)
4. Ajoute l'URL dans le nœud "Sources RSS" de n8n
5. Relance le workflow en mode "Test" pour vérifier

## Volume attendu

Avec cette liste (~20 sources actives), compte **40 à 80 articles bruts par jour**, dont Claude te gardera **10 à 15 pépites** après filtrage.
