# Часто Задаваемые Вопросы

* [Где взять примеры для деплоймента (Examples)](FAQ.MD#где-взять-примеры-для-деплоймента-examples-)
* [Мне нужен сервер базы данных, где его взять и как подключиться](FAQ.MD#мне-нужен-сервер-базы-данных-где-его-взять-и-как-подключиться-)
* [Мне нужно создать ещё одну БД, что делать ?](FAQ.MD#мне-нужно-создать-ещё-одну-бд-что-делать-)
* [Как добавить два образа в процесс ci\cd ?](FAQ.MD#как-добавить-два-образа-в-процесс-cicd-)
* [Зачем использовать тег kuber ?](FAQ.MD#зачем-использовать-тег-kuber-)
* [Как сделать ci\cd с одной ветки и как запретить сборку с другой ветки?](FAQ.MD#как-сделать-cicd-с-одной-ветки-и-как-запретить-сборку-с-другой-ветки)
* [Как мне игнорировать сборку некоторых бранчей ?](FAQ.MD#как-мне-игнорировать-сборку-некоторых-бранчей-)
* [Зачем нужны переменные деплоймента](FAQ.MD#зачем-нужны-переменные-деплоймента)
* [У меня ошибка error: flag docker-username is required](FAQ.MD#у-меня-ошибка-error-flag-docker-username-is-required)
* [Как мне задать client_max_body_size](FAQ.MD#как-мне-задать-client_max_body_size)


## Где взять примеры для деплоймента (Examples) ?
- Их можно взять в этом репозитории, директория [examples](../examples)

## Мне нужен сервер базы данных, где его взять и как подключиться ?
- Мы предоставляем наши сервера БД, плюсы в том что вам нет необходимости разварачивать свой сервер БД.
Более детально в разделе [База данных](DATABASE.MD)

## Мне нужно создать ещё одну БД, что делать ?
- Более подробно в разделе - [Нужно создать ещё одну базу данных](DATABASE.MD#нужно-создать-ещё-одну-базу-данных)

## Как добавить два образа в процесс ci\cd ?
- Для добавления ещё одного образа в проект, воспользуйтесь переменными, нужно создать дополнительную переменную для образа, 
например:

```
...

variables:
  DOCKER_DRIVER: overlay
  DOCKER_HOST: "tcp://localhost:2375"
  REPOSITORY_IMAGE: "${CI_REGISTRY}/${CI_PROJECT_PATH}:${CI_PIPELINE_IID}"
  REPOSITORY_IMAGE_NGINX: "${CI_REGISTRY}/${CI_PROJECT_PATH}:nginx-${CI_PIPELINE_IID}"
  REPOSITORY_IMAGE_EX_NGINX: "${CI_REGISTRY}/${CI_PROJECT_PATH}:exnginx-${CI_PIPELINE_IID}"
...
```
Где *REPOSITORY_IMAGE_NGINX* и *REPOSITORY_IMAGE_EX_NGINX* дополнительные образы с тегами *nginx-latest* и *exnginx-latest*

Подставить их в файл деплоймента можно с помощью переменных деплоймента 

```
  - sed -i -e "s@__REPOSITORY_IMAGE__@${REPOSITORY_IMAGE_EX_NGINX}@g" dp-back-nginx-ex.yml
  - kubectl apply -f dp-back-nginx-ex.yml
```
Где с переменной REPOSITORY_IMAGE_EX_NGINX мы подставляем значение в [переменную для деплоя](VARIABLES.MD#переменные-деплоймента) `__REPOSITORY_IMAGE__`



## Зачем использовать тег kuber ?

- Тег *kuber* обязателен в файле [.gitlab-ci.yml](CI-CD.MD). Он нужен для того что бы runner с помощью которого происходит деплоймент, понимал что мы используем его

```
project-deploy:
  tags:
  - kuber
  stage: deploy
...
```

## Как сделать ci\cd с одной ветки и как запретить сборку с другой ветки?

- Для этого в файле длстейджи добавьте опцию *only:* с именем бранча, например:

```
build:
  tags:
    - kuber
  stage: build
  only:
    - staging
...
```
Где *staging* имя бранча

## Как мне игнорировать сборку некоторых бранчей ?

- Для этого в файле [.gitlab-ci.yml](CI-CD.MD) укажите опцию *only:* с именем бранча, которые нужно собирать например:


```
build:
  tags:
    - kuber
  stage: build
  only:
    - staging
    - dev
    - ...
...
```
Где *staging* и *dev* имена бранчей. Но нужно учитывать то, что каждая сборка, перезатрет предыдущую, то есть если вы соберете ветку *staging* с её изменениями, а потом запустите сборку ветки *dev* то деплоймент ветки *staging* заменится веткой *dev*.

## Зачем нужны переменные деплоймента
- [Переменные деплоймента](VARIABLES.MD#переменные-деплоймента) нужны для того, что бы записать изменения в файл для деплоймента dp.yml и выполнить его.


## У меня ошибка error: flag docker-username is required
- Сгенерируйте [Deploy Token](PULL_SECRET.MD)



## Как мне задать client_max_body_size 
- client_max_body_size задается анотациями в [Ingress контроллере](KIND_INGRESS.MD)

Пример:
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: __CI_PROJECT_NAME__-ingress
  namespace: __NAMESPACE_NAME__
  annotations:
    ingress.kubernetes.io/ssl-redirect: "true"
    kubernetes.io/tls-acme: "true"
    certmanager.k8s.io/cluster-issuer: letsencrypt-prod
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/proxy-body-size: "100m"
spec:
  tls:
  - hosts:
...
```
