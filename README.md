# Veille quotidienne — workflow n8n + Claude

> Un mail chaque matin à 7h30 avec une sélection curée d'articles sur le product design, la discovery, le produit et l'IA. Lecture : 5 min café en main ☕

---

## Comment ça marche (résumé)

```
  ⏰ 7h00                   📰 Collecte              🧠 Claude trie              📬 7h30
 n8n se réveille   →    15 flux RSS lus     →    score + résume    →    email livré
```

Trois acteurs :

1. **n8n** (sur ton serveur local) : le chef d'orchestre. Il se réveille, va chercher les articles, prépare tout.
2. **Claude** (via l'API Anthropic) : le journaliste. Il lit les 40-80 articles collectés, garde les bons, les résume en 2 phrases.
3. **Gmail** : livre le mail final dans ta boîte.

---

## Ce qu'il y a dans ce repo

```
veille/
├── README.md                    ← tu es ici
├── sources.md                   ← liste des sources RSS commentée
├── n8n/
│   └── workflow.json            ← à importer dans n8n
├── prompts/
│   └── system-prompt.md         ← les instructions données à Claude
└── templates/
    ├── email.html               ← la maquette de l'email
    └── category-block.html      ← les blocs par catégorie
```

---

# 🛠️ Installation pas à pas

Tout se fait en **4 étapes**. Compte 30-45 minutes la première fois.

## Étape 1 — Récupérer une clé API Claude

Pour que n8n puisse "parler" à Claude, il faut une clé (c'est comme un badge d'accès).

1. Va sur [console.anthropic.com](https://console.anthropic.com)
2. Crée un compte si tu n'en as pas
3. Ajoute un moyen de paiement (onglet **Billing**). Mets **10 $ de crédit** pour commencer — ça te durera plusieurs mois vu le volume du workflow.
4. Va dans **API Keys** → **Create Key**
5. Donne-lui un nom genre "n8n veille"
6. **Copie la clé** qui commence par `sk-ant-...` et **garde-la au chaud dans un bloc-notes** — elle ne sera plus affichée après.

> 💰 **Coût estimé** : ~0,03 € par exécution quotidienne = **~1 €/mois**. Rassurant.

---

## Étape 2 — Préparer Gmail pour l'envoi

On va permettre à n8n d'envoyer des mails **depuis** ton compte `maddworkflow@gmail.com`.

### 2a — Activer la double authentification (si pas déjà fait)

1. [myaccount.google.com/security](https://myaccount.google.com/security)
2. Active la **validation en 2 étapes**

### 2b — Créer un "App Password"

1. Va sur [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
2. Crée un mot de passe d'application pour "n8n veille"
3. **Copie les 16 caractères** générés (type `abcd efgh ijkl mnop`)

> 🔑 Cet app password n'est PAS ton mot de passe Gmail. C'est une clé dédiée à n8n, révocable à tout moment.

---

## Étape 3 — Configurer n8n

### 3a — Créer la credential Claude

1. Ouvre ton n8n
2. Menu de gauche → **Credentials** → **Add Credential**
3. Cherche et choisis **Header Auth**
4. Remplis :
   - **Name** : `Anthropic API key`
   - **Header name** : `x-api-key`
   - **Header value** : ta clé Claude `sk-ant-...`
5. **Save**

### 3b — Créer la credential Gmail

Deux options, choisis la plus simple selon ton n8n :

**Option A (plus simple) — SMTP :**

1. **Credentials** → **Add Credential** → **SMTP**
2. Remplis :
   - **User** : `maddworkflow@gmail.com`
   - **Password** : l'App Password (16 caractères, sans les espaces)
   - **Host** : `smtp.gmail.com`
   - **Port** : `465`
   - **SSL/TLS** : activé
3. **Save**

> ⚠️ Si tu choisis SMTP, il faudra remplacer le nœud "Envoyer l'email" (actuellement Gmail OAuth) par un nœud **Send Email** après import. Voir Étape 3d.

**Option B — Gmail OAuth2 :**

1. **Credentials** → **Add Credential** → **Gmail OAuth2**
2. Clique **Sign in with Google** et autorise n8n
3. **Save**

> Option B plus élégante mais demande parfois de configurer un projet Google Cloud. Commence par A si tu débutes.

### 3c — Importer le workflow

1. Dans n8n, menu en haut à droite → **Import from File** (ou copier/coller JSON)
2. Sélectionne `n8n/workflow.json` de ce repo
3. Le workflow apparaît

### 3d — Brancher les credentials sur le workflow

1. Clique sur le nœud **Claude - Curation**
2. Dans "Credential for Header Auth" → choisis `Anthropic API key`
3. **Save**

Puis :

4. Clique sur le nœud **Envoyer l'email**
5. Si tu as choisi **SMTP (Option A)** : supprime ce nœud, remplace par un nœud **Send Email**, connecte-le au nœud précédent, configure :
   - **From** : `maddworkflow@gmail.com`
   - **To** : `maddworkflow@gmail.com`
   - **Subject** : `={{ $json.sujet }}`
   - **HTML** : `={{ $json.html }}`
   - **Credential** : ton SMTP Gmail
6. Si tu as choisi **Gmail OAuth2 (Option B)** : sélectionne ta credential dans le nœud existant
7. **Save**

---

## Étape 4 — Tester

1. Dans le workflow, en haut à droite → clique **Execute Workflow**
2. Regarde chaque nœud s'exécuter vert (ou rouge en cas d'erreur)
3. Vérifie ta boîte Gmail : le mail doit arriver dans la minute

### Si ça plante

- **Nœud RSS rouge** : une source a un flux cassé. Clique sur le nœud, regarde la source fautive, retire-la de la liste.
- **Nœud Claude rouge** : vérifie la clé API et que tu as du crédit sur ton compte Anthropic.
- **Nœud Email rouge** : vérifie ton App Password Gmail / ta credential OAuth.
- **JSON parse error dans "Construire l'email"** : Claude a renvoyé autre chose que du JSON. Rare, mais ça peut arriver. Relance, ça repasse.

### Activer le cron

Quand le test passe :
1. En haut à droite du workflow, passe le toggle **Inactive → Active**
2. À partir de demain 7h, tu recevras le mail automatiquement

---

# 🎛️ Personnalisation

## Ajouter une source

1. Ouvre `sources.md` pour voir comment trouver un flux RSS
2. Dans n8n, ouvre le nœud **Liste des sources**
3. Ajoute un objet `{"nom": "Nom affiché", "url": "https://exemple.com/feed"}` dans le tableau
4. **Save**, relance un test

## Changer l'heure d'envoi

1. Ouvre le nœud **Chaque matin 7h00**
2. Modifie l'expression cron :
   - `0 7 * * *` = 7h00 tous les jours
   - `30 6 * * *` = 6h30
   - `0 7 * * 1-5` = 7h uniquement en semaine

## Ajuster le ton / la sévérité

Ouvre le nœud **Claude - Curation**, dans le champ "JSON Body" tu peux éditer le `system` prompt. Voir `prompts/system-prompt.md` pour la version commentée.

Leviers rapides :
- **Plus sévère** : remplace `score >= 6` par `score >= 7` dans le prompt
- **Plus FR** : ajoute au prompt "Priorise les sources françaises quand elles sont pertinentes"

## Passer en Phase 2 (lire les newsletters Gmail sans RSS)

Certaines de tes sources (UX Pilot, The Product Crew, Tech Trash) n'ont pas de RSS. En Phase 2, on ajoutera un nœud **Gmail → Get Many Messages** filtré sur un label, pour intégrer ces emails directement. Dis-moi quand tu veux qu'on s'y mette.

---

# 🤔 Questions fréquentes

**Est-ce que ça marche si mon serveur n8n est éteint la nuit ?**
Oui tant qu'il tourne à 7h pile. Si le serveur est éteint au moment du cron, l'exécution est sautée (pas de rattrapage).

**Je peux lire la veille sur mon téléphone ?**
Oui, l'email est en HTML responsive, il s'affiche bien sur mobile.

**Claude "voit" tout mon contenu, c'est safe ?**
Les articles envoyés à l'API Anthropic ne sont pas utilisés pour entraîner les modèles (politique de confidentialité API). Et ce sont des articles publics de toute façon.

**Je peux partager la veille avec un collègue ?**
Oui, ajoute son adresse dans le champ "To" du nœud email, séparée par une virgule. Ou mieux : dupliquer le workflow pour que chacun ait son profil d'intérêt dans le prompt.

---

Bon café et bonne veille ☕
