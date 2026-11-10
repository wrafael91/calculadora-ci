# calculadora-ci

![CI](https://github.com/wrafael91/calculadora-ci/actions/workflows/ci.yml/badge.svg)

Proyecto de práctica de la **Guía de ejercicios: Manejo de repositorios** del curso Integración Continua (Politécnico Grancolombiano).
Recorre el ciclo completo de un proyecto pequeño en Python: primer commit, ramas y conflictos, recuperación de versiones, pull requests, configuración por ambiente y pipeline de integración continua con GitHub Actions.

**Autor:** William Rafael Manchabajoy Paredes

## Cómo ejecutarlo

Requisitos: Python 3.12 o superior y pytest.

```bash
python -m pip install pytest
python -m pytest
```

Para cambiar de ambiente se usa la variable `APP_ENV` (`desarrollo` por defecto, `pruebas` o `produccion`):

```bash
# Git Bash, macOS o Linux
APP_ENV=produccion python -c "from config import cargar_config; print(cargar_config())"
```

```powershell
# PowerShell
$env:APP_ENV = "produccion"
python -c "from config import cargar_config; print(cargar_config())"
Remove-Item Env:APP_ENV
```

## Estructura

| Archivo | Contenido |
|---|---|
| `calculadora.py` | Funciones `sumar`, `restar`, `dividir`, `multiplicar` y `potencia` |
| `test_calculadora.py` | Pruebas de la calculadora |
| `config.py` | Configuración separada para desarrollo, pruebas y producción |
| `test_config.py` | Pruebas de la configuración por ambiente |
| `.github/workflows/ci.yml` | Pipeline de GitHub Actions con matriz de Python 3.12 y 3.13 |
| `.gitignore` | Excluye `__pycache__/`, `.pytest_cache/` y `.env` |

## Ejercicios y evidencias

Los ejercicios 4 y 6 son en parejas. Los hice de forma individual, en los dos papeles: abrí cada pull request y lo revisé y fusioné yo mismo.

| # | Ejercicio | Evidencia en el repositorio |
|---|---|---|
| 1 | Primer repositorio | Commit `Agrega calculadora con pruebas` y `.gitignore` |
| 2 | Ramas y conflicto | Commit de fusión `Fusiona multiplicar con dividir`, con las dos funciones conservadas (`git log --oneline --graph`) |
| 3 | Recuperar una versión | Commit `Ajusta sumar` y su reversión con `git revert`, sin reescribir el historial |
| 4 | Remoto y pull request | Pull request de `feature/potencia` con comentario de revisión, fusionado en `main` |
| 5 | Ambientes | `config.py`, `test_config.py` y la tabla de clasificación de abajo |
| 6 | Pipeline de CI | Pull request de `prueba/romper-ci`: primer check en rojo (falla `test_sumar`) y último check en verde tras el `git revert` |
| Reto 6 | Matriz de versiones | Pull request de `feature/matriz-ci` con checks en verde para Python 3.12 y 3.13 |

## Respuestas a las preguntas de la guía

### Ejercicio 1: ¿qué diferencia hay entre el directorio de trabajo, el área de preparación y el repositorio?

- **Directorio de trabajo:** los archivos tal como están en la carpeta. Los archivos nuevos aparecen como *untracked* hasta que Git empieza a seguirlos.
- **Área de preparación (staging):** con `git add` se eligen los cambios que entrarán en el próximo commit.
- **Repositorio:** con `git commit` esos cambios quedan guardados en el historial con un identificador único y se pueden recuperar en cualquier momento.

### Ejercicio 2: ¿por qué Git no eligió una de las dos versiones por su cuenta?

Porque las dos ramas modificaron la misma zona de `calculadora.py`. Git no puede saber si la intención era conservar una versión, la otra o ambas, así que marca el conflicto y deja la decisión al desarrollador. Si eligiera solo, podría descartar código sin que nadie lo notara.

### Ejercicio 3: ¿por qué se prefiere `git revert` a borrar commits cuando otras personas ya tienen el historial?

`git revert` crea un commit nuevo que deshace el cambio, sin modificar los commits existentes. Si se borran commits que otras personas ya descargaron, sus copias quedan desincronizadas con el remoto, aparecen conflictos y se puede perder trabajo. Con `revert` el historial queda completo: se ve el error y también su corrección.

### Ejercicio 4: si GitHub se cae hoy, ¿qué copias del proyecto siguen existiendo y qué tendría que hacer el equipo?

Git es distribuido: cada clon tiene el repositorio completo, con todo el historial y las ramas. Si GitHub se cae, siguen existiendo las copias locales de cada integrante, y desde cualquiera de ellas se puede reconstruir el proyecto. Se pierde temporalmente lo que vive solo en la plataforma: pull requests, comentarios, issues y la ejecución de GitHub Actions.

El equipo tendría que:
1. Seguir trabajando y haciendo commits en sus copias locales, porque Git no necesita conexión para eso.
2. Si necesitan compartir cambios, crear un repositorio en otro servicio (GitLab o Bitbucket), agregarlo con `git remote add` y subir todas las ramas con `git push --all`.
3. Cuando GitHub vuelva, sincronizar con `git pull` y `git push`.

### Ejercicio 5: clasificación de situaciones

| Situación | Ambiente |
|---|---|
| Una desarrolladora prueba una función nueva con datos ficticios | Desarrollo |
| El equipo QA ejecuta sus pruebas en un servidor compartido | Pruebas |
| El cliente y usuarios invitados evalúan una versión antes del lanzamiento | Entorno intermedio (staging o preproducción) |
| Un desarrollador copia datos reales de clientes a su portátil | **Mala práctica:** los datos reales solo deben estar en producción y están protegidos por la ley de protección de datos |
| Alguien edita un archivo directamente en el servidor que atiende a los usuarios | **Mala práctica:** el cambio no queda versionado ni pasa por pruebas, y se salta el flujo de integración continua |

### Ejercicio 6: ¿qué parte de lo que hizo a mano en el ejercicio 1 ahora hace el pipeline?

Ejecutar las pruebas con `python -m pytest`. En el ejercicio 1 había que correrlas a mano antes de cada commit. Ahora el pipeline lo hace solo en cada push y pull request hacia `main`, en un entorno limpio, con las dos versiones de Python de la matriz, y marca el pull request en rojo si algo falla. Así un error no llega a `main` aunque alguien olvide probar.
