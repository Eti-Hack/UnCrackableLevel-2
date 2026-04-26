# LAB — Analyse d'une application Android avec code natif JNI

## Objectif

Nous avons analysé une application Android (UnCrackable Level 2) dont la logique de vérification est cachée dans une bibliothèque native. L'objectif était de retrouver le secret attendu par l'application.

---

## Outils utilisés

- `adb` — installation de l'APK
- **JADX** — décompilation du code Java
- `unzip` — extraction du contenu de l'APK
- **Ghidra** — analyse de la bibliothèque native

---

## Ce que nous avons fait

### 1. Installation et observation

Nous avons installé l'APK via `adb` et testé l'application. En saisissant des valeurs aléatoires (`test`, `1234`, `hello`), nous avons constaté qu'un message d'erreur s'affichait à chaque fois, confirmant que l'application attendait une chaîne précise.

### 2. Décompilation avec JADX

Nous avons ouvert l'APK dans JADX et analysé `MainActivity`. Nous avons retracé le chemin de la valeur saisie par l'utilisateur jusqu'à un appel vers la classe `CodeCheck`.

### 3. Découverte du code natif

Dans la classe `CodeCheck`, nous avons trouvé deux éléments clés :

```java
System.loadLibrary("foo");
private native boolean bar(String s);
```

La méthode `bar` est déclarée `native` — son code n'est pas en Java mais dans une bibliothèque compilée en C.

### 4. Extraction de libfoo.so

Nous avons extrait le contenu de l'APK et localisé la bibliothèque :

```bash
unzip UnCrackable-Level2.apk -d uncrackable_l2
# → uncrackable_l2/lib/x86/libfoo.so
```

### 5. Analyse avec Ghidra

Nous avons importé `libfoo.so` dans Ghidra et lancé l'analyse automatique. Nous avons retrouvé la fonction JNI exportée :

```
Java_sg_vantagepoint_uncrackable2_CodeCheck_bar
```

Dans le pseudo-code, nous avons repéré un appel à `strncmp` comparant l'entrée utilisateur à une valeur hexadécimale stockée en mémoire :

```
6873696620656874206c6c6120726f6620736b6e616854
```

### 6. Décodage du secret

Nous avons décodé cette valeur en Python :

```python
hex_data = "6873696620656874206c6c6120726f6620736b6e616854"
print(bytes.fromhex(hex_data).decode("ascii"))
# → hsif eht lla rof sknahT
```

La chaîne étant stockée à l'envers, nous l'avons inversée :

```python
print("hsif eht lla rof sknahT"[::-1])
# → Thanks for all the fish
```

---

## Résultat

Nous avons saisi le secret dans l'application et validé le challenge avec succès.

```
Thanks for all the fish
```

---

## Conclusion

Ce lab nous a permis de comprendre comment une application Android peut déléguer sa logique sensible à une bibliothèque native via JNI. En combinant JADX pour le code Java et Ghidra pour le code natif, nous avons retracé le chemin complet de la vérification et retrouvé le secret.
