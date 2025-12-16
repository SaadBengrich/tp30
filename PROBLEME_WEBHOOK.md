# 🔴 Problème : Webhook GitHub ne Déclenche pas Jenkins

## 📊 Diagnostic Effectué

### ✅ Ce qui Fonctionne
- Git push vers GitHub : **OK** ✅
- Jenkins tourne : **OK** ✅  
- Plugins GitHub installés : **OK** ✅
- Webhook GitHub configuré : **OK** ✅

### ❌ Ce qui Ne Fonctionne PAS
- **Aucune requête POST reçue par Jenkins**
- **Aucune trace de webhook dans les logs Jenkins**
- **Le build ne se déclenche pas automatiquement**

---

## 🎯 Causes Possibles

### 1️⃣ Build Trigger NON Activé dans le Job ⚠️ **CAUSE PRINCIPALE**

**✅ SOLUTION CONFIRMÉE** :

Le webhook GitHub arrive bien à Jenkins (visible dans Ngrok), mais Jenkins ne déclenche pas le build car **le Build Trigger n'est pas activé**.

**Actions à faire IMMÉDIATEMENT** :

1. Ouvrez Jenkins : **http://localhost:8080**
2. Cliquez sur le nom de votre job (ex: **"my project"** ou **"tp30-pipeline"**)
3. Cliquez sur **"Configure"** (menu gauche)
4. Scrollez jusqu'à la section **"Build Triggers"**
5. ✅ **Cochez** : **"GitHub hook trigger for GITScm polling"**
6. Cliquez sur **"Save"** en bas de la page

⚠️ **CRITIQUE** : Sans cette case cochée, Jenkins **IGNORE** tous les webhooks GitHub !

**Note** : Le nom de votre job semble être **"my project"** (avec un espace). Utilisez ce nom exact dans Jenkins.

---

### 2️⃣ Ngrok N'est PAS en Cours d'Exécution

**Vérification** :
```powershell
# Dans une fenêtre PowerShell, lancez ngrok :
ngrok http 8080
```

Vous devriez voir :
```
Forwarding  https://a92c4b39124b.ngrok-free.app -> http://localhost:8080
```

⚠️ Si Ngrok n'est pas lancé, GitHub ne peut PAS atteindre Jenkins !

---

### 3️⃣ URL du Webhook GitHub Incorrecte

**Vérification** :
1. GitHub → Repository → Settings → Webhooks
2. L'URL doit être : `https://xxxx.ngrok-free.app/github-webhook/`
3. ⚠️ Le `/` final est **OBLIGATOIRE**

---

### 4️⃣ Le Job Pipeline n'Existe Pas ou Mal Configuré

**Vérification** :
1. Dashboard Jenkins → Devez voir votre job (ex: `tp30-pipeline`)
2. Si absent, créez-le :
   - **New Item** → Nom : `tp30-pipeline` → Type : **Pipeline**
   - **Pipeline** → **SCM** : Git
   - **Repository URL** : `https://github.com/SaadBengrich/tp30.git`
   - **Branch** : `*/main`
   - **Script Path** : `Jenkinsfile`

---

## ✅ Solution Complète Étape par Étape

### Étape 1 : Vérifier Ngrok

```powershell
# Lancez ngrok dans une fenêtre PowerShell
ngrok http 8080
```

**Résultat attendu** :
```
Session Status                online
Forwarding                    https://a92c4b39124b.ngrok-free.app -> http://localhost:8080
```

🔑 **Notez l'URL** : `https://a92c4b39124b.ngrok-free.app`

---

### Étape 2 : Mettre à Jour le Webhook GitHub

1. **GitHub** → https://github.com/SaadBengrich/tp30/settings/hooks
2. **Cliquez sur votre webhook** (ou Add webhook si absent)
3. **Payload URL** : `https://a92c4b39124b.ngrok-free.app/github-webhook/`
4. **Content type** : `application/json`
5. **Which events** : `Just the push event`
6. ✅ **Active**
7. **Update webhook**

---

### Étape 3 : Configurer le Job Jenkins

1. **Dashboard** → **tp30-pipeline** → **Configure**

2. **Section Build Triggers** :
   ```
   ✅ GitHub hook trigger for GITScm polling
   ```

3. **Section Pipeline** :
   - **Definition** : `Pipeline script from SCM`
   - **SCM** : `Git`
   - **Repository URL** : `https://github.com/SaadBengrich/tp30.git`
   - **Credentials** : `-none-` (si public)
   - **Branch Specifier** : `*/main`
   - **Script Path** : `Jenkinsfile`

