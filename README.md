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

> ⚠️ **MODE TEMPORAIRE ACTIF** : la branche Gmail est actuellement **désactivée** dans le workflow (compte Gmail bloqué temporairement par Google). L'envoi se fait via **Resend** au lieu de Gmail. Voir la section [🔁 Mode dégradé Resend](#-mode-dégradé-resend-quand-gmail-est-bloqué) plus bas pour les détails et le retour à Gmail.

Phase 2 **activée** : le workflow lit normalement une **branche Gmail en parallèle** du flux RSS, intègre les newsletters au tri Claude, et les **archive automatiquement** après envoi de ta veille.

### Comment ça s'articule

```
  ⏰ 7h                    📰 RSS (25 sources) ─────┐
 trigger  ─────┬─>                                    ├─> Fusionner → Dédup → Claude → HTML → Envoyer
               └─> 📧 Gmail label "veille" ──────────┘                                            │
                                                                                                   ▼
                                                                          📁 Archiver les newsletters traitées
```

### Setup Gmail (à faire une fois)

**1. Créer le label `veille` dans Gmail**
- Gmail → menu de gauche → **Créer un libellé** → nomme-le `veille`

**2. Créer un filtre pour chaque newsletter à intégrer**

Pour chaque newsletter que tu veux inclure (Design Systems Weekly, The Rundown AI, Superhuman AI, Thiga, NoCode France, Supernova, etc.) :

