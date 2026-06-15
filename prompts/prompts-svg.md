# Prompts

herramienta: opencode
modelo: DeepSeek V4 Flash Free OpenCode Zen - medium


## Prompt 1
´´´text
/init
´´´

---

## Prompt 2
´´´text
Generate a prompt in the folder @prompts/ where I want to generate a brand new skill for a devops senior engineer, expert en github actions and aws with the best practives. Do not include the actual skill, only generate the best prompt possible to be exectued to achive the goal
´´´

---

## Prompt 3
´´´text
execute @prompts/prompts-generate-devops-skill.md
´´´

---

## Prompt 4
´´´text
/devops-pipeline crea un fichero pipeline.yml en @.github/workflows/ para que se dispare con un push a una rama con un pull request abierto que se encargue de pasar los tests de backend
´´´

---

## Prompt 5
´´´text
/devops-pipeline haz lo mismo en el fichero pipeline-test.yml pero para hacer pruebas y ver que se ejecuta. En esta ocasión, debe ejectuarse cada vez que hago push a la rama feature en la que estoy desarrollando. Solo debe encargarse de pasar los tests de backend
´´´

---

## Prompt 6
´´´text
! [remote rejected] feature/pipeline-svg -> feature/pipeline-svg (refusing to allow a Personal Access Token to create or update workflow `.github/workflows/pipeline-test.yml` without `workflow` scope)
error: failed to push some refs to 'https://github.com/svg80/AI4Devs-pipeline-202603SRII.git' 

´´´

---

## Prompt 7
´´´text
test
Node.js 20 actions are deprecated. The following actions are running on Node.js 20 and may not work as expected: actions/cache@v4, actions/checkout@v4, actions/setup-node@v4. Actions will be forced to run with Node.js 24 by default starting June 16th, 2026. Node.js 20 will be removed from the runner on September 16th, 2026. Please check if updated versions of these actions are available that support Node.js 24. To opt into Node.js 24 now, set the FORCE_JAVASCRIPT_ACTIONS_TO_NODE24=true environment variable on the runner or in your workflow file. Once Node.js 24 becomes the default, you can temporarily opt out by setting ACTIONS_ALLOW_USE_UNSECURE_NODE_VERSION=true. For more information see: https://github.blog/changelog/2025-09-19-deprecation-of-node-20-on-github-actions-runners/
´´´

---

## Prompt 8
´´´text
/devops-pipeline modifica el fichero @.github/workflows/pipeline.yml para añadir un paso encargado de generar el build del backend
´´´

---

## Prompt 9
´´´text
/devops-pipeline el nombre del job no es descriptivo con lo que hace. Separa en dos jobs, uno para test y otro para build. Además incluye un paso inicial de lint en el backend para detectar que no hay errores de sintaxis y que todo es correcto antes de pasar tests
´´´

---

## Prompt 10
´´´text
/devops-pipeline modifica el fichero @.github/workflows/pipeline.yml para indicar que el on pull request solo debe ejecutarse cuando sea contra la rama main. Además añade otro on para que se ejectue en el push de cualquier rama, de manera que pueda eliminar .github/workflows/pipeline-test.yml
´´´

---

## Prompt 11
´´´text

´´´

---

## Prompt 12
´´´text

´´´

---

## Prompt 13
´´´text

´´´

---

## Prompt 14
´´´text

´´´

---

