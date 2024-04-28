# Plantilla de Github acitons y Docker

## Descripción del proyecto

**Plantilla prototipo** que sirve para generar el **versionamiento semantico** automatico de un **proyecto de consola** implementando **Github Actions**, siguiendo el standar de flujo de trabajo de **GitFlow**. De igual manera se configura para **construir** una imagen de Docker del proyecto (generar previamente el archivo Dockerfile),**logearse** a la cuenta de Dockerhub y **publicar** la imagen de Docker en **Dockerhub**.

## Estado del proyecto

<a href="https://github.com/nqtronix/git-template/blob/master/badges.md#project-status"><img src="https://img.shields.io/badge/status-active-brightgreen.svg" alt="status: active"></a>

## Características

1. Crea veriones automaticas en base a la implementación de:
    - **Github actions**.
    - **GitFlow**.
    - **Versionado semantico** (v1.0.0).
2. `Funcionalidad 2`: Iniciar sesión en Dockerhub.
3. `Funcionalidad 3`: Consturir imagen de Docker.
4. `Funcionalidad 4`: Publicar imagen en Dockerhub.

## Pasos de implementación

1. Generar un repositorio remoto y trabajar en base al flujo de **Gitflow**.
2. Clonar el repositorio remoto.
3. Generar una rama **feature** y trabajar sobre ella para los siguientes pasos.
4. Generar archivo principal de la aplicación "app.js" que contendra el código de ejemplo que solicite algún dato por consola:

    ~~~bash
    const readline = require("readline");

    const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
    });

    rl.question("Enter your name: ", (name) => {
    console.log(`Hello, ${name}!. How are you?`);
    rl.close();
    });
    ~~~

5. Generar el archivo **Dockerfile** que contendra las instrucciones basicas para ejecutar un programa por consola:

    ~~~bash
    FROM node:lts-alpine3.19
    WORKDIR /app
    COPY app.js ./
    CMD [ "node", "app.js" ]
    ~~~

    >[!NOTE]
    >
    >Tener en cuenta que si la aplicación requiere dependencias o alguna configuraión extra para el su funcionamiento correcto, se debera modificar el archivo Dockerfile para poder constuir la imagen de la aplicación correctamente.

6. Generar un archivo .github/workfloww/plantilla.yml que contendra los siguientes pasos:
    1. Revisar repositorio:

        ~~~bash
        - name: Review code
            uses: actions/checkout@v4
            with:
                ref: ${{ github.event.pull_request.merge_commit_sha }}
                fetch-depth: "0"
        ~~~

        > [!IMPORTANT]
        >
        >Agregar el postfijo correspondiente a la modificación realizada en el proyecto "**major | minor | patch**" en el **commit** del **Pull Request** hacia la rama **main**, ya que este especifica que número de version aumentara con respecto al **versionado semantico**

    2. Aumento de version y creación del tag:

        ~~~bash
        - name: Bump version and push tag
        id: github_tag
        uses: anothrNick/github-tag-action@v1
        env:
          GITHUB_TOKEN: ${{secrets.TOKEN_SUPER }}
          WITH_V: true
        ~~~

        > [!IMPORTANT]
        >
        >Generar previamente el secret __TOKEN_SUPER_ en **Gihub** para **Actions** y hacer referencia en **env**.

    3. Iniciar sesión en DockerHub:

        ~~~bash
        - name: Login to DockerHub
        env:
          DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
          DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
        run: |
          docker login --username $DOCKERHUB_USERNAME --password $DOCKERHUB_TOKEN
        ~~~

        > [!IMPORTANT]
        >
        >- Contar con una cuenta en Dockerhub.
        >- Generar secrets para el usuario y contraseña/token de Dockerhub.

    4. Construir imagen de Docker:

        ~~~bash
        - name: Build Docker image
        env:
          NEW_VERSION: ${{ steps.github_tag.outputs.new_tag }}
        run: |
          docker build --tag miguetheking/template_github_actions:$NEW_VERSION .
          docker build --tag miguetheking/template_github_actions:latest .
        ~~~

        > [!IMPORTANT]
        >
        >- Crear previamente el archivo Dockerfile con las configuraciones necesarias para crear la imagen.

    5. Publicar imagen en DockerHub:

        ~~~bash
        - name: Publish image to Dockerhub
        env:
          NEW_VERSION: ${{ steps.github_tag.outputs.new_tag }}
        run: |
          docker push miguetheking/template_github_actions:$NEW_VERSION
          docker push miguetheking/template_github_actions:latest
        ~~~

        > [!IMPORTANT]
        >
        >- Crear previamente un repositorio en Dockerhub, en el cual se estara publicando las nuevas versiones de la imagen.

7. Subir los archivos creados al repositorio remoto y realizar el pull requeste a la rama **main**. En automatico se tendria que realizar el **versionado**, creación del **tag**, consturcción y publicación de imagen en **DockerHub**

## Recursos útilies

- [GitHub Actions](https://docs.github.com/es/actions) - Saber más sobre la implementación de **Actions**
- [GitHub Tag Bump](https://github.com/marketplace/actions/github-tag-bump) - Aumentar número de version y generar tag automaticamente.
- [GitHub Checkout](https://github.com/marketplace/actions/checkout) - Comprueba el repositorio para que el flujo de trabajo pueda acceder a él.
- [GitHub Secret](https://docs.github.com/es/actions/security-guides/using-secrets-in-github-actions) - Generación de secrets.
- [Docker](https://www.docker.com/#build) - Creación de **imagenes Docker**.
- [DockerHub](https://hub.docker.com/) - Respositorio de **imagenes** en **Docker**.

## Autores

[miguelalonsocoba](https://github.com/miguelalonsocoba)
