# Laboratorio: Análisis Estático de Código con SonarCloud y Python en GitHub Codespaces

## **1. Preparar el Repositorio**

### Crear el Repositorio
1. Crea un repositorio en GitHub para tu proyecto Python.
2. Incluye los siguientes archivos básicos:
   - **`main.py`**: Un script simple de Python.
   - **`requirements.txt`**: Lista de dependencias del proyecto.

### Configurar Codespaces
Crea un archivo `devcontainer.json` para configurar el entorno de desarrollo:

```json
{
    "name": "Python with SonarCloud",
    "image": "mcr.microsoft.com/devcontainers/python:3.10",
    "features": {
        "ghcr.io/devcontainers/features/docker-in-docker:2": {}
    },
    "customizations": {
        "vscode": {
            "extensions": [
                "SonarSource.sonarlint-vscode"
            ]
        }
    }
}
```

### Agregar `.gitignore`
Asegúrate de incluir un archivo `.gitignore` para evitar subir archivos innecesarios, como:

```
__pycache__/
*.pyc
*.pyo
```

---

## **2. Configurar SonarCloud**

### Crear un Proyecto en SonarCloud
1. Regístrate o inicia sesión en [SonarCloud](https://sonarcloud.io).
2. Conecta tu repositorio de GitHub y crea un nuevo proyecto.
3. Copia el token de acceso generado por SonarCloud.

### Configurar `sonar-project.properties`

Crea un archivo `sonar-project.properties` en la raíz del repositorio con el siguiente contenido:

```
sonar.projectKey=your_project_key
sonar.organization=your_organization
sonar.host.url=https://sonarcloud.io
sonar.login=your_sonarcloud_token
sonar.language=py
```

Reemplaza `your_project_key`, `your_organization` y `your_sonarcloud_token` con los valores correspondientes.

---

## **3. Configurar el Pipeline de GitHub Actions**

### Crear el Archivo de Workflow
Guarda el siguiente contenido en `.github/workflows/sonarcloud-analysis.yml`:

```yaml
defaults:
  run:
    working-directory: ./

name: SonarCloud Analysis

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  sonarcloud:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run SonarCloud analysis
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: |
          curl -sSL https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip > sonar-scanner.zip
          unzip sonar-scanner.zip -d $HOME
          export PATH=$HOME/sonar-scanner-4.8.0.2856-linux/bin:$PATH
          sonar-scanner
```

### Configurar Secretos en GitHub
1. Ve a la configuración del repositorio en GitHub.
2. Agrega un secreto llamado `SONAR_TOKEN` con el token de SonarCloud.

---

## **4. Usar Codespaces para Desarrollo**

### Iniciar Codespaces
1. Abre el repositorio en GitHub Codespaces.
2. Asegúrate de que el entorno incluye Python 3.10 y las extensiones necesarias.

### Probar con SonarLint

Usa la extensión **SonarLint** en VSCode para obtener retroalimentación local sobre problemas de código.

---

## **5. Ejecutar Análisis**

1. Realiza un commit y un push al repositorio para disparar el workflow.
2. Los resultados del análisis estarán disponibles en el tablero de SonarCloud.
