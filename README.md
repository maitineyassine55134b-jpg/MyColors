# MyColors – Application Android (APK)

*Des couleurs pour tous.* Ce dépôt emballe l'application web `www/index.html` dans une vraie
application Android avec **Capacitor** et construit l'APK automatiquement sur **GitHub Actions**
(aucune installation d'Android Studio nécessaire).

```
MyColors-Android/
├─ www/index.html                  ← l'application MyColors (fichier unique)
├─ capacitor.config.json           ← nom, identifiant (com.mycolors.app), couleurs
├─ package.json / package-lock.json
├─ android/                        ← projet Android natif (généré par Capacitor, icônes MyColors incluses)
├─ resources/                      ← icône 1024 px + icône Play Store 512 px
└─ .github/workflows/android-apk.yml   ← construit l'APK sur GitHub
```

## 1. Mettre le projet sur GitHub

1. Créez un dépôt **vide** sur <https://github.com/new> (ex. `MyColors-Android`), sans README.
2. Décompressez le ZIP, puis dans un terminal :

```bash
cd MyColors-Android
git init
git add .
git commit -m "MyColors Android"
git branch -M main
git remote add origin https://github.com/VOTRE-NOM/MyColors-Android.git
git push -u origin main
```

> Avec **GitHub Desktop**, faites *File → Add local repository*, puis *Publish repository*.
> Évitez le glisser-déposer dans le navigateur : il ignore souvent le dossier caché `.github`,
> et le workflow ne se lancerait pas.

## 2. Récupérer l'APK

1. Ouvrez l'onglet **Actions** du dépôt : le workflow *Build Android APK* démarre tout seul au `push`
   (≈ 5 à 8 minutes la première fois). Vous pouvez aussi le relancer avec *Run workflow*.
2. Quand il est vert, ouvrez l'exécution et téléchargez l'artefact **MyColors-debug-apk** (en bas de page).
3. Décompressez-le : `MyColors-debug.apk`.

## 3. Installer sur un téléphone

Copiez `MyColors-debug.apk` sur le téléphone, ouvrez-le et autorisez l'installation depuis
cette source (« Installer des applications inconnues »).

L'APK *debug* est signé automatiquement avec une clé de test : parfait pour tester et partager.
Pour le **Play Store**, utilisez l'option suivante.

## 4. (Optionnel) APK + AAB signés pour le Play Store

1. Créez une clé de signature (une seule fois, **gardez-la précieusement**) :

```bash
keytool -genkeypair -v -keystore mycolors.jks -alias mycolors \
  -keyalg RSA -keysize 2048 -validity 10000
```

2. Encodez-la en base64 :
   - Linux / macOS : `base64 -w0 mycolors.jks`
   - Windows PowerShell : `[Convert]::ToBase64String([IO.File]::ReadAllBytes("mycolors.jks"))`
3. Dans GitHub : *Settings → Secrets and variables → Actions → New repository secret*, créez :

| Secret | Valeur |
|---|---|
| `ANDROID_KEYSTORE_BASE64` | le texte base64 du fichier `.jks` |
| `ANDROID_KEYSTORE_PASSWORD` | le mot de passe du keystore |
| `ANDROID_KEY_ALIAS` | `mycolors` |
| `ANDROID_KEY_PASSWORD` | le mot de passe de la clé |

Au prochain `push`, l'artefact **MyColors-release-signed** contient `MyColors-release.apk` et
`MyColors-release.aab` (ce dernier est le format demandé par Google Play).

## 5. Publier une Release GitHub avec l'APK

```bash
git tag v1.0.0
git push origin v1.0.0
```

Le workflow crée une *Release* et y joint les fichiers APK/AAB.

## Mettre à jour l'application

Modifiez `www/index.html`, faites `git commit` + `git push` : un nouvel APK est construit.
Le numéro de version interne (`versionCode`) augmente automatiquement à chaque exécution.

## Personnalisation

| Je veux changer… | Où |
|---|---|
| le nom affiché | `capacitor.config.json` (`appName`) **et** `android/app/src/main/res/values/strings.xml` |
| l'identifiant (`com.mycolors.app`) | `capacitor.config.json` (`appId`), `android/app/build.gradle` (`namespace`, `applicationId`), `strings.xml`, et le dossier/paquet de `MainActivity.java` |
| le numéro de version affiché | `android/app/build.gradle` → `versionName` |
| les icônes | `android/app/src/main/res/mipmap-*` (fond adaptatif : `drawable/ic_launcher_background.xml`) |

## Tester en local (optionnel)

Prérequis : Node 22+, JDK 21, Android Studio.

```bash
npm install
npx cap sync android
npx cap open android      # ouvre Android Studio, puis ▶ Run
# ou en ligne de commande :
cd android && ./gradlew assembleDebug
# APK : android/app/build/outputs/apk/debug/app-debug.apk
```

## Notes techniques

- Capacitor 8 · `minSdk 24` (Android 7.0+) · `targetSdk 36`.
- L'application fonctionne **hors ligne** ; les données (comptes, couleurs enregistrées, préférences)
  sont stockées dans le `localStorage` du téléphone. Désinstaller l'app les supprime.
- Seule modification apportée à votre `index.html` : un petit bloc CSS « zone sûre » (barre d'état Android en
  mode plein écran) et la synchronisation de la couleur des icônes de la barre d'état avec le thème clair/sombre.
  Sans effet dans un navigateur ordinaire.
