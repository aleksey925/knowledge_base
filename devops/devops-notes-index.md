Заметки о gitlab ci
================

# Оглавление

- [Пример настройки CI](#Пример-настройки-CI)


<a name='Пример-настройки-CI'></a>
### Пример настройки CI

Для реализация непрерывной интеграции в `CI / CD` -> `Settings` -> `Variables`
необходимо создать следующие переменные окружения:

- DOCKER_AUTH_CONFIG (не защищенная и не маскируемая) - необходима для того, 
    чтобы работать с docker образами из приватных реппозитриев docker hub
- GITHUB_USER (не защищенная) - логин github аккаунта
- GITHUB_TOKEN (не защищенная) - токен для доступа к github аккаунту


Пример содержимого DOCKER_AUTH_CONFIG:

{
    "auths": {
        "registry.hub.docker.com": {
            "auth": "yWxleDkyNTowOTa3YjFmNi01YuhhLuizODUtYWE="
        }
    }
}

Значение для ключа auth генерируется командой:

```bash
echo -n "<docker-hub-login>:<docker-hub-logon-or-token>" | base64
```

Токен для docker hub можно создать по [ссылке](https://hub.docker.com/settings/security) 

После того как переменная `DOCKER_AUTH_CONFIG` создана, необходимо в `image` 
прописать корректный URL до docker образа. Ниже представлен шаблон на основании
которого нужно его формировать:

```
registry.hub.docker.com/<docker-hub-user>/<repo-name>:<tag>
```

Полезные ссылки описывающие доступ к приватным docker hub репозиториям: 

- https://docs.gitlab.com/ee/ci/docker/using_docker_images.html#define-an-image-from-a-private-container-registry
- https://stackoverflow.com/questions/51580858/cant-access-private-mysql-docker-image-from-gitlab-ci




CI_JOB_TOKEN переменная уже объявлена по умолчанию, генерировать токен самостоятельно не нужно.

https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
https://stackoverflow.com/questions/25409700/using-gitlab-token-to-clone-without-authentication/29570677#29570677
https://docs.gitlab.com/ee/user/project/new_ci_build_permissions_model.html#dependent-repositories


Клонирование реппозитрия при помощи токена
https://stackoverflow.com/questions/52445475/git-pull-with-access-token-git-push-with-username-password


GITHUB_USER and GITHUB_TOKEN - переменную нельзя помечать protected

https://aleksey925:95b606dbeba172a8ab34e7c73832f9aa288f3103@github.com/aleksey925/tets.git