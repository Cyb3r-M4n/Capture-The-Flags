# Gates

**Catégorie :** Reverse Engineering  
**Architecture :** Mach-O ARM64  
**Flag :** `EcowasCTF{0nly_0n3_g4t3_0p3ns_th3_w4y}`

## Description

Binaire macOS ARM64 avec des dizaines de faux flags (bloating).

## Writeup

### Étape 1 — Analyse initiale

```bash
file gates
# gates: Mach-O 64-bit arm64 executable, flags:<NOUNDEFS|DYLDLINK|TWOLEVEL|PIE>
```

Binaire **Mach-O ARM64** — ne s'exécute pas sur Linux x86.

### Étape 2 — Bloating de strings

```bash
strings gates | grep EcowasCTF
# → plus de 100 flags potentiels !
```

L'auteur a noyé le vrai flag dans une centaine de faux flags.

### Étape 3 — Reverse avec Ghidra

On importe dans Ghidra. La fonction `entry` compare l'input utilisateur à des constantes hexadécimales sur la pile (Little-Endian) :

```c
if (local_b8 == 0x54437361776f6345 &&
    lStack_b0 == 0x305f796c6e307b46 && ...)
```

### Étape 4 — Conversion Little-Endian → ASCII

| Valeur hex | Octets LE | ASCII |
|------------|-----------|-------|
| `0x54437361776f6345` | `45 63 6f 77 61 73 43 54` | `EcowasCT` |
| `0x305f796c6e307b46` | `46 7b 30 6e 6c 79 5f 30` | `F{0nly_0` |

En combinant toutes les constantes : **`EcowasCTF{0nly_0n3_g4t3_0p3ns_th3_w4y}`**

## Flag

```
EcowasCTF{0nly_0n3_g4t3_0p3ns_th3_w4y}
```
