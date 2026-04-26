# 5 Friend

**Catégorie :** Cryptographie  
**Flag :** `EcowasCTF{90715ab05d61b03306f1ebf7e0fced7a}`

## Writeup

### Attaque en plusieurs étapes

**1. Trouver notre `e` via l'algorithme de Wiener** (fractions continues de `d/N`) → `e = 65537`

**2. Calculer `phi(N)` :**

```
phi = (e*d - 1) / k   avec k = 2160
# Vérifié : pow(2, phi, N) == 1
```

**3. Attaque Common Modulus**

Le flag était chiffré avec le **produit de TOUS les exposants des amis** :

```
e1 * e2 * e3 * e4 * e5 = 72719 * 105899 * 70351 * 117307 * 119267
```

**4. Déchiffrement :**

```python
d_final = pow(e1*e2*e3*e4*e5, -1, phi)
m = pow(c, d_final, N)
```

## Flag

```
EcowasCTF{90715ab05d61b03306f1ebf7e0fced7a}
```
