# Финал мегаолимпиады 2026

## Краткий обзор

С помощью Ansible была написана автоматизация развертывания production-ready окружений для запуска приложений в **docker compose**, влючающая в себя:
- Разворачивание и настройка менеджера репозиториев Nexus
- Разворачивание и настройка production-ready node VM
- Разворачивание и настройка централизованного мониторинга с применением инструментов **Prometheus**, **Grafana**

## Ход работы

Реализована настройка пользователей, путем создания групп **developers** , **admins**:

Группе **admins** выдаются права на выполнение **sudo** без ввода пароля. Пользователи обеих групп добавлены в группу **docker** для доступ к управлению контейнерами без **sudo**.
Эта автоматизация описана в ролях *users* и *users_docker*.

<img width="575" height="67" alt="sudo_admin" src="https://github.com/user-attachments/assets/0e78cb7b-905c-46cf-b0d9-71a483bd3bd4" />

<img width="447" height="96" alt="docker_developers" src="https://github.com/user-attachments/assets/5e27bf1f-2923-4297-bbf0-b88551a4cc75" />

На все хосты установлены экспортеры метрик: **node-exporter** для экспорта метрик хоста и **cadvisor** для экспорта метрик контейнеров.
Роли: *hosts_monitoring*

Также на все хосты установлен **docker** и **docker compose**. Добавлена возможность получать проксируемые образы через self-host **Nexus** репозиторий.
Роль для настройки docker окружения: *docker*
Роль для разворачивания Nexus: *nexus*

На отдельный сервер устанавливается **Grafana** и **Prometheus**.
Роль: *server_monitoring*

В графану был импортирован дашборд для мониторинга системы по метрикам cadvisor:

![Uploading grafana_dashborad.png…]()


## Использование



