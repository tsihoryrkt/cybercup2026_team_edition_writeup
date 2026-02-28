# Fréquences Malgaches

## Description

L'agent a transmis un second fichier : un enregistrement audio capté sur une fréquence radio illicite dans la région malgache. > Les analystes ont passé des heures sur le spectrogramme. Rien. > Pourtant, le fichier contient quelque chose. Regardez là où les outils ne regardent pas. *Prérequis narratif : avoir résolu "Fantôme de la Vanille"

## Analyse
Le spectrogramme n'a rien donné, si quelque chose est dissimulé dans le fichier, il est possible que des données textuelles y soient directement embarquées en clair dans les métadonnées ou entre les blocs audio. 
### Identification du fichier
```bash
file frequences_malgaches.wav
```
> frequences_malgaches.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 44100 Hz

### Extraction des chaînes lisibles
strings va passer le fichier en revue octet par octet à la recherche de séquences lisibles.
```bash
strings frequences_malgaches.wav
``` 
```console
OCOI
FCRPGER_ABQR
IIJbIIXIKuZPnPcZKLCueQ==
TvMkR4/JaawTmqFDbrfJJj3UWFHezyKtB5ydGi60uHA6iFJRiMo861eR80o=
Uk9UMTMoY29kZW5hbWUpIC0+IFhPUiBrZXkgPSBNRDUoY29kZW5hbWUp
```
Plusieurs chaînes suspectes remontent. Certaines ressemblent à du Base64, d'autres à du texte obfusqué. 
```bash
echo "Uk9UMTMoY29kZW5hbWUpIC0+IFhPUiBrZXkgPSBNRDUoY29kZW5hbWUp" | base64 -d
```
```console
ROT13(codename) -> XOR key = MD5(codename)
```
**Indice révélé** : le fichier cache un schéma de chiffrement en trois étapes.
```
1. Appliquer ROT13 à un "codename" pour obtenir sa forme déchiffrée
2. Calculer le MD5 de ce codename → c'est la clé XOR
3. XOR-er le payload Base64 avec cette clé pour obtenir le flag
```
### Déchiffrement

Le codename est `SPECTRE_NODE` et la seconde chaîne Base64 est la payload chiffré

```python
import codecs, base64, hashlib

codename = codecs.decode('FCRPGER_ABQR', 'rot13')
key = hashlib.md5(codename.encode()).digest()

payload = base64.b64decode('TvMkR4/JaawTmqFDbrfJJj3UWFHezyKtB5ydGi60uHA6iFJRiMo861eR80o=')

flag = bytes([payload[i] ^ key[i % len(key)] for i in range(len(payload))])
print('flag:', flag)
```

- L'exécution du script révèle le flag.