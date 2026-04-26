# Silent Whispers II

**Catégorie :** Steganographie  
**Flag :** `flag{l@y37s_0n_l@y3rs}`

## Description

> We intercepted another message… and this time, it feels intentional. At first glance, everything looks normal. But given past incidents, we suspect something is hidden beneath the surface. This may not be a simple extraction. Be prepared to dig deeper.

Fichier fourni : `information_II.txt`

## Writeup

### Étape 1 — Extraction avec stegsnow

```bash
stegsnow -C information_II.txt
```

On obtient une longue chaîne base64.

### Étape 2 — Décodage du ZIP en base64

```bash
echo "UEsDBBQACQAIAIkGc1wipgyP..." | base64 -d > Stg.zip
```

![Décodage base64 vers ZIP](images/img2.png)

### Étape 3 — Extraction du ZIP protégé par mot de passe

![ZIP protégé](images/img3.png)

Le ZIP est protégé par un mot de passe. On utilise `john` pour le cracker :

```bash
zip2john Stg.zip > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

![Cracking john](images/img4.png)

Mot de passe trouvé : `stealth123`

### Étape 4 — Lecture du flag

![Flag](images/img1.png)

## Flag

```
flag{l@y37s_0n_l@y3rs}
```
