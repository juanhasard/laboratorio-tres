# Laboratorio 3 - Juan Pablo Morales Hasard

API NestJS desplegada en Docker Desktop Kubernetes. El endpoint `/lab` devuelve
AMBIENTE desde un ConfigMap y API_KEY desde un Secret con un valor ficticio.

## Archivos de entrega

- `Dockerfile` y `.dockerignore`: construccion de la imagen.
- `entrega.yaml`: Namespace, ConfigMap, Secret, Deployment con dos replicas y Service.
- `Jenkinsfile`: comandos de install, test, build, push y deploy.
- `agent.yaml`: plantilla del agente Kubernetes con Node.js, kubectl y BuildKit.
- `jenkins-rbac.yaml`: cuenta jenkins-agentes y permisos dentro del namespace.
- `evidencias/`: salidas y capturas de la ejecucion; incluir el log completo de Jenkins.

## Despliegue inicial

Ejecutar desde este directorio con el contexto de Kubernetes del laboratorio:

```bash
kubectl config current-context
kubectl apply -f entrega.yaml
kubectl apply -f jenkins-rbac.yaml
kubectl rollout status deployment/app-juan-pablo-morales-hasard -n ns-juan-pablo-morales-hasard
kubectl port-forward svc/svc-juan-pablo-morales-hasard 8081:80 -n ns-juan-pablo-morales-hasard
```

En otra terminal: `curl http://localhost:8081/lab`. Se usa 8081 porque Jenkins
ocupa 8080. API_KEY es ficticia: el endpoint y las evidencias exponen su valor.

## Pipeline Jenkins

El cloud se llama `kubernetes-lab3` y usa el namespace
`ns-juan-pablo-morales-hasard`. Su credencial Kubernetes debe estar vigente.
Guardar los tokens de registros como credenciales Username with password:

| ID | Usuario | Password |
| --- | --- | --- |
| dockerhub-lab3 | jpmorales2985 | Token Docker Hub con lectura y escritura |
| ghcr-lab3 | juanhasard | Token GitHub classic con write:packages |

Crear una tarea Pipeline con Pipeline script from SCM, SCM Git, repositorio
`https://github.com/juanhasard/laboratorio-tres.git`, la rama publicada y Script
Path `Jenkinsfile`. Si el repositorio es privado, agregar una credencial de lectura.
No aplicar `agent.yaml` con kubectl: Jenkins lo lee para crear el pod temporal.
El agente necesita acceso a Internet. BuildKit se ejecuta como usuario 1000,
con los ajustes de seccomp, AppArmor y proceso necesarios para el constructor
rootless dentro del pod. Esta configuracion esta destinada al laboratorio local.

El pipeline instala dependencias, ejecuta pruebas unitarias y e2e, y compila
NestJS en build. En push, BuildKit construye y publica las etiquetas
`juan-pablo-morales-hasard` y `3.0.0` en ambos registros:

- `jpmorales2985/entrega_juan_pablo_morales`
- `ghcr.io/juanhasard/entrega_juan_pablo_morales`

Se sigue la estructura del ejemplo de clase:
https://github.com/carlosmarind/curso-contenedores/blob/main/Jenkinsfile
y la seleccion de herramientas de Jenkinsfile.v2. Los comandos estan dentro
del Jenkinsfile y la plantilla externa se llama agent.yaml, como pide la tarea.
El ejemplo completo construye y publica con buildctl-daemonless.sh; aqui se
mantiene ese mecanismo y se personalizan registros, etiquetas y recursos.

Dos diferencias respecto al profesor aprovechan la configuracion ya realizada:
las credenciales de registros vienen de Jenkins y se escriben temporalmente
en un config.json compartido con BuildKit, eliminado al terminar push; kubectl
usa la cuenta jenkins-agentes montada en el pod, sin withKubeConfig ni otro plugin.
El archivo de credenciales no esta en el repositorio ni en el contexto de build.

El stage deploy actualiza la imagen del Deployment existente y reinicia sus
pods, ya que la etiqueta del nombre se reutiliza. `imagePullPolicy: Always` en
entrega.yaml permite descargar la imagen publicada. Luego espera el rollout y
comprueba que `/lab` responde con ambas variables configuradas.
Los cambios de ConfigMap, Secret o Service en entrega.yaml deben aplicarse
manualmente con `kubectl apply -f entrega.yaml` antes de ejecutar el pipeline.

## Evidencias pendientes de recopilar

Guardar el Console Output completo de la ejecucion real como
`evidencias/pipeline-jenkins.txt`, con `Finished: SUCCESS`, y descargar los
artefactos que Jenkins archiva desde `evidencias/pipeline/`.
La preparacion de estos archivos no acredita una ejecucion exitosa del pipeline.

Capturar ademas cluster-info, nodos, pods, Deployment, Service, logs, printenv,
ConfigMap, Secret y la consulta con port-forward y curl solicitados en la tarea.
En los comandos de recursos del laboratorio, usar siempre el namespace
`ns-juan-pablo-morales-hasard`. No incluir tokens de acceso en las evidencias.
El PDF del enunciado esta excluido por .gitignore y no forma parte de la entrega.

