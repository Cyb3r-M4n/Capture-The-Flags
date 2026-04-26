# Nested Layers

**Catégorie :** MISC  
**Flag :** `EcowasCTF{n3st3d_l4y3rs_0f_s4nk0f4_w1sd0m}`

## Writeup

Le challenge consiste à décoder une chaîne passant par plusieurs couches d'encodage :

```
cipher.txt → Base64 → Base85 → HEX → Base32 → ???
```

### Pipeline de décodage

```python
import base64

# Étape 1 : Base64
step1 = base64.b64decode(open("cipher.txt").read())

# Étape 2 : Base85
step2 = base64.b85decode(step1)

# Étape 3 : Hex decode
step3 = bytes.fromhex(step2.decode())

# Étape 4 : Base32
step4 = base64.b32decode(step3)

print(step4.decode())
```

## Flag

```
EcowasCTF{n3st3d_l4y3rs_0f_s4nk0f4_w1sd0m}
```
