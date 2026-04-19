# Guia completa de Git: de principiante a avanzado (hasta produccion con CI en GitHub)

## 1. Que es Git y por que usarlo

Git es un sistema de control de versiones distribuido. Te permite:

- Guardar el historial de cambios de tu codigo.
- Trabajar en equipo sin pisar cambios.
- Volver a versiones anteriores cuando algo falla.
- Automatizar pruebas y despliegues con integracion continua.

## 2. Instalacion y configuracion inicial

### 2.1 Instalar Git

- Windows: instalar desde https://git-scm.com/
- Verificar instalacion:

```bash
git --version
```

### 2.2 Configuracion global basica

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu_correo@ejemplo.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"
```

Ver configuracion actual:

```bash
git config --list
```

## 3. Conceptos clave

- Repositorio: carpeta controlada por Git.
- Working tree: archivos en tu disco.
- Staging area (index): zona donde preparas cambios para commit.
- Commit: snapshot de cambios con mensaje.
- Branch: linea de trabajo paralela.
- Merge/Rebase: formas de integrar cambios.
- Remote: repositorio remoto (por ejemplo GitHub).
- Tag: marca de version (v1.0.0, v2.1.3, etc.).

## 4. Flujo basico local (principiante)

### 4.1 Crear o clonar repositorio

Crear repo nuevo:

```bash
mkdir mi-proyecto
cd mi-proyecto
git init
```

Clonar repo existente:

```bash
git clone https://github.com/usuario/mi-proyecto.git
cd mi-proyecto
```

### 4.2 Ciclo diario minimo

Ver estado:

```bash
git status
```

Agregar cambios al stage:

```bash
git add .
# o especifico
git add src/app.js
```

Hacer commit:

```bash
git commit -m "feat: agrega formulario de registro"
```

Ver historial:

```bash
git log --oneline --graph --decorate --all
```

## 5. Buenas practicas de commits

Usa mensajes claros y convencionales:

- feat: nueva funcionalidad
- fix: correccion de bug
- docs: documentacion
- refactor: mejora sin cambiar comportamiento
- test: pruebas
- chore: tareas de mantenimiento

Ejemplos:

```bash
git commit -m "fix(auth): corrige validacion de token expirado"
git commit -m "docs(readme): agrega seccion de despliegue"
```

## 6. Trabajo con ramas (intermedio)

### 6.1 Crear y cambiar ramas

```bash
git branch
git switch -c feature/login
# alternativa clasica
# git checkout -b feature/login
```

### 6.2 Integrar cambios

Fusionar rama feature en main:

```bash
git switch main
git pull origin main
git merge feature/login
```

Eliminar rama fusionada:

```bash
git branch -d feature/login
```

### 6.3 Resolver conflictos

Cuando hay conflicto:

1. Edita archivos marcados por Git.
2. Elige o combina cambios.
3. Marca como resuelto y confirma.

```bash
git add .
git commit -m "fix: resuelve conflictos en modulo auth"
```

## 7. Git remoto con GitHub

### 7.1 Conectar repo local a GitHub

```bash
git remote add origin https://github.com/usuario/mi-proyecto.git
git push -u origin main
```

### 7.2 Comandos remotos esenciales

```bash
git remote -v
git fetch origin
git pull origin main
git push origin feature/login
```

## 8. Flujo profesional con Pull Requests

Flujo recomendado:

1. Crear rama desde `main`.
2. Commits pequenos y claros.
3. Subir rama a GitHub.
4. Abrir Pull Request (PR).
5. Ejecutar CI (tests, lint, build).
6. Code review.
7. Merge a `main`.

Comandos:

```bash
git switch main
git pull origin main
git switch -c feature/pagos
# ... cambios
git add .
git commit -m "feat(payments): integra pasarela"
git push -u origin feature/pagos
```

## 9. Herramientas avanzadas de Git

### 9.1 Rebase para historial limpio

```bash
git switch feature/pagos
git fetch origin
git rebase origin/main
```

Si hay conflictos durante rebase:

```bash
git add .
git rebase --continue
# para abortar
git rebase --abort
```

### 9.2 Stash para guardar cambios temporales

```bash
git stash push -m "wip: formulario pagos"
git stash list
git stash pop
```

### 9.3 Cherry-pick para traer un commit puntual

```bash
git cherry-pick <hash-del-commit>
```

### 9.4 Revertir cambios de forma segura

```bash
git revert <hash-del-commit>
```

### 9.5 Etiquetas de version (releases)

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

## 10. Estrategia de ramas recomendada

Para equipos pequenos/medianos:

- `main`: solo codigo estable para produccion.
- `develop` (opcional): integracion previa.
- `feature/*`: nuevas funcionalidades.
- `hotfix/*`: correcciones urgentes de produccion.

Si quieres simplicidad total, usa GitHub Flow:

- `main` protegida + ramas `feature/*` + PR obligatorio + CI obligatorio.

## 11. De local a produccion: paso a paso

### 11.1 Preparar el proyecto local

- Tener tests automatizados.
- Tener linters/formatters.
- Variables de entorno separadas por ambiente.

### 11.2 Subir y versionar en GitHub

```bash
git add .
git commit -m "chore: prepara pipeline inicial"
git push origin main
```

### 11.3 Proteger rama principal en GitHub

En GitHub > Settings > Branches > Add branch protection rule para `main`:

- Require pull request before merging.
- Require status checks to pass.
- Require branches to be up to date.
- Restrict who can push (opcional).

Esto evita que llegue codigo roto a produccion.

## 12. Integracion continua (CI) con GitHub Actions

Crea el archivo:

- `.github/workflows/ci.yml`

Ejemplo base (Node.js):

```yaml
name: CI

on:
  pull_request:
    branches: ["main"]
  push:
    branches: ["main"]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint --if-present

      - name: Test
        run: npm test --if-present

      - name: Build
        run: npm run build --if-present
```

Que hace este pipeline:

- Se ejecuta en cada PR y push a `main`.
- Instala dependencias limpias.
- Corre lint, tests y build.
- Si algo falla, bloquea el merge (si proteges `main`).

## 13. Entrega continua / despliegue a produccion (CD)

Puedes desplegar automaticamente tras pasar CI.

Ejemplo conceptual de flujo:

1. Merge a `main`.
2. GitHub Actions corre CI.
3. Si todo pasa, job de deploy se ejecuta.
4. Se publica en proveedor (AWS, Azure, GCP, Vercel, Render, etc.).

Ejemplo simplificado agregando job deploy:

```yaml
name: CI-CD

on:
  push:
    branches: ["main"]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
      - run: npm ci
      - run: npm test --if-present

  deploy:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: echo "Aqui va el comando real de deploy"
```

## 14. Manejo de secretos en GitHub Actions

Nunca subas claves al repo.

Usa Secrets:

- GitHub > Settings > Secrets and variables > Actions.
- Crear secretos como `API_TOKEN`, `DEPLOY_KEY`, etc.

Uso en workflow:

```yaml
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

## 15. Checklist real para llevar un proyecto a produccion

1. Configurar Git local (`user.name`, `user.email`).
2. Inicializar repo y primer commit.
3. Crear repo en GitHub y conectar remote.
4. Definir estrategia de ramas y PR.
5. Configurar branch protection en `main`.
6. Agregar pipeline CI (`.github/workflows/ci.yml`).
7. Asegurar lint + test + build en CI.
8. Gestionar secretos en GitHub.
9. Crear pipeline de deploy (CD) condicionado a CI exitoso.
10. Etiquetar releases con `git tag`.
11. Monitorear y abrir hotfix via ramas `hotfix/*`.

## 16. Comandos Git mas usados (resumen rapido)

```bash
# Config
git config --global user.name "Tu Nombre"
git config --global user.email "tu_correo@ejemplo.com"

# Inicio
git init
git clone <url>

# Estado e historial
git status
git log --oneline --graph --decorate --all

git diff

# Stage y commit
git add .
git commit -m "feat: mensaje"

# Ramas
git switch -c feature/x
git switch main
git branch
git merge feature/x

# Remoto
git remote add origin <url>
git fetch origin
git pull origin main
git push -u origin main

# Avanzado
git rebase origin/main
git stash push -m "wip"
git stash pop
git cherry-pick <hash>
git revert <hash>

# Versionado
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

## 17. Errores comunes y como evitarlos

- Hacer `git push -f` en ramas compartidas: evita hacerlo.
- Commits gigantes: mejor pequenos y atomicos.
- No actualizar tu rama antes del PR: usa `git fetch` y `rebase`/`merge`.
- Subir secretos al repositorio: usa `.gitignore` y GitHub Secrets.
- No tener CI: agrega al menos lint + test + build.

## 18. Recomendacion final de aprendizaje

Secuencia sugerida:

1. Domina ciclo basico: `status`, `add`, `commit`, `push`, `pull`.
2. Aprende ramas y PRs.
3. Aprende conflictos, `rebase`, `stash` y `revert`.
4. Implementa CI obligatoria en `main`.
5. Agrega CD y versionado con tags.

Con esto pasas de un uso basico de Git a un flujo profesional listo para produccion.
