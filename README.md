# Финал мегаолимпиады 2026
**(!) Эта версия README была дописана после окончания олимпиады, из-за того, что не успел дописать часть информации. (!)**

[Оригинальный README](https://github.com/GoldenJaden/gusev.y-itmo-megaschool-devops-2026/blob/main/README_old.md)

## Краткий обзор

С помощью Ansible была написана автоматизация развертывания production-ready окружений для запуска приложений в **docker compose**, влючающая в себя:
- Разворачивание и настройка менеджера репозиториев Nexus
- Разворачивание и настройка production-ready node VM
- Разворачивание и настройка централизованного мониторинга с применением инструментов **Prometheus**, **Grafana**

Последние n строк разворачивания плейбука:

<img width="1291" height="672" alt="ansible_logs" src="https://github.com/user-attachments/assets/7b4a6c45-67f7-47da-ac76-6eaf9d70980c" />

## Ход работы

### 2. Контроль доступа пользователей
Роль [*users*](https://github.com/GoldenJaden/gusev.y-itmo-megaschool-devops-2026/tree/main/playbook/roles/users/tasks): создание групп **developers** и **admins**:

Группе **admins** выдаются права на выполнение **sudo** без ввода пароля. 

__Публичные__ ssh ключи пользователей закинуты на хосты для доступа без ввода пароля.

Пользователи и группы, необходимые к созданию, описываются в файле переменных ([playbook/group_vars/all.yml](https://github.com/GoldenJaden/gusev.y-itmo-megaschool-devops-2026/blob/main/playbook/group_vars/all.yml)).

Можно гибко добавлять новых пользователей и групп, дописывая их в списки **"user_groups"** и **"users"**.

<img width="575" height="67" alt="sudo_admin" src="https://github.com/user-attachments/assets/0e78cb7b-905c-46cf-b0d9-71a483bd3bd4" />

<img width="447" height="96" alt="docker_developers" src="https://github.com/user-attachments/assets/5e27bf1f-2923-4297-bbf0-b88551a4cc75" />

### 3. Контейнеризация
Роль [*nexus*](https://github.com/GoldenJaden/gusev.y-itmo-megaschool-devops-2026/tree/main/playbook/roles/nexus): разворачивание платформы **Nexus**. Эта платформа позволит проксировать образы с DockerHub.

Следующие 2 роли были раскатаны на все хосты, так как все инструменты в этой работе устанавливаются в контейнеризованном виде с помощью docker compose для изоляции компонентов.

Роль [*docker*](https://github.com/GoldenJaden/gusev.y-itmo-megaschool-devops-2026/tree/main/playbook/roles/docker): установка  **docker** и **docker compose**.

Роль [*users_docker*](https://github.com/GoldenJaden/gusev.y-itmo-megaschool-devops-2026/tree/main/playbook/roles/users_docker/tasks): пользователи групп **developers** и **admins** добавлены в группу **docker** для доступа к управлению контейнерами без **sudo**.

### 4. Мониторинг
Роль [*hosts_monitoring*](https://github.com/GoldenJaden/gusev.y-itmo-megaschool-devops-2026/tree/main/playbook/roles/hosts_monitoring): установка инструментов для экспорта метрик: **node-exporter** - выгрузка метрик хоста; **cadvisor** - выгрузка метрик контейнеров. Раскатана на все хосты, для включения всех хостов в мониторинг.

Роль [*server_monitoring*](https://github.com/GoldenJaden/gusev.y-itmo-megaschool-devops-2026/tree/main/playbook/roles/server_monitoring): установка систем мониторинга и визуализации **Prometheus** и **Grafana**. 

Реализовано включение новых VM в систему мониторинга без её перезагрузки при помощи механизма [**file_sd_config**](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#file_sd_config), позволяющего подкладывать файлы с новыми таргетами для **Prometheus**. Эти файлы он перечитывает раз в n секунд, поэтому перезагрузка не требуется.

В графану был импортирован дашборд для мониторинга системы по метрикам cadvisor:

<img width="2767" height="1222" alt="grafana_dashborad" src="https://github.com/user-attachments/assets/07feedaa-e7c0-4c2f-ad03-6783de49cf1a" />

По метрикам были настроены алерты внутри Grafana:
- Контейнер пропал
- У контейнера высокая утилизация RAM (>80%)

На рисунках ниже была протестирована работоспособность первого алерта (убили контейнер -> алерт загорелся -> подняли контейнер -> алерт потух)

<img width="807" height="483" alt="alerts_test" src="https://github.com/user-attachments/assets/844cab65-c904-4cc6-8da7-3f8cf6224115" />

<img width="946" height="417" alt="alerts_normal" src="https://github.com/user-attachments/assets/47f772ce-b842-442a-8e19-79ee1c3440cc" />


## Использование

Вначале необходимо развернуть Nexus.
`ansible-playbook nexus.yml`

Далее нужно настроить репозиторий для проксирования:

<img width="2075" height="1126" alt="nexus" src="https://github.com/user-attachments/assets/b8330e14-4672-454d-8ab2-ba6c6856e732" />

<img width="776" height="747" alt="nexus_repo" src="https://github.com/user-attachments/assets/9275e043-9c77-4338-9fbb-6fb8cf0250e7" />

Дальше можно вызывать основной плейбук для настройки всех нод и мониторинга:
`ansible-playbook playbook.yml`








