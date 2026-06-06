

# 🛡️ Laboratoire JNI Défensif – Anti-Debug & Inspection Mémoire

## 🎯 Objectif
Ajouter une couche de sécurité native à une application Android pour détecter :
- Un débogueur attaché au processus (`ptrace`)
- La présence de bibliothèques suspectes (Frida, Xposed, etc.) via `/proc/self/maps`

L’application adapte alors son comportement (affichage, blocage de fonctions).

---

## 📚 Prérequis
- Projet `JNIDemo` du TP précédent
- Android Studio + NDK, CMake, LLDB installés
- Notions de base JNI

---

## 🧠 Fonctionnement

1. **Appel Java** : `isDebugDetected()` depuis `MainActivity`
2. **Contrôles natifs** :
   - `ptrace(PTRACE_TRACEME)` → détection de trace
   - Lecture de `/proc/self/maps` → recherche de mots-clés (`frida`, `xposed`, `magisk`, etc.)
3. **Remontée** : un booléen vers Java
4. **Réaction** : affichage d’un statut / blocage des autres fonctions JNI

---

## 📁 Structure modifiée

```
app/src/main/
├── cpp/
│   ├── native-lib.cpp      ← ajout des fonctions anti-debug
│   └── CMakeLists.txt      ← inchangé
├── java/.../MainActivity.java  ← ajout de isDebugDetected() et logique d’affichage
└── res/layout/activity_main.xml ← layout enrichi
```

---

## 🔍 Extraits clés

### C++ – Détection `ptrace`
```cpp
static bool isBeingTraced() {
    return ptrace(PTRACE_TRACEME, 0, 0, 0) == -1;
}
```

### C++ – Inspection mémoire
```cpp
if (strstr(line, "frida") || strstr(line, "xposed") || ...)
```

### Java – Adaptation UI
```java
if (suspicious) {
    tvStatus.setText("environnement suspect");
    // désactivation des fonctionnalités natives
} else {
    tvHello.setText(helloFromJNI());
}
```

---

## 🧪 Scénarios de test

| Contexte | Résultat attendu |
|----------|------------------|
| Lancement normal | Statut OK, affichage des résultats JNI |
| Débogage (Android Studio) | Détection possible → affichage “environnement suspect” |
| Présence de Frida (simulée) | Détection dans `/proc/self/maps` |

---

## 📊 Logs (Logcat)
Filtrer par `ANTI_DEBUG` pour observer :
```
Aucun trace/debug detecte via ptrace
Aucune signature suspecte trouvee dans /proc/self/maps
Etat de securite : OK
```

## 📖 Références
- [Documentation officielle NDK](https://developer.android.com/ndk)
- [JNI tips (Android)](https://developer.android.com/training/articles/perf-jni)

--- 

Ce laboratoire prolonge `JNIDemo` en ajoutant une **logique défensive crédible et observable**, sans illusion d’invincibilité. 🧪
