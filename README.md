## Antes de empezar

Crea una cuenta en Vercel

## Configuración de Vercel

Estando en la consola, en la raíz del proyecto que queramos desplegar, escribimos lo siguiente:

```js
  npx vercel link
```

Esto te creará una carpeta `.vercel` y dentro un fichero `project.json`. Dentro de este fichero estarán el campo `projectId` y `orgId`

Lo siguiente será desactivar Github para Vercel, así no subirá automáticamente a Vercel los cambios sobre Github. Para ello, crea un fichero `vercel.json` en la raíz del proyecto y pon lo siguiente:

```js
{
  "github": {
    "enabled": false,
    "silent": true
  }
}
```

Queda una cosita más, hay que generar un Token en Vercel para poder hacer los deploys correctamente:

> vercel.com/dashboard => settings => tokens => create token

Una vez creado, copia el token y ponlo en un lugar seguro para usarlo después

## Github Secrets

Lo primero que vamos a hacer aquí será crear `secrets`. Estas variables nos servirán como variables de entorno.

> settings => secrets => new repository secret

Crea los siguientes secretos:

- VERCEL_ORG_ID: valor de `orgId` del fichero `project.json`
- VERCEL_PROJECT_ID: valor de `projectId` del fichero `project.json`
- VERCEL_TOKEN: el token que creamos en la página de Vercel

## Github Actions

En la raíz de tu proyecto, crea la carpeta `.github` y dentro la carpeta `workflows`. Dentro de esta última carpeta crea un fichero `.yml` con el nombre con el que quieras identificar el entorno al que quieras subir tu proyecto (el nombre será solo para que tú mismo u otros identifiquen el propósito de dicho workflow)

> .github/workflows/staging.yml

- name: lo que quieras, será el nombre de la acción
- on: será el evento que lance la acción. Este puede ser `push`, `pull_request`, `tag`, etc, sobre alguna de las ramas existentes del proyecto
- jobs/build: aquí es donde está la chicha, aquí vendrá el sistema sobre el que correrá la acción y los diferentes pasos que se irán sucediendo.

```yml
name: staging, build, vercel

on:
  pull_request:
    branches:
      - master
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
```

## Step para subir a Vercel el proyecto

Opciones del comando de Vercel

- confirm: evita que te pregunte si quieres linkar el proyecto (linkado anteriormente) a Vercel
- prod: promociona las subidas a producción, estas versiones subidas irán siempre a refrescar el mismo dominio en lugar de generar uno diferente cada vez
- n: nombre del proyecto (project id)
- t: token del proyecto (vercel token)
- scope: ámbito del proyecto (org id)
- b: variable establecida en tiempo de compilación (build)
- e: variable establecida en tiempo de ejecución (runtime)

Ejemplo de step:

```yml
- name: deploy to Vercel
  id: branch_deploy_vercel
  run: |
    URL=$(npx vercel --prod --confirm -n ${{ secrets.VERCEL_PROJECT_ID }} -t ${{ secrets.VERCEL_TOKEN }} --scope ${{ secrets.VERCEL_ORG_ID }} -b DEPLOY_ENV=staging -e DEPLOY_ENV=staging)
```

- Más referencias sobre Vercel CLI: https://vercel.com/docs/cli
