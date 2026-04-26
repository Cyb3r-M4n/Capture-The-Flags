# Obscura

**Catégorie :** Mobile  
**Flag :** `EcowasCTF{h4rdc0d3d_m0b1l3_xpl0it}`

## Description

Un fichier APK nommé `Obscura.apk` est fourni. L'objectif est d'analyser l'application Android pour extraire des credentials cachés et accéder à une API distante.

## Writeup

### Étape 1 — Décompilation avec Apktool

```bash
apktool d Obscura.apk -o Mobile
```

Structure obtenue :

```
Mobile/
├── AndroidManifest.xml
├── res/values/strings.xml
└── smali/com/metasploit/stage/
    ├── MainActivity.smali
    └── ...
```

> Le package `com.metasploit.stage` indique un payload généré via **Metasploit** (msfvenom Android reverse shell).

### Étape 2 — Permissions suspectes (AndroidManifest.xml)

L'application demande des permissions très intrusives : `RECORD_AUDIO`, `CAMERA`, `READ_SMS`, `READ_CONTACTS`, `ACCESS_FINE_LOCATION` → c'est un **RAT (Remote Access Trojan)**.

### Étape 3 — Credentials dans strings.xml

```xml
<string name="api_url">http://192.168.45.5:5003/login</string>
<string name="password">user_access</string>
<string name="username">user</string>
```

### Étape 4 — Mot de passe caché en Base64 (MainActivity.smali)

```smali
const-string v0, "czNjcjN0IQ=="
```

```bash
echo "czNjcjN0IQ==" | base64 -d
# s3cr3t!
```

### Étape 5 — Exploitation de l'API

```bash
# Connexion user (succès mais pas de flag)
curl -X POST http://labs.ecowasctf.com.gh:5003/login \
  -H "Content-Type: application/json" \
  -d '{"username":"user", "password":"user_access"}'
# → {"message":"Login successful"}

# Tentative admin avec le mot de passe caché
curl -X POST http://labs.ecowasctf.com.gh:5003/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin", "password":"s3cr3t!"}'
# → {"flag":"EcowasCTF{h4rdc0d3d_m0b1l3_xpl0it}","message":"Welcome admin"}
```

### Résumé des artefacts

| Artefact | Valeur | Source |
|----------|--------|--------|
| Username | `user` | `strings.xml` |
| Password (surface) | `user_access` | `strings.xml` |
| Password (caché) | `s3cr3t!` | `MainActivity.smali` (Base64) |
| API Endpoint | `http://labs.ecowasctf.com.gh:5003/login` | `strings.xml` |

## Flag

```
EcowasCTF{h4rdc0d3d_m0b1l3_xpl0it}
```