## Referencia del proyecto NestJS

<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="120" alt="Nest Logo" /></a>
</p>

[circleci-image]: https://img.shields.io/circleci/build/github/nestjs/nest/master?token=abc123def456
[circleci-url]: https://circleci.com/gh/nestjs/nest

  <p align="center">A progressive <a href="http://nodejs.org" target="_blank">Node.js</a> framework for building efficient and scalable server-side applications.</p>
    <p align="center">
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/v/@nestjs/core.svg" alt="NPM Version" /></a>
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/l/@nestjs/core.svg" alt="Package License" /></a>
<a href="https://www.npmjs.com/~nestjscore" target="_blank"><img src="https://img.shields.io/npm/dm/@nestjs/common.svg" alt="NPM Downloads" /></a>
<a href="https://circleci.com/gh/nestjs/nest" target="_blank"><img src="https://img.shields.io/circleci/build/github/nestjs/nest/master" alt="CircleCI" /></a>
<a href="https://discord.gg/G7Qnnhy" target="_blank"><img src="https://img.shields.io/badge/discord-online-brightgreen.svg" alt="Discord"/></a>
<a href="https://opencollective.com/nest#backer" target="_blank"><img src="https://opencollective.com/nest/backers/badge.svg" alt="Backers on Open Collective" /></a>
<a href="https://opencollective.com/nest#sponsor" target="_blank"><img src="https://opencollective.com/nest/sponsors/badge.svg" alt="Sponsors on Open Collective" /></a>
  <a href="https://paypal.me/kamilmysliwiec" target="_blank"><img src="https://img.shields.io/badge/Donate-PayPal-ff3f59.svg" alt="Donate us"/></a>
    <a href="https://opencollective.com/nest#sponsor"  target="_blank"><img src="https://img.shields.io/badge/Support%20us-Open%20Collective-41B883.svg" alt="Support us"></a>
  <a href="https://twitter.com/nestframework" target="_blank"><img src="https://img.shields.io/twitter/follow/nestframework.svg?style=social&label=Follow" alt="Follow us on Twitter"></a>
</p>
  <!--[![Backers on Open Collective](https://opencollective.com/nest/backers/badge.svg)](https://opencollective.com/nest#backer)
  [![Sponsors on Open Collective](https://opencollective.com/nest/sponsors/badge.svg)](https://opencollective.com/nest#sponsor)-->

## Description

[Nest](https://github.com/nestjs/nest) framework TypeScript starter repository.

## Project setup

```bash
$ pnpm install
```

## Compile and run the project

```bash
# development
$ pnpm run start

# watch mode
$ pnpm run start:dev

# production mode
$ pnpm run start:prod
```

## Run tests

```bash
# unit tests
$ pnpm run test

# e2e tests
$ pnpm run test:e2e

# test coverage
$ pnpm run test:cov
```

## Deployment

When you're ready to deploy your NestJS application to production, there are some key steps you can take to ensure it runs as efficiently as possible. Check out the [deployment documentation](https://docs.nestjs.com/deployment) for more information.

If you are looking for a cloud-based platform to deploy your NestJS application, check out [Mau](https://mau.nestjs.com), our official platform for deploying NestJS applications on AWS. Mau makes deployment straightforward and fast, requiring just a few simple steps:

```bash
$ pnpm install -g @nestjs/mau
$ mau deploy
```

With Mau, you can deploy your application in just a few clicks, allowing you to focus on building features rather than managing infrastructure.

## Resources

Check out a few resources that may come in handy when working with NestJS:

- Visit the [NestJS Documentation](https://docs.nestjs.com) to learn more about the framework.
- For questions and support, please visit our [Discord channel](https://discord.gg/G7Qnnhy).
- To dive deeper and get more hands-on experience, check out our official video [courses](https://courses.nestjs.com/).
- Deploy your application to AWS with the help of [NestJS Mau](https://mau.nestjs.com) in just a few clicks.
- Visualize your application graph and interact with the NestJS application in real-time using [NestJS Devtools](https://devtools.nestjs.com).
- Need help with your project (part-time to full-time)? Check out our official [enterprise support](https://enterprise.nestjs.com).
- To stay in the loop and get updates, follow us on [X](https://x.com/nestframework) and [LinkedIn](https://linkedin.com/company/nestjs).
- Looking for a job, or have a job to offer? Check out our official [Jobs board](https://jobs.nestjs.com).

## Support

Nest is an MIT-licensed open source project. It can grow thanks to the sponsors and support by the amazing backers. If you'd like to join them, please [read more here](https://docs.nestjs.com/support).

## Stay in touch

- Author - [Kamil Myśliwiec](https://twitter.com/kammysliwiec)
- Website - [https://nestjs.com](https://nestjs.com/)
- Twitter - [@nestframework](https://twitter.com/nestframework)

## License

Nest is [MIT licensed](https://github.com/nestjs/nest/blob/master/LICENSE).
