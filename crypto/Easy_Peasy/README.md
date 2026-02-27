# Write-up – Challenge "Easy Peasy"

## 1. Analyse du code

On nous donne :

- un fichier chiffré `challenge.bin`
- le code Python qui a servi à chiffrer le flag

Le code fourni :

```python
from pathlib import Path
import random

FLAG = b"REDACTED"
KEY_MIN = 1
KEY_MAX = 80
KEY = random.randint(KEY_MIN, KEY_MAX)

data = bytes((((b + 2) & 255) ^ KEY) for b in FLAG)
Path("challenge.bin").write_bytes(data)
print(data.hex())
print(KEY)
```

## 2. Comprendre le chiffrement

Pour chaque byte du flag :

```python
cipher = ((b + 2) & 255) ^ KEY
```

Donc :

1. On ajoute 2 au byte
2. On applique un AND 255 (pour rester sur 1 octet)
3. On applique un XOR avec la clé

La clé est générée aléatoirement entre 1 et 80.

C’est donc un chiffrement simple : addition + AND + XOR.

## 3. Observation importante

La clé est comprise entre 1 et 80.
Il y a donc seulement 80 possibilités.
On peut donc tester toutes les clés et chercher le flag dans le format connu, par exemple `CTF{...}`.

## 4. Reverse du chiffrement

La formule inverse :

```python
original = ((cipher ^ KEY) - 2) & 255
```

## 5. Script de déchiffrement (bruteforce)

```python
from pathlib import Path

data = Path("challenge.bin").read_bytes()

for key in range(1, 81):
    decrypted = bytes(((b ^ key) - 2) & 255 for b in data)

    try:
        text = decrypted.decode()

        if "CCOI26{" in text :
            print(f"[+] key: {key}")
            print(text)
            print("-" * 30)
    except:
        pass
```

## 6. Explication

- On lit le fichier chiffré
- On teste toutes les clés de 1 à 80
- On applique la formule inverse
- On affiche seulement les résultats lisibles
- On cherche un format de flag connu

## 7. Conclusion

Ce challenge est simple car :

- L’algorithme est connu
- La clé est petite (1 → 80)
- On peut bruteforce facilement

La solution consiste donc à tester toutes les clés possibles et chercher le flag.
