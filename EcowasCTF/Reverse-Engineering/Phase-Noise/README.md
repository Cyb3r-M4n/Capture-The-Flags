# Phase Noise

**Catégorie :** Reverse Engineering  
**Architecture :** Mach-O ARM64  
**Flag :** `EcowasCTF{vm_lifted_nibbles_are_louder_than_strings}`

## Description

Validation nibble par nibble via une table de lookup de 16 entrées — résiste totalement à `strings`.

## Writeup

### Étape 1 — Reconnaissance

```bash
file phase_noise
# Mach-O 64-bit arm64 executable

strings phase_noise | grep -i ctf
# → rien. Le flag n'est pas en clair.
```

### Étape 2 — Tables statiques (Ghidra)

```
DAT_f62 (16 bytes) : 6d 2b f0 91 3c a7 58 de 04 bb 72 19 e5 40 8a cd
DAT_f72 (52 bytes) : valeurs attendues pour chaque itération
```

`DAT_f72` contient exactement **52 bytes** → longueur du flag imposée.

### Étape 3 — Condition de validation

```python
# Condition 1 — longueur
(len(passphrase) & 0xFF) ^ 0x34 == 0  →  len == 52

# Condition 2 — chaque itération i
DAT_f72[i] == uVar1   # OR accumulé sur uVar11 doit rester == 0
```

Comme chaque contrainte est **indépendante et inversible** : complexité **O(n × 256)** au lieu de O(256ⁿ).

### Étape 4 — Solver

```python
DAT_f62 = bytes.fromhex("6d2bf0913ca758de04bb7219e5408acd")
DAT_f72 = bytes.fromhex(
    "317a8b34f07e86d02174d380d536a94c"
    "a93a0b8cafbe16b127e22304f994174e"
    "3596d3a409066f66a9cbc381a9b6fb50"
    "39ea93ec"
)

def invert_char(target_uVar1, state, uVar14):
    uVar7, _, uVar13, _, iVar9, iVar10 = state
    rot = compute_rot(uVar7)
    inner_target = (uVar13 ^ target_uVar1) & 0xFF
    candidates = []
    for rotated in range(256):
        idx = (iVar9 + rotated) & 0xF
        if (iVar10 + DAT_f62[idx] + rotated) & 0xFF == inner_target:
            char_byte = (rotate_right(rotated, rot) ^ uVar14) & 0xFF
            candidates.append(char_byte)
    return candidates

# Récupération position par position
result = recurse(0, initial_state, [])
print(bytes(result).decode())
# → EcowasCTF{vm_lifted_nibbles_are_louder_than_strings}
```

### Takeaway

Quand une validation accumule des résultats via **OR** sur une valeur devant rester nulle, chaque terme est une contrainte individuelle inversible → décomposition position par position.

## Flag

```
EcowasCTF{vm_lifted_nibbles_are_louder_than_strings}
```
