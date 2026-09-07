# Guia practica de Git y GitHub

Referencia reutilizable para iniciar repositorios, trabajar con ramas, revisar
cambios y colaborar mediante pull requests. No es un script para ejecutar de
principio a fin: elige la seccion correspondiente a tu tarea.

Los ejemplos usan `main` como rama base y `origin` como remoto. Sustituye
`OWNER`, `REPOSITORY`, `BRANCH`, `FILE` y `PR_NUMBER` por los valores reales.
Las convenciones de nombres, commits y merge deben adaptarse a cada equipo.

## Conceptos esenciales

| Concepto | Significado |
| --- | --- |
| Working tree | Archivos con los que trabajas localmente |
| Staging o index | Contenido seleccionado para el siguiente commit |
| Commit | Registro de una version del contenido preparado |
| Branch | Referencia a un commit que avanza al crear nuevos commits |
| Remote | Conexion con otro repositorio, identificado por un nombre y URL |
| Upstream | Rama de seguimiento usada por comandos como status, pull y push |
| Pull request (PR) | Propuesta de integracion y espacio de revision en GitHub |
| Milestone | Agrupacion de issues y PRs por objetivo en GitHub |

Crear un commit no publica archivos. Publicar una rama no la integra en `main`.
Crear una rama tampoco crea ni vincula automaticamente un milestone.

## Comprobar herramientas e identidad

```bash
git --version
gh --version
git config --get user.name
git config --get user.email
gh auth status
```

Git administra el historial; GitHub CLI (`gh`) interactua con GitHub.
El nombre y correo del commit identifican al autor y pueden quedar publicos.
No determinan la cuenta que autentica un push por SSH o HTTPS.

Si necesitas configurar una identidad solo para el repositorio actual:

```bash
git config --local user.name "Your Name"
git config --local user.email "you@example.com"
```

`--local` requiere un repositorio inicializado. Usa un correo verificado en
GitHub o el correo `noreply` de tu cuenta si quieres asociar los commits a ella.

## Iniciar o clonar un repositorio

Para una carpeta local que aun no tiene Git:

```bash
git init -b main
git status --short --branch
```

Para obtener un repositorio existente, usa esta alternativa desde la carpeta
que lo contendra:

```bash
git clone https://github.com/OWNER/REPOSITORY.git
```

Clonar descarga el historial y configura `origin`. No necesitas ejecutar
`git init` dentro de un repositorio clonado.

## Excluir archivos locales

Define las exclusiones en `.gitignore` antes de preparar archivos. Por ejemplo:

```gitignore
.DS_Store
.env
.env.*
!.env.example
node_modules/
```

Adapta estas reglas al proyecto. Un archivo de ejemplo de variables de entorno
debe contener valores ficticios, nunca credenciales reales.

Para investigar por que se ignora un archivo no rastreado:

```bash
git check-ignore -v -- FILE
```

`.gitignore` no deja de rastrear archivos ya versionados ni elimina secretos
del historial. Si se expone una credencial, revocala o rotala; borrarla en un
commit posterior no basta.

## Inspeccionar cambios

```bash
git status --short --branch
git diff
git diff --cached
git diff --cached --stat
git diff --cached -- FILE
```

- `status` muestra rama, archivos pendientes y estado del staging.
- `diff` muestra cambios no preparados en archivos rastreados.
- `diff --cached` muestra lo preparado para el siguiente commit.
- `--stat` resume archivos y volumen de cambios.
- `-- FILE` limita la salida a una ruta.

Los archivos nuevos no rastreados no aparecen en `git diff`: revisalos tambien.
El contenido de archivos binarios debe comprobarse con una herramienta adecuada.

## Preparar y crear commits

Selecciona archivos concretos y revisa toda la seleccion antes del commit:

```bash
git add -- FILE
git diff --cached --stat
git diff --cached
git commit -m "[DOCS] Add setup instructions"
git status --short --branch
```

`git add` captura el contenido actual. Si editas despues, debes agregar de
nuevo el archivo para incluir esos cambios. Agregar un directorio prepara los
cambios elegibles dentro de el; evita hacerlo sin revisar su contenido.

Un commit incluye todos los archivos preparados, no solo el ultimo agregado.
Ejecuta las validaciones relevantes del proyecto antes de registrarlo.

Para retirar un archivo del staging sin borrar tus cambios, cuando ya existe
al menos un commit:

```bash
git restore --staged -- FILE
```

No confundas este comando con `git restore -- FILE`, que descarta cambios
locales no preparados del archivo.

### Mensajes de commit

Usa un mensaje breve que describa el cambio y conserva la convencion del equipo.
Dos formatos posibles, que no conviene mezclar sin acuerdo:

| Proposito | Prefijo con corchetes | Conventional Commits |
| --- | --- | --- |
| Mantenimiento | `[CHORE] Update dependencies` | `chore: update dependencies` |
| Funcionalidad | `[FEAT] Add health endpoint` | `feat: add health endpoint` |
| Correccion | `[FIX] Handle shutdown timeout` | `fix: handle shutdown timeout` |
| Pruebas | `[TEST] Cover invalid input` | `test: cover invalid input` |
| Documentacion | `[DOCS] Explain configuration` | `docs: explain configuration` |

Git acepta ambos formatos. Conventional Commits facilita integraciones con
herramientas de changelog y versionado automatico.

## Conectar con GitHub

Comprueba la autenticacion y, si hace falta, inicia sesion:

```bash
gh auth status
gh auth login
```

### Crear un repositorio remoto nuevo

Desde un repositorio local con al menos un commit:

