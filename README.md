# Финал мегаолимпиады 2026

## Краткий обзор

С помощью Ansible была написана автоматизация развертывания production-ready окружений для запуска приложений в **docker compose**, влючающая в себя:
- Разворачивание и настройка менеджера репозиториев Nexus
- Разворачивание и настройка production-ready node VM
- Разворачивание и настройка централизованного мониторинга с применением инструментов **Prometheus**, **Grafana**

Последние строк разворачивания плейбука:

<img width="1291" height="672" alt="ansible_logs" src="https://github.com/user-attachments/assets/7b4a6c45-67f7-47da-ac76-6eaf9d70980c" />


## Ход работы

Реализована настройка пользователей, путем создания групп **developers** , **admins**:

Группе **admins** выдаются права на выполнение **sudo** без ввода пароля. __Публичные__ ssh ключи пользователей закинуты на хосты для доступа без пароля. Пользователи обеих групп добавлены в группу **docker** для доступ к управлению контейнерами без **sudo**.
Эта автоматизация описана в ролях *users* и *users_docker*.

<img width="575" height="67" alt="sudo_admin" src="https://github.com/user-attachments/assets/0e78cb7b-905c-46cf-b0d9-71a483bd3bd4" />

<img width="447" height="96" alt="docker_developers" src="https://github.com/user-attachments/assets/5e27bf1f-2923-4297-bbf0-b88551a4cc75" />

Также на все хосты установлен **docker** и **docker compose**. Добавлена возможность получать проксируемые образы через self-host **Nexus** репозиторий.
Роль для настройки docker окружения: *docker*
Роль для разворачивания Nexus: *nexus*

На все хосты установлены экспортеры метрик: **node-exporter** для экспорта метрик хоста и **cadvisor** для экспорта метрик контейнеров.
Роли: *hosts_monitoring*

На отдельный сервер устанавливается **Grafana** и **Prometheus**.
Роль: *server_monitoring*


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







