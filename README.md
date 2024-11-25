# Описание проекта
Развертывание шаблонного сервиса в рамках финальной работы курса «DevOps-инженер. Основы»

Репозиторий является рабочей площадкой на инфраструктуре с [проекта](https://github.com/avkoptev/infra.git)

# Развертывание
Для работы необходим развернутый сервер runners c [проекта](https://github.com/avkoptev/infra.git)

Настройки gitlab:
1. ssh на gitlab сервере работает через 2222 порт. Узнать пароль для первого захода в root можно командами:
```shell
sudo docker exec -t -i gitlab /bin/bash
```
```shell 
cat /etc/gitlab/initial_root_password | grep Password:
```

2. В Settings -> General -> Import and export settings ставим галочки на "Repozitory by Url" и "enabled" для "Allow migrating GitLab groups and projects by direct transfer". Импортируем приватно проект.

3. Добавляем в проект runner командой:
```shell
sudo docker exec -it gitlab-runner gitlab-runner register -n \
  --url "https://ci.warspoon.ru" \
  --registration-token REGISTRATION_TOKEN \
  --executor docker \
  --description "docker runners" \
  --docker-image "docker:24.0.5" \
  --docker-privileged \
  --docker-volumes "/certs/client"
  --tag-list "docker"
```
где REGISTRATION_TOKEN заменяем на токен проекта для runner

4. Устанавливаем ssh соединение между gitlab сервером и сервером с разворачиваемым проектом. 
Переходим в настройки Settings -> CI / CD -> Variables, нажимаем Add Variable. Добавим новую переменную со следующими значениями:
* Key: ANSIBLE_SSHKEY
* Value: {приватная часть ssh ключа}
* Flags: Protect variable и Expand variable reference
Нажимаем кнопку Add variable

5. Создаем приватный репозиторий docker с именем skillbox
Переходим в настройки Settings -> CI / CD -> Variables, нажимаем Add Variable. Добавим переменные со следующими значениями:
* Key: DOCKER_HUB
* Value: {приватный docker hub}
* Flags: Expand variable reference
  
Нажимаем кнопку Add variable

* Key: ACCESS_TOKEN
* Value: {пароль от него}
* Flags: Protect variable и Expand variable reference
  
Нажимаем кнопку Add variable


* Key: STAGE
* Value: {адрес тестового сервера}
* Flags: Expand variable reference
  
Нажимаем кнопку Add variable


* Key: PROD
* Value: {адрес продакт сервера}
* Flags: Expand variable reference
  
Нажимаем кнопку Add variable

6. Обновляем ansible/inventory под актуальные адреса