```bash
gh repo create OWNER/REPOSITORY --private --source=. --remote=origin --push
```

Usa `--public` en lugar de `--private` solo si quieres publicar el contenido.
El comando crea el repositorio, agrega `origin` e intenta hacer push. Si falla
parcialmente, comprueba que se creo antes de repetirlo.

### Conectar un remoto existente

Si el repositorio remoto ya existe y aun no tienes `origin`:

```bash
git remote add origin https://github.com/OWNER/REPOSITORY.git
```

Si `origin` ya existe y necesitas cambiar su URL:

```bash
git remote set-url origin https://github.com/OWNER/REPOSITORY.git
```

Comprueba la configuracion y publica la rama inicial cuando corresponda:

```bash
git remote -v
git push -u origin main
```

`-u` establece el seguimiento de la rama remota. Si el remoto ya contiene
historial, inspeccionalo antes de intentar integrarlo; no uses force-push para
resolver un rechazo. Si `main` esta protegida, utiliza una rama y un PR.

## Resolver diferencias de autenticacion

GitHub CLI y Git por SSH pueden utilizar cuentas distintas. Si GitHub deniega
el push a una cuenta inesperada, revisa:

```bash
gh auth status
git remote -v
ssh -T git@github.com
```

El saludo de SSH indica a que cuenta pertenece la clave aceptada. GitHub no
ofrece acceso shell, por lo que la comprobacion puede terminar con codigo 1
incluso si la autenticacion funciona. En la primera conexion, verifica la
huella del servidor con la documentacion oficial antes de aceptarla.

Una alternativa a configurar claves SSH por cuenta es usar HTTPS con la cuenta
activa de GitHub CLI. Confirma primero que esa cuenta tiene acceso al remoto:

```bash
git remote set-url origin https://github.com/OWNER/REPOSITORY.git
git config --local credential.https://github.com.helper ""
git config --local --add credential.https://github.com.helper '!gh auth git-credential'
git push -u origin BRANCH
```

La entrada vacia descarta helpers heredados para ese host. La siguiente usa
GitHub CLI. La configuracion afecta solo al repositorio actual y no guarda el
token en archivos versionados. No es un paso necesario en cada push.

## Crear una rama de trabajo

Antes de cambiar de rama, revisa los cambios pendientes y decide donde deben
registrarse. Con el working tree limpio:

```bash
git switch main
git pull --ff-only origin main
git switch -c BRANCH
git push -u origin BRANCH
```

`pull --ff-only` descarga y actualiza la rama solo si puede avanzar sin crear
un merge. Si hay divergencia, se detiene para que puedas investigarla.
`switch -c` crea una rama; para volver a una existente, usa `git switch BRANCH`.

Ejemplos de nombres, segun la unidad de trabajo:

```text
feature/health-endpoint
fix/shutdown-timeout
docs/setup-guide
milestone/01-http-foundation
```

## Publicar cambios y abrir un pull request

Despues de crear y validar tus commits en la rama de trabajo:

```bash
git push
gh pr create --base main --head BRANCH --title "Add HTTP foundation" --body "Describe scope, decisions, and validation results."
```

`git push` usa el seguimiento configurado bajo la configuracion habitual de
Git. Si no existe seguimiento, usa `git push -u origin BRANCH`.
Sustituye el titulo y cuerpo del ejemplo por una descripcion real.

Para revisar un PR:

```bash
gh pr view PR_NUMBER
gh pr diff PR_NUMBER
gh pr checks PR_NUMBER
```

Que no existan checks no significa que las pruebas hayan pasado. Revisa el
diff, las validaciones y las conversaciones antes de integrar.

## Integrar y limpiar ramas

Si el equipo usa merge commits y las reglas del repositorio lo permiten:

```bash
gh pr merge PR_NUMBER --merge
```

Un merge commit conserva los commits individuales. Squash los combina en uno;
rebase los reaplica sobre la base. Elige el metodo acordado por el equipo.
La proteccion de historial lineal no es compatible con merge commits.

Tras confirmar que el PR se integro, con el working tree limpio:

```bash
git switch main
git pull --ff-only origin main
git branch -d BRANCH
git fetch --prune origin
```

`branch -d` elimina la rama local con una comprobacion de integracion. Si se
niega, revisa el historial; no cambies automaticamente a `-D`. Con squash o
rebase puede requerirse una comprobacion adicional de los cambios integrados.

`fetch --prune` actualiza referencias remotas y elimina las obsoletas; no borra
ramas locales ni ramas en GitHub. Si la rama remota ya no se necesita y no se
elimino automaticamente, puedes borrarla explicitamente:

```bash
git push origin --delete BRANCH
```

Antes de borrarla, comprueba que el trabajo este integrado y que nadie la use.

## Consultar el historial

```bash
git log --oneline -10
git log --oneline --graph --decorate --all
git show --stat HEAD
```

`HEAD` identifica el commit actual. Las referencias como `origin/main`
representan el ultimo estado remoto conocido localmente, no una consulta en
tiempo real a GitHub. Usa `git fetch origin` para actualizarlas sin integrar
cambios en tu rama.

## Checklist antes de publicar

- Revisar rama actual, staging y diff completo.
- Verificar que no se incluyan secretos ni datos privados, tampoco en commits anteriores.
- Ejecutar las pruebas y validaciones que correspondan al cambio.
- Confirmar el remoto, la cuenta y la visibilidad del repositorio.
- Respetar las reglas de PR y proteccion de la rama base.
- No usar comandos destructivos ni force-push como solucion automatica a errores.
