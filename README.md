﻿# Итоговый проект

В проекте собраны **ansible** и **docker** файлы, необходимые для выполнения итогового проета SF. Также добавлены некоторые файлы конфигурации приложений.
Тестирование и установка сервисов проводилась для серверов Ubuntu20.04 (YandexCloud)

### Файлы
#### -/app_configs/openvpn/ - конфиги openvpn-сервера
#### -/compose/ - файлы dicker-compose для утстановки zabbix&grafana, elastic&kibana
#### -playbooks - playbooks файлы для базовой установки  сервисов ( apache и nginx, php, bind9, postfix и dovecot, filebeat, PostgreSQL12, Elasticsearch и Kibana )
#### -ansible.cfg - базовый 
#### -hosts - пример файла

## Использование ansible
Для прокатки playbook нужно:
1) заполнить файл hosts
2) разрешить выполнение sudo на сервере-клиенте: 
				
		$ sudo visudo
	добавить строчку в файл

		%sudo ALL=(ALL)       NOPASSWD: ALL
3) настроить ssh доступ для пользователя  [ansible_user] на сервере-клиенте
4) выполнить команду 

		ansible-playbook ./playbooks/[playbook].yml -u [ansible_user]

** перед прокаткой ролей необходимо просмотреть/скорректировать файлы и переименовать [файлы_example] в [файлы]:
 - ./playbooks/templates/clientvpn01.conf.j2_example (только для openvpn_playbook.yml)
 - ./playbooks/roles/[role_name]/{templates,vars}/[file].yml


## Использование docker
Каталог compose содержит два  docker-compose файла:
- docker-compose_zabbix_grafana.yml
- docker-compose_ek.yml

Для запуска сервисов в докере: 
1) установить docker и docker-compose
2) добавить пользователя от которого будем запускать контейнеры в группу docker

### docker-compose_zabbix_grafana.yml
Данный docker-compose файл создает 2 контейнера: 
- zabbix-server5.0.36 с поддержкой внешнего mysql-сервера
- grafana-server7.5.17 с установкой плагина alexanderzobnin-zabbix-app 4.1.5

Для запуска контейнеров, нужно выполнить команду: 

	docker-compose -f docker-compose_zabbix_grafana.yml up

Перед запуском контейнера zabbix необходимо:
- подготовить mysql-сервер для внешнего подключения пользователя zabbix@'%' 
- если нет существующей базы (в проекте она была ), то  [создать базу zabbix,  импортировать схему и данные, подготовить фронтэнд для сервера zabbix](https://www.zabbix.com/download?zabbix=5.0&os_distribution=ubuntu&os_version=20.04&components=server_frontend_agent&db=mysql&ws=apache) согласно инструкции
- переименовать файл zabbix.env.example в zabbix.env, заполнить значения 

      DB_SERVER_HOST=my_mysql_server_ip
	  MYSQL_USER=zabbix
	  MYSQL_PASSWORD=passwd

Перед запуском контейнера grafana необходимо:
- создать каталог ./grafana с правами xrw группы root
- переименовать файл grafana.env.example в grafana.env заполнить значения

      GF_SECURITY_ADMIN_USER=admin
	  GF_SECURITY_ADMIN_PASSWORD=passwd

### docker-compose_ek.yml
Данный docker-compose файл создает 2 контейнера: 
- elasticsearch:7.17.1 с конфигурацией безопасности - minimal security
- kibana:7.17.1

Для запуска контейнеров, нужно выполнить команду:

	docker-compose -f docker-compose_ek.yml up

Перед запуском контейнера elasticsearch необходимо: 
- создать каталог ./elastic/data с правами xrw группы root
- в файле ./elastic.env заполнить значения:
 
		ELASTIC_PASSWORD=mypass
- в файле ./kibana/kibana.yml заполнить значения:

      elasticsearch.password: mypass
      elasticsearch.hosts: ["http://10.129.0.12:9200"]

```