1. Ouvre un email de la newsletter dans Gmail
2. Menu `⋮` (trois points en haut à droite de l'email) → **Filtrer les messages similaires**
3. Dans "De :", laisse l'adresse de l'expéditeur (ex : `newsletter@designsystems.surf`)
4. Clique **Créer un filtre**
5. Coche **Appliquer le libellé** → choisis `veille`
6. **Ne coche PAS** "Ignorer la boîte de réception" — il faut qu'ils arrivent en INBOX pour que le workflow les attrape
7. Optionnel : coche **Appliquer aussi ce filtre à X conversations correspondantes** pour tagger les anciennes
8. **Créer le filtre**

Répète pour chaque newsletter. Compte 30 secondes par source.

**3. Créer / vérifier la credential Gmail OAuth2**

Le workflow a besoin d'accès **lecture + modification** à ta boîte. Si tu as choisi SMTP pour l'envoi en Phase 1, il te faut maintenant **en plus** une credential **Gmail OAuth2** — c'est obligatoire, SMTP ne permet que d'envoyer.

👉 Le setup OAuth2 est un peu technique (projet Google Cloud, écran de consentement, ID client). C'est détaillé **pas à pas dans l'annexe en fin de document** : [Annexe — Configurer Gmail OAuth2](#annexe--configurer-gmail-oauth2).

Une fois la credential créée, sélectionne-la sur les deux nouveaux nœuds Gmail :
- **Gmail - Récupérer newsletters**
- **Gmail - Archiver newsletters**

### Comment ça fonctionne au quotidien

- **7h00** : le workflow lit toutes les newsletters non archivées avec le label `veille` des 7 derniers jours
- **Parsing** : chaque email devient un "article" avec `source = 📧 Nom de l'expéditeur`, titre = objet du mail, description = début du corps (1800 caractères)
- **Fusion** : RSS + Gmail sont mergés et passent ensemble au nœud Claude
- **Après envoi** : les newsletters traitées sont archivées (label `INBOX` retiré). Elles restent accessibles via le label `veille` si tu veux les relire.

### Points d'attention

- **Si l'étape Claude ou l'envoi plante** : les newsletters ne sont **pas** archivées. Elles seront ré-essayées au prochain run. C'est voulu.
- **Si un email fetché n'a pas pu être parsé proprement** : il est quand même archivé (on n'ajoute pas de complexité pour le rattraper — les doublons futurs sont rares).
- **Requête Gmail** : `label:veille in:inbox newer_than:7d` (éditable dans le nœud "Gmail - Récupérer newsletters" → champ "q")
- **Volume** : limite à 50 newsletters par exécution, largement suffisant

### Si tu veux un audit plus strict

Au lieu d'archiver (remove INBOX), tu peux ajouter un label `veille/traité` à la place. Ouvre le nœud **Gmail - Archiver newsletters** :
- Remplace `removeLabels` par `addLabels`
- Remplace `INBOX` par le nom/ID de ton label `veille/traité` (à créer au préalable)

Ça garde les mails en inbox mais évite de les retraiter (en ajustant aussi la requête de fetch).

---

# 🔁 Mode dégradé Resend (quand Gmail est bloqué)

> Mode actif tant que `maddworkflow@gmail.com` n'est pas débloqué par Google.

## Pourquoi

Google verrouille parfois les comptes Gmail récents qui font de l'OAuth — un déblocage prend généralement 24-72h, mais ça peut traîner. En attendant, on remplace Gmail par **Resend**, un service email transactionnel gratuit (3000 mails/mois) qui n'a aucun lien avec Gmail.

## Ce que ça change dans le workflow

```
Avant (Gmail) :
  ⏰ → RSS + Gmail Récup → Fusion → Dédup → Claude → HTML → Gmail Send → Archive
                                                                           Gmail
Maintenant (Resend) :
  ⏰ → RSS ──────────────────────► Dédup → Claude → HTML → Resend HTTP

  (les nœuds Gmail Récup/Parser/Fusion/IDs/Archiver restent dans le canvas
   mais déconnectés — easy à reconnecter quand le compte revient)
```

## Setup Resend (10 min, à faire une fois)

### 1. Créer le compte Resend

1. Va sur [resend.com/signup](https://resend.com/signup)
2. Inscris-toi avec **`fuzier.maddlyn@gmail.com`** (l'email où tu veux recevoir la veille)
3. Vérifie ton email (clic sur le lien reçu)

> 🔑 **Important** : sans configurer un domaine custom, Resend gratuit n'autorise l'envoi **que vers l'email de ton compte Resend**. C'est pour ça qu'on utilise `fuzier.maddlyn@gmail.com` à la fois comme compte Resend et comme destinataire.

### 2. Récupérer la clé API

1. Dans Resend → menu de gauche → **API Keys**
2. **Create API Key**
3. Nom : `n8n veille` · Permission : **Full access** · Domain : *None* (ou sélectionne par défaut)
4. **Add**
5. Copie la clé qui commence par `re_...` → garde-la sous le coude (Resend ne te la remontrera pas)

### 3. Créer la credential dans n8n

1. n8n → **Credentials** → **Add Credential**
2. Cherche **Header Auth**
3. Remplis :
   - **Name** : `Resend API key` *(exactement, pour matcher le workflow)*
   - **Header Name** : `Authorization`
   - **Header Value** : `Bearer re_XXXXXXXXXX` *(remplace par ta clé, garde le `Bearer ` devant avec un espace)*
4. **Save**

### 4. Re-importer le workflow

Comme d'habitude :
1. Télécharge la dernière version de `n8n/workflow.json` depuis le repo
2. Dans n8n, supprime ou renomme l'ancien workflow → **Add workflow** → **Import from File**
3. Vérifie que :
   - Le nœud **Envoyer l'email** est bien un HTTP Request (pas Gmail)
   - Sa credential `Resend API key` est auto-sélectionnée
   - Le nœud **Claude - Curation** a sa credential `Anthropic API key` auto-sélectionnée
4. **Save** (Ctrl+S)
5. **Execute Workflow** → tu devrais recevoir la veille sur `fuzier.maddlyn@gmail.com` en quelques secondes

### Détail du nœud Resend

Le nœud HTTP Request envoie ce body à `https://api.resend.com/emails` :

```json
{
  "from": "Jean-Pierre <onboarding@resend.dev>",
  "to": "fuzier.maddlyn@gmail.com",
  "subject": "Ta veille du ...",
  "html": "..."
}
```

L'expéditeur affiché sera **Jean-Pierre** (modifiable dans le `jsonBody` du nœud).

## Revenir à Gmail quand le compte est débloqué

Quand `maddworkflow@gmail.com` refonctionne :

1. Vérifie que ta credential **Gmail OAuth2** dans n8n est toujours valide (ré-authentifie si besoin)
2. Reconnecte les fils dans le canvas n8n :
   - **Chaque matin 7h00** → **Gmail - Récupérer newsletters** *(en plus de Liste des sources)*
   - **Filtrer 24h + nettoyer** → **Fusionner RSS + Gmail** *(input 0)*
   - **Fusionner RSS + Gmail** → **Dédupliquer** *(et supprimer la connexion directe Filtrer→Dédup)*
   - **Envoyer l'email** → **IDs Gmail à archiver**
3. Re-transforme **Envoyer l'email** d'HTTP Request vers Gmail node :
   - Plus simple : supprime le nœud HTTP Request, ajoute un nouveau nœud **Gmail Send Email** à la place, configure-le avec ta credential Gmail OAuth2 et les expressions `={{ $json.sujet }}` / `={{ $json.html }}`
4. **Save**, teste, réactive le cron

> 💡 Si tu veux je te referai un commit qui te file le workflow.json en mode "tout Gmail" prêt à importer le moment venu.

## Limitations du mode dégradé

- ❌ Pas de lecture des newsletters via label `veille` (les newsletters reçues pendant ce temps ne seront pas dans la veille)
- ❌ Pas d'archivage automatique des emails
- ✅ Tout le reste fonctionne (RSS, Claude, email)
- ✅ La veille arrive dans `fuzier.maddlyn@gmail.com` chaque matin à 7h

---

# 🔐 Annexe — Configurer Gmail OAuth2

Procédure complète pour créer une credential Gmail OAuth2 dans n8n. Compte **10-15 minutes** la première fois.

## Le principe

Une credential "Gmail OAuth2" c'est une **autorisation signée** qui dit à Google : "Ce n8n a le droit de lire et modifier les mails de `maddworkflow@gmail.com`". Google exige que tu crées une "app" sur Google Cloud Console pour générer cette autorisation — c'est gratuit.

## Étapes

### 1. Créer un projet Google Cloud

1. Va sur [console.cloud.google.com](https://console.cloud.google.com) (connecte-toi avec `maddworkflow@gmail.com`)
2. En haut à gauche à côté du logo → sélecteur de projet → **Nouveau projet**
3. Nom : `n8n-veille` → **Créer**
4. Attends 10-20 secondes, puis sélectionne le projet

### 2. Activer l'API Gmail

1. Menu burger (☰) → **APIs et services** → **Bibliothèque**
2. Cherche `Gmail API` → clique → **Activer**

### 3. Configurer l'écran de consentement

1. Menu burger → **APIs et services** → **Écran de consentement OAuth**
2. Type d'utilisateur : **Externe** → **Créer**
3. Remplis le minimum :
   - **Nom de l'application** : `n8n veille`
   - **Email d'assistance utilisateur** : `maddworkflow@gmail.com`
   - **Coordonnées du développeur** : `maddworkflow@gmail.com`
4. **Enregistrer et continuer**
5. **Niveaux d'accès** → **Ajouter ou supprimer des champs d'application** → coche :
   - `.../auth/gmail.modify` (lire, envoyer, modifier — **sans** supprimer)
   - `.../auth/gmail.labels`
6. **Mettre à jour** → **Enregistrer et continuer**
7. **Utilisateurs test** → **Ajouter des utilisateurs** → ajoute `maddworkflow@gmail.com`
   > ⚠️ **Étape critique** : sans ça, tu auras une erreur `access_denied` à l'authentification.
8. **Enregistrer et continuer** → **Retour au tableau de bord**

> 🔒 L'app reste en **mode test** — aucun souci, ça marche très bien pour un usage perso. Pas besoin de vérification Google.

### 4. Créer les identifiants OAuth 2.0

1. Menu burger → **APIs et services** → **Identifiants**
2. En haut → **Créer des identifiants** → **ID client OAuth**
3. **Type d'application** : `Application Web`
4. **Nom** : `n8n veille client`
5. **URI de redirection autorisés** → laisse vide pour l'instant (on reviendra)
6. **Créer**
7. Une modale affiche :
   - **ID client** (format `xxxxxxxx.apps.googleusercontent.com`)
   - **Code secret du client** (format `GOCSPX-...`)
8. **Copie les deux** dans un bloc-notes

### 5. Créer la credential dans n8n

1. Dans n8n → **Credentials** → **Add Credential**
2. Cherche **Gmail OAuth2 API**
3. Colle :
   - **Client ID** = l'ID client Google
   - **Client Secret** = le code secret Google
4. n8n affiche une **OAuth Redirect URL** en bas (type `https://ton-n8n.domaine.fr/rest/oauth2-credential/callback`) → **copie-la**

### 6. Ajouter l'URL de redirection côté Google

1. Retourne sur [Google Cloud Console → Identifiants](https://console.cloud.google.com/apis/credentials)
2. Clique sur ton **ID client OAuth** (icône crayon pour éditer)
3. **URI de redirection autorisés** → **Ajouter un URI** → colle l'URL copiée depuis n8n
4. **Enregistrer**

### 7. Lier le compte dans n8n

1. Retourne dans n8n sur ta credential Gmail OAuth2
2. Clique **Sign in with Google**
3. Popup Google → choisis `maddworkflow@gmail.com`
4. ⚠️ Écran **"Google n'a pas vérifié cette application"** → **Paramètres avancés** → **Accéder à n8n veille (non sécurisé)** → accepte les permissions
5. La popup se ferme, n8n affiche ✅ **Account connected**
6. **Save**

C'est prêt. Tu peux maintenant sélectionner cette credential dans les 3 nœuds Gmail du workflow.

## Pièges classiques

| Erreur | Cause | Solution |
|---|---|---|
| `Error 403: access_denied` | Compte pas dans "Utilisateurs test" | Étape 3.7 : ajoute `maddworkflow@gmail.com` |
| `redirect_uri_mismatch` | URL de redirection pas copiée exactement | Étape 6 : recopie depuis n8n, attention aux `/` finaux |
| `This app isn't verified` | Normal en mode test | Clique "Paramètres avancés" → accéder quand même |
| `invalid_grant` après 7 jours | Refresh token expiré en mode test | Re-clique "Sign in with Google" dans la credential |

## Le souci du refresh token de 7 jours

En **mode test**, les refresh tokens Google expirent après **7 jours** → il faut re-authentifier la credential chaque semaine, sinon le workflow plante.

Deux options pour éviter ça :

**Option A — Publier l'app** (recommandé pour usage perso)
1. Google Cloud Console → **Écran de consentement OAuth** → **Publier l'application**
2. Google te demande une vérification uniquement si tu dépasses 100 utilisateurs ou si tu utilises des scopes "restricted"
3. Pour notre cas (`gmail.modify` + `gmail.labels`, mode perso), tu peux publier sans vérification → refresh token valide sans expiration

**Option B — Rester en test** et re-connecter toutes les semaines (pénible).

Je conseille l'Option A dès que tu as validé que le workflow tourne bien.

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