4. **Save**

---

### Étape 4 : Tester le Webhook

#### A. Test depuis GitHub UI

1. **GitHub** → **Settings** → **Webhooks** → Votre webhook
2. **Recent Deliveries** → Sélectionnez une livraison → **Redeliver**
3. **Vérifiez la Response** :
   - Code **200** = OK ✅
   - Code **404** = URL incorrecte ❌
   - Code **500** = Erreur Jenkins ❌

#### B. Test avec un Push

```powershell
# Modifiez un fichier
echo "# Test webhook" >> README.md
git add .
git commit -m "Test: déclenchement webhook"
git push origin main
```

---

### Étape 5 : Vérifier les Logs Jenkins

```powershell
# Voir les logs en temps réel
docker logs -f jenkins

# Ou vérifier les dernières lignes
docker logs jenkins --tail 50 | Select-String -Pattern "POST|webhook|GitHub"
```

**Ce que vous devriez voir après un push** :
```
Received POST from /github-webhook/
Notifying about SCM change
Scheduling project: tp30-pipeline
```

---

## 🔧 Solution Rapide si Ngrok a Redémarré

⚠️ **Ngrok change l'URL à chaque redémarrage !**

```powershell
# 1. Vérifiez la nouvelle URL Ngrok
# Dans la fenêtre Ngrok, copiez la nouvelle URL

# 2. Mettez à jour le webhook GitHub
# GitHub → Settings → Webhooks → Edit
# Payload URL: https://NOUVELLE_URL.ngrok-free.app/github-webhook/

# 3. Testez immédiatement
git commit --allow-empty -m "Test webhook"
git push origin main
```

---

## 📊 Checklist Complète

### Avant de faire un push, vérifiez :

- [ ] Ngrok tourne : `ngrok http 8080`
- [ ] Jenkins tourne : `docker ps | findstr jenkins`
- [ ] Webhook GitHub configuré avec la bonne URL Ngrok
- [ ] Build Trigger activé dans le job Jenkins
- [ ] Job Pipeline existe et pointe vers le bon repository
- [ ] Jenkinsfile existe dans le repository : `Jenkinsfile`

---

## 🎯 Test Final Complet

```powershell
# 1. Vérifier que tout est en place
docker ps | findstr jenkins
# → Devrait afficher le conteneur jenkins

# 2. Vérifier Ngrok (dans sa fenêtre)
# → Devrait afficher: Forwarding https://xxx.ngrok-free.app -> http://localhost:8080

# 3. Faire un changement
cd C:\Users\Saad\Desktop\tp30
echo "Test $(Get-Date)" >> README.md
git add .
git commit -m "Test: webhook automatique"
git push origin main

# 4. Attendre 5 secondes et vérifier Jenkins
Start-Sleep -Seconds 5
docker logs jenkins --tail 20

# 5. Vérifier le Dashboard Jenkins
# → Devrait voir un nouveau build en cours
```

---

## 🆘 Si Rien Ne Fonctionne : Utiliser Poll SCM

**Alternative temporaire** (Jenkins vérifie GitHub toutes les minutes) :

1. **tp30-pipeline** → **Configure**
2. **Build Triggers** → ✅ **Poll SCM**
3. **Schedule** : `* * * * *`
4. **Save**

Jenkins vérifiera GitHub toutes les minutes et lancera un build si des changements sont détectés.

---

## ✅ Résultat Attendu

Après la configuration correcte :

1. Vous faites `git push origin main`
2. GitHub envoie un webhook à Ngrok
3. Ngrok transmet à Jenkins (`/github-webhook/`)
4. Jenkins détecte le push sur la branche `main`
5. Le job `tp30-pipeline` se lance automatiquement
6. Dans Console Output, vous voyez : `Started by GitHub push by SaadBengrich`

---

## 📝 Logs à Surveiller

### Jenkins doit afficher :
```
Received POST from /github-webhook/
Notifying about SCM change
Scheduling project: tp30-pipeline
```

### GitHub Recent Deliveries doit afficher :
```
Response: 200 OK
```

### Ngrok doit afficher :
```
POST /github-webhook/ 200 OK
```

---

**Date** : 2025-12-16
**Problème** : Webhook GitHub ne déclenche pas le build Jenkins
**Cause** : Build Trigger non activé OU Ngrok non lancé OU URL webhook incorrecte

