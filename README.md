# Занятие .Автоматизация администрирования.

## Задача


Что нужно сделать?

Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере используя Ansible необходимо развернуть nginx со следующими условиями:

    Необходимо использовать модуль yum/apt;
    Конфигурационные файлы должны быть взяты из шаблона jinja2 с перемененными;
    После установки nginx должен быть в режиме enabled в systemd;
    Должен быть использован notify для старта nginx после установки;
    Сайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible.


## Решение задачи

### 1.  Создадим структуру папок 

Структура Папок будет следующая:
<pre>
 Корень
 |
 |- ansible.cfg
 |- Nginx.yaml
 |
 |-inventory
           |-host.ini
|
|-services
            |-Nginx
                   |-Nginx.conf.j2
</pre>
ansible.cfg - Будет содержать настройки анисбла - Где расположен инвенторий и отключена проверка ssh ключей
Nginx.yml - Плейбук для ansibla

hosts.ini -  файл содержит список серверов для применения настроек
Nginx.conf.j2 - шаблон настроек для Nginx


### 2. Описание Плейбука

```bash
---
- name: Nginx install to server # Общее название
  hosts: Nginx    # Хосты к ктороым таски ниже будут применены
  become: true    # Запуск от имени root
  vars:           # Переменные
    nginx_listen_port: 8080 # Параметр
  tasks:
    - name: Install system updates for Rpm-Based           # Наимнование задания
      ansible.builtin.dnf:                                 # Имя подключаемого Модуль ансибла 
        name: 'nginx'                                      # Наименование пакета
        Install system updates for Rpm-Based state: latest # Параметр указывающий на установку последней версии
        update_cache: true                                 # Проверка обновления кэша пакетов в репозитории
        cache_valid_time: 3600                             # Время которое кэш пакетов будет считаться обновленным
      when:                                                # Условие выполенния
        - ansible_os_family == "RedHat"
        - ansible_distribution == "CentOS"

    - name: "Install system updates for ubuntu systems"    # Наимнование задания
      ansible.builtin.apt:                                 # Имя подключаемого Модуль ансибла
        name: 'nginx'                                      # Наименование пакета
        state: latest                                      # Параметр указывающий на установку последней версии
      when:                                                # Условие выполенния
        - ansible_os_family == "Debian"
        - ansible_distribution == "Ubuntu"

    - name: Ensure Nginx is started and enabled            #  Наимнование задания
      ansible.builtin.systemd:                             #  Имя подключаемого Модуль ансибла
        name: nginx                                        #  Наименование службы
        state: started                                     #  Состояние службы
        enabled: true                                      #  Включение службы при перезапуске системы

    - name: "Nginx file"                                   # Наимнование задания
      ansible.builtin.template:                            # Имя подключаемого Модуль ансибла
        src: services/Nginx/nginx.conf.j2                  # Местоложжение шаблона
        dest: /etc/nginx/nginx.conf                        # Местоположение куда его необходимо скопировать на сервере ктороый мы настриваем
        mode: preserve                                     # Режим обработки ( обязательный - если файл пристуствует он будет перезаписан)
      notify:
        - Reload Nginx
  handlers:
    - name: Restart Nginx                                  # Наимнование задания
      ansible.builtin.systemd:                             # Имя подключаемого Модуль ансибла
        name: nginx                                        # Наименование службы
        state: restarted                                   # Выполненение перезапуска службы
        enabled: true                                      # Включение службы при перезапуске системы
    - name: Reload Nginx                                   # Наимнование задания
      ansible.builtin.systemd:                             # Имя подключаемого Модуль ансибла
        name: nginx                                        # Наименование службы
        state: reloaded                                    # Выполненение перезапуска службы ( перечитать свои настройки )
```


### 3. Выполненеие

```bash
git clone https://github.com/jecka2/Ansible.git
cd Ansible
ansible-playbook Nginx.yaml
  jecka   ~/Documents/Ansible git-[ main]-  ansible-playbook Ngnix.yaml

PLAY [Nginx install to server] *****************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************
[WARNING]: Platform linux on host nginx is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of
that path. See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [nginx]

TASK [Install system updates for Rpm-Based] ****************************************************************************************************************************************************
skipping: [nginx]

TASK [Install system updates for ubuntu systems] ***********************************************************************************************************************************************
changed: [nginx]

TASK [Ensure Nginx is started and enabled] *****************************************************************************************************************************************************
ok: [nginx]

TASK [Nginx file] ******************************************************************************************************************************************************************************  jecka   ~/Documents/Ansible git-[ main]-  ansible-playbook Ngnix.yaml

PLAY [Nginx install to server] *****************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************
[WARNING]: Platform linux on host nginx is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of
that path. See https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [nginx]

TASK [Install system updates for Rpm-Based] ****************************************************************************************************************************************************
skipping: [nginx]

TASK [Install system updates for ubuntu systems] ***********************************************************************************************************************************************
changed: [nginx]

TASK [Ensure Nginx is started and enabled] *****************************************************************************************************************************************************
ok: [nginx]

TASK [Nginx file] ******************************************************************************************************************************************************************************
changed: [nginx]

RUNNING HANDLER [Reload Nginx] *****************************************************************************************************************************************************************
changed: [nginx]

PLAY RECAP *************************************************************************************************************************************************************************************
nginx                      : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
changed: [nginx]

RUNNING HANDLER [Reload Nginx] *****************************************************************************************************************************************************************
changed: [nginx]

PLAY RECAP *************************************************************************************************************************************************************************************
nginx                      : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

```

После выполненя  мы можем увидеть, что  nginx установлен и принемает запросы на порту 8080

```bash
agrant@nginx:~$ curl http://192.168.11.150:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```