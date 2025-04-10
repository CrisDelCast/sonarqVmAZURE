# 📘 Guía de Integración de SonarQube en una VM y CI/CD con GitHub Actions

## 📌 Introducción

Esta guía explica cómo desplegar SonarQube en una máquina virtual (VM), configurarlo y utilizarlo dentro de un pipeline de CI/CD con GitHub Actions para analizar un proyecto.

![image](https://github.com/user-attachments/assets/43299d2b-b731-4b88-98d9-5adf66896bb6)


##  Paso 1: Configurar SonarQube en la VM

### Instalar Docker y Docker Compose
Ejecuta los siguientes comandos para instalar Docker y Docker Compose:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose
sudo systemctl start docker
sudo systemctl enable docker
```

![image](https://github.com/user-attachments/assets/adc18dd7-e9db-4506-a66d-0022426fe6e5)


### Configurar y ejecutar SonarQube

Crea un archivo `docker-compose.yml` en tu VM:

```yaml
version: '3'
services:
  sonarqube:
    image: sonarqube:lts
    container_name: sonarqube
    restart: always
    ports:
      - "9000:9000"
    environment:
      - SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
```

Ejecuta el siguiente comando para iniciar SonarQube:

```bash
docker-compose up -d
```

![image](https://github.com/user-attachments/assets/cfe3ae1e-6d08-4ab2-85e1-f2ae2fb479b6)


Accede a SonarQube en `http://<IP_DE_LA_VM>:9000/` con las credenciales predeterminadas:

- Usuario: `admin`
- Contraseña: `admin`



##  Paso 2: Configurar un Token de Autenticación en SonarQube

1. Ve a **My Account** en SonarQube.
2. Dirígete a la pestaña **Security**.
3. Genera un nuevo **token** con permisos de análisis.
4. Guarda el token en **GitHub Secrets** (`SONAR_TOKEN`).
![image](https://github.com/user-attachments/assets/a7981bc9-39f2-484d-8a1c-ddfa40e6e104)

## 🔧 Paso 3: Configurar GitHub Actions para la Integración con SonarQube

Crea un archivo `.github/workflows/ci-sonar.yml` en tu repositorio con el siguiente contenido:

```yaml
name: CI/CD Pipeline con SonarQube

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout del código
        uses: actions/checkout@v3

      - name: Configurar Java y Maven
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'
          cache: 'maven'

      - name: Compilar y ejecutar pruebas
        run: mvn clean verify

      - name: Ejecutar análisis con SonarQube
        run: |
          mvn sonar:sonar \
            -Dsonar.projectKey=nombre-proyecto \
            -Dsonar.host.url=${{ secrets.SONAR_HOST_URL }} \
            -Dsonar.login=${{ secrets.SONAR_TOKEN }}
```

![image](https://github.com/user-attachments/assets/af029954-7c65-4623-a50c-ee460157136d)


## Paso 4: Verificar los Resultados en SonarQube

Después de ejecutar el pipeline, dirígete a **SonarQube** y verifica los resultados del análisis de código.

![image](https://github.com/user-attachments/assets/fa1ee6be-10af-46bd-bf36-c6dcc3d5f510)

## Conclusión

Siguiendo estos pasos, has integrado SonarQube en una máquina virtual y lo has configurado para analizar automáticamente el código en un pipeline de CI/CD con GitHub Actions.

