# 🟢 SCORPION TECH — VCF Contact Hub

---

## 📁 Structure du projet

```
vcf-contact-hub/
├── index.js              ← Serveur Express principal
├── package.json
├── .env.example          ← Copier en .env
├── models/
│   └── Contact.js        ← Modèle MongoDB (name, phone, date)
├── routes/
│   └── contacts.js       ← API REST (/status, /register, /download)
└── public/
    └── index.html        ← Frontend complet (bilingue EN/FR)
```

---

## ⚙️ Variables d'environnement

Copie `.env.example` → `.env` et remplis :

```env
MONGODB_URI=mongodb+srv://<user>:<password>@cluster0.xxxxx.mongodb.net/vcf_hub?retryWrites=true&w=majority
PORT=3000
MAX_CONTACTS=500
```

---

## 🚀 Déploiement sur Render + MongoDB Atlas

### ÉTAPE 1 — MongoDB Atlas (base de données gratuite)

1. Va sur **https://cloud.mongodb.com** → crée un compte gratuit
2. **New Project** → nom : `vcf-hub`
3. **Build a Database** → choisir **M0 (Free)**
4. Région : choisis la plus proche de toi
5. **Create** → attends 1-2 min
6. **Database Access** → Add new user :
   - Username : `vcf_admin`
   - Password : génère un mot de passe fort → note-le
   - Role : `readWriteAnyDatabase`
7. **Network Access** → Add IP Address → `0.0.0.0/0` *(autorise tout — nécessaire pour Render)*
8. **Clusters** → **Connect** → **Drivers** → copie la connection string :
   ```
   mongodb+srv://vcf_admin:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
   Remplace `<password>` par ton mot de passe et ajoute `/vcf_hub` avant le `?` :
   ```
   mongodb+srv://vcf_admin:TONMDP@cluster0.xxxxx.mongodb.net/vcf_hub?retryWrites=true&w=majority
   ```

---

### ÉTAPE 2 — Push sur GitHub

```bash
# Dans le dossier vcf-contact-hub/
git init
git add .
git commit -m "🚀 Initial commit - VCF Contact Hub"
git branch -M main
git remote add origin https://github.com/SCORPION/scorpion-vcf.git
git push -u origin main
```

---

### ÉTAPE 3 — Déploiement sur Render (gratuit)

1. Va sur **https://render.com** → connecte ton GitHub
2. **New** → **Web Service**
3. Sélectionne le repo `vcf-contact-hub`
4. Configure :
   | Champ | Valeur |
   |-------|--------|
   | Name | `vcf-contact-hub` |
   | Region | Frankfurt (EU) ou Oregon (US) |
   | Branch | `main` |
   | Runtime | **Node** |
   | Build Command | `npm install` |
   | Start Command | `node index.js` |
   | Plan | **Free** |

5. **Environment Variables** → clique "Add Environment Variable" :
   - `MONGODB_URI` = ta connection string complète
   - `MAX_CONTACTS` = `500`

6. **Create Web Service** → attend 2-3 minutes

7. ✅ Ton site est live sur `https://vcf-contact-2fkl.onrender.com`

---

## 🔌 API Endpoints

| Method | Route | Description |
|--------|-------|-------------|
| `GET` | `/api/status` | Compte total, slots restants, full ou non |
| `POST` | `/api/register` | Inscrire un contact `{ name, phone }` |
| `GET` | `/api/download` | Télécharge le VCF — **bloqué si < 500** |

### Exemple de réponse `/api/status`
```json
{
  "count": 247,
  "max": 500,
  "full": false,
  "slotsLeft": 253
}
```

### Exemple `/api/register`
```bash
curl -X POST https://ton-site.onrender.com/api/register \
  -H "Content-Type: application/json" \
  -d '{"name":"scorpion","phone":"+27 73 816 4314"}'
```

### Codes d'erreur
| Code | Signification |
|------|--------------|
| `MISSING_FIELDS` | Nom ou numéro manquant |
| `DUPLICATE_PHONE` | Numéro déjà inscrit |
| `LIST_FULL` | 500/500 atteint |
| `NOT_READY` | Download demandé avant 500 contacts |
| `TOO_MANY_REQUESTS` | Rate limit (5 inscriptions / 15min / IP) |

---

## 🛡️ Sécurité intégrée

- ✅ **Liste invisible** — aucun endpoint public n'expose les contacts
- ✅ **Download bloqué** côté serveur (pas juste frontend)
- ✅ **Rate limiting** — 5 inscriptions / 15min / IP (anti-spam)
- ✅ **Anti-doublon** — index unique MongoDB sur `phone`
- ✅ **Download rate limit** — 10 requêtes / min / IP
- ✅ **Validation** des champs côté serveur

---

## 💻 Développement local

```bash
# Clone / télécharge le projet
cd vcf-contact-hub

# Installe les dépendances
npm install

# Copie et configure .env
cp .env.example .env
# → édite .env avec ta MONGODB_URI

# Lance en mode dev (avec auto-reload)
npm run dev

# Ouvre http://localhost:3000
```

---

## 📱 Comment fonctionne le VCF ?

Quand les 500 contacts sont atteints, `/api/download` génère un fichier `.vcf` standard :

```
BEGIN:VCARD
VERSION:3.0
FN:scorpion
TEL;TYPE=CELL:+27 073 632 4314
END:VCARD
```

Ce format est reconnu nativement par :
- 📱 **Android** — import automatique dans les contacts
- 🍎 **iPhone (iOS)** — ouverture directe dans Contacts
- 💻 **WhatsApp** — tous les numéros deviennent cliquables

---

## 🔧 Personnalisation

| Paramètre | Fichier | Variable |
|-----------|---------|----------|
| Limite contacts | `.env` | `MAX_CONTACTS=500` |
| Port serveur | `.env` | `PORT=3000` |
| Lien GitHub | `public/index.html` | ligne ~130 |
| Lien WhatsApp Channel | `public/index.html` | ligne ~137 |

---

*Built with ❤️ by **SCORPION TECH *** **  · [WhatsApp Channel](https://whatsapp.com/channel/0029VbDNVNs002T3TGUozn0g)
