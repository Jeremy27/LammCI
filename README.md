# LammCI

Workflows GitHub Actions réutilisables pour la suite **Lamm**.

Évite la duplication des `.github/workflows/release.yml` à travers chaque projet : chaque repo Lamm appelle ces workflows via `workflow_call` avec quelques paramètres.

## Workflows disponibles

### `library-release.yml`

Pour les libs Lamm publiées sur **GitHub Packages** (LammUI). Vérifie que la version du pom == tag, puis `mvn deploy`.

**Caller (LammUI) :**
```yaml
name: Publish to GitHub Packages

on:
  push:
    tags: ['v*']

jobs:
  publish:
    uses: Jeremy27/LammCI/.github/workflows/library-release.yml@main
    permissions:
      contents: read
      packages: write
```

### `app-release.yml`

Pour les apps Lamm packagées via **jpackage** (DEB Linux + MSI Windows). Build matrix linux+windows, vérifie pom == tag, upload des installateurs comme assets sur la GitHub Release du tag.

**Caller (Lammprimante) :**
```yaml
name: Release Lammprimante

on:
  push:
    tags: ['v*']

jobs:
  release:
    uses: Jeremy27/LammCI/.github/workflows/app-release.yml@main
    with:
      app-name: Lammprimante
    permissions:
      contents: write
      packages: read
```

## Inputs `app-release`

| Input | Default | Description |
|---|---|---|
| `app-name` | (requis) | Nom dans le titre de la release |
| `linux-profile` | `dist-linux` | Profil Maven Linux |
| `windows-profile` | `dist-win` | Profil Maven Windows |
| `linux-artifact-glob` | `target/*.deb` | Glob des assets Linux |
| `windows-artifact-glob` | `target/*.msi` | Glob des assets Windows |
| `java-version` | `25` | Version JDK Temurin |

## Inputs `library-release`

| Input | Default | Description |
|---|---|---|
| `java-version` | `25` | Version JDK Temurin |

## Prérequis côté projet appelant

### Apps (jpackage)
- Profils Maven `dist-linux` et `dist-win` (configurables via inputs) qui exécutent jpackage et déposent les artefacts dans `target/`
- Repo Maven `github-lammui` déclaré dans le pom pour tirer LammUI

### Libs
- `<distributionManagement>` qui pointe sur `https://maven.pkg.github.com/Jeremy27/<lib>`

## Versioning

Les callers utilisent `@main` pour suivre la dernière version. Pour pinner sur un commit ou un tag, remplacer par `@<sha>` ou `@v1.0.0`.

## Pourquoi un repo séparé ?

Pour ne pas mélanger l'infra CI avec le code applicatif (LammUI). Un seul endroit à modifier quand une action GitHub se met à jour ou que la version JDK change.
