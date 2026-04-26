# Djinn

**Catégorie :** Reverse Engineering  
**Architecture :** ELF x86-64  
**Flag :** `EcowasCTF{Dj1nN_5t4t3_mAcH1n3_0xD34D}`

## Description

Binaire strippé avec anti-debug et machine à états.

## Writeup

### Étape 1 — Reconnaissance

```bash
file djinn        # ELF 64-bit PIE, stripped
strings djinn     # /proc/self/status, TracerPid, strace, ltrace
```

### Étape 2 — Comportement anti-debug

```bash
echo "test" | ./djinn        # "Silence."
echo "test" | strace ./djinn  # "The djinn speaks: correct." ← LEURRE
```

Sous strace, le binaire dit "correct" **sans vérifier l'input** : c'est un leurre anti-debug.

### Étape 3 — Architecture — Machine à états (~37 états)

```
État 999 → Vérification anti-debug
    ├── Debugger détecté → État 0x3e5 → "correct" (LEURRE)
    └── Pas de debugger → Checksum table (256 bytes)
            └── Checksum != 0 → État 0x68 → Machine à états réelle
                    ↓
              Comparaison input[0..36] XOR expected[0..36] == 0
```

**Pièges intentionnels :**
- Anti-debug via `/proc/self/status` (TracerPid)
- Checksum inversé : `sum ≠ 0` = table VALIDE
- Sous strace → "correct" sans checker l'input

### Étape 4 — PRNG interne (seed `0xDEADBEEF`)

```python
def prng(x):
    x &= 0xFFFFFFFF
    edx = (x >> 1) ^ (x >> 0x15) ^ x ^ (x >> 0x1f)
    edx &= 1
    eax = ((x * 2) + edx) & 0xFFFFFFFF
    return (eax - ((eax - (x*2 ^ edx)) >> 1)) & 0xFFFFFFFF
```

### Étape 5 — Extraction du flag

On patche la fonction anti-debug pour qu'elle retourne 0 :

```python
# Patch FUN_001028c0: xor eax,eax; ret
data[0x28c0] = 0x31; data[0x28c1] = 0xc0; data[0x28c2] = 0xc3
```

Puis sous GDB :

```gdb
file djinn_patched
tb fgets
r <<< "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"
fin
x/37cb $rdi
# → "EcowasCTF{Dj1nN_5t4t3_mAcH1n3_0xD34D}"
```

## Flag

```
EcowasCTF{Dj1nN_5t4t3_mAcH1n3_0xD34D}
```

> **Djinn** caché dans une **machine à états** avec seed `0xDEAD` (0xDEADBEEF).
