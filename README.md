# Инструкция по Ansible
Ansible — это инструмент для автоматизации управления конфигурациями, развертывания приложений и оркестрации. Он использует SSH для подключения к хостам и не требует установки агентов на управляемых машинах.

# 1. Установка Ansible
#### Установка на Linux
Если вы используете Linux, выполните следующие команды в терминале, чтобы установить Ansible с помощью менеджера пакетов вашего дистрибутива:
#### Ubuntu и Debian:
```bash
$ sudo apt-get update
$ sudo apt-get install ansible
# Или установка последней версии через PPA
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```
#### CentOS и Red Hat:
```bash
$ sudo yum update
$ sudo yum install ansible
```
#### Установка на macOS
```bash
$ brew update
$ brew install ansible
```
#### Установка с помощью pip (универсальный способ)
```bash
$ pip3 install ansible
# Или с virtualenv (рекомендуется)
$ python3 -m venv ansible-env
$ source ansible-env/bin/activate
$ pip install ansible
```
Проверка установки `$ ansible --version`

# 2. Настройка инвентаря `Inventory`
Ansible использует инвентарь для определения хостов и групп хостов, на которых будут выполняться задачи.
#### Создание инвентаря
Для создания инвентаря создайте файл с именем `inventory.ini` в любом месте на вашем компьютере и добавьте следующие строки:
```ini
[webservers]
web1.example.com
web2.example.com

[database]
db1.example.com
```
#### Переменные инвентаря
Кроме перечисления хостов, в инвентаре можно использовать переменные для настройки подключения к хостам. Для этого можно определить переменные для каждой группы хостов или для отдельных хостов
Например, вы можете определить переменную `ansible_user` для каждого хоста, чтобы указать имя пользователя, с которым нужно подключаться к хосту
```ini
[webservers]
web1.example.com ansible_user=ubuntu
web2.example.com ansible_user=root

[database]
db1.example.com ansible_user=dbadmin
```
Также вы можете определить общие переменные для всех хостов в инвентаре, создав файл `group_vars/all.yml` и определив переменные в нем:
```yml
---
ansible_ssh_private_key_file: /path/to/private_key
```
Это позволит указать путь к файлу с приватным ключом SSH, который будет использоваться при подключении к хостам

# 3. Создание плейбуков `Playbooks`
Плейбуки - это файлы YAML, которые содержат инструкции по выполнению задач на хостах. Каждый плейбук может содержать одну или несколько задач, которые Ansible будет выполнять последовательно на каждом хосте
#### Создание плейбука
Для создания плейбука создайте файл с именем `playbook.yml` в любом месте на вашем компьютере и добавьте следующий код:
```yml
---
- name: Описание плейбука (опционально, но рекомендуется)
  hosts: webservers           # Целевая группа хостов
  become: true                # Использовать sudo (опционально)
  gather_facts: true          # Собирать информацию о хостах (по умолчанию true)
  vars:                       # Переменные плейбука
    http_port: 80
    server_name: example.com

  tasks:
    - name: Установка Nginx
      apt:
        name: nginx
        state: present
        update_cache: true

    - name: Запуск Nginx
      service:
        name: nginx
        state: started
        enabled: true
```
Этот плейбук выполняет задачу установки `Nginx` на все хосты из группы `webservers`. Атрибут become указывает на то, что Ansible должен использовать привилегии суперпользователя для выполнения задач
#### Выполнение плейбука
Для выполнения плейбука используйте команду `ansible-playbook` и передайте ей имя файла плейбука:
```bash
# Базовая команда
$ ansible-playbook -i inventory.ini playbook.yml

# С запросом пароля sudo
$ ansible-playbook -i inventory.ini playbook.yml --ask-become-pass

# Проверка синтаксиса
$ ansible-playbook --syntax-check playbook.yml

# Просмотр, что изменится (dry-run)
$ ansible-playbook --check playbook.yml

# Пошаговое выполнение
$ ansible-playbook --step playbook.yml

# С ограничением на конкретные хосты
$ ansible-playbook -i inventory.ini playbook.yml --limit web1.example.com

# С подробным выводом
$ ansible-playbook -i inventory.ini playbook.yml -v   # -vvvv для максимальной детализации
```
Ansible будет выполнять плейбук на каждом хосте из группы `webservers`, установив `Nginx` на каждом хосте

# 4. Использование модулей `Modules`
Ansible предоставляет множество модулей, которые можно использовать для выполнения различных задач на хостах. Например, с помощью модуля `copy` можно копировать файлы на хосты, а с помощью модуля `service` можно управлять службами на хостах
#### Использование модуля `copy`
Для копирования файла на удаленный хост используйте модуль `copy`. В следующем примере мы скопируем файл `index.html` на все хосты из группы `webservers`:
```yml
---
- hosts: webservers
  become: true
  tasks:
    - name: Копирование файла index.html
      copy:
        src: /path/to/local/index.html
        dest: /var/www/html/index.html
```
Этот плейбук выполняет задачу копирования файла `index.html` с локального хоста на удаленные хосты из группы `webservers`

#### Использование модуля `service`
Для управления службами на удаленном хосте используйте модуль `service`. В следующем примере мы будем перезапускать службу `Nginx` на всех хостах из группы `webservers`:
```yml
---
- hosts: webservers
  become: true
  tasks:
    - name: Перезапуск службы Nginx
      service:
        name: nginx
        state: restarted
```
Этот плейбук выполняет задачу перезапуска службы `Nginx` на всех хостах из группы `webservers`

# 5. Использование обработчиков `Handler`
Handlers выполняются только один раз в конце плейбука, если были оповещены задачами.
```yml
---
- name: Настройка Nginx
  hosts: webservers
  become: true

  tasks:
    - name: Копирование конфигурации
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - reload nginx
        - test nginx config   # Несколько обработчиков

    - name: Копирование сайта
      copy:
        src: index.html
        dest: /var/www/html/index.html
      notify: reload nginx

  handlers:
    - name: test nginx config
      command: nginx -t
      listen: "reload nginx"   # Группировка обработчиков

    - name: reload nginx
      service:
        name: nginx
        state: reloaded
      listen: "reload nginx"
```

# 6. Использование ролей `Roles`
Роли позволяют организовать плейбуки и задачи в более структурированном виде, что делает их более читабельными и управляемыми. Роль - это набор плейбуков, задач и переменных, которые можно использовать для выполнения определенной функции на хостах.
#### Структура роли
```bash
myrole/
├── defaults/          # Переменные по умолчанию (низкий приоритет)
│   └── main.yml
├── files/             # Статические файлы
│   └── index.html
├── handlers/          # Обработчики
│   └── main.yml
├── meta/              # Метаданные роли (зависимости)
│   └── main.yml
├── tasks/             # Основные задачи (обязательно)
│   └── main.yml
├── templates/         # Jinja2 шаблоны
│   └── config.conf.j2
├── tests/             # Тесты роли
│   ├── inventory
│   └── test.yml
└── vars/              # Высокоприоритетные переменные
    └── main.yml
```
#### Создание роли
Для создания роли используйте команду `ansible-galaxy`:
```bash
$ ansible-galaxy init myrole
```
Затем создайте файл `nginx/tasks/main.yml` и добавьте следующий код:
```yml
---
- name: Установить Nginx
  apt:
    name: nginx
    state: present
    update_cache: true

- name: Настройка конфигурации Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
  notify: Перезапуск службы Nginx
```
Этот файл определяет две задачи: первая задача устанавливает `Nginx` с помощью модуля `apt`, а вторая задача копирует файл конфигурации `nginx.conf.j2` в `/etc/nginx/nginx.conf` на удаленных хостах с помощью модуля `template`. Затем задача оповещает модуль `service` о том, что необходимо перезапустить службу `Nginx`, используя оповещение `notify`
#### Использование роли в плейбуке
Для использования роли в плейбуке добавьте раздел `roles` и укажите имя роли. В следующем примере мы используем роль `nginx` в плейбуке:
```yml
---
- hosts: webservers
  become: true
  roles:
    - nginx
```
Этот плейбук выполнит задачи из роли `nginx` на всех хостах из группы `webservers`.

# 7. `Ansible-Galaxy`
#### Создание роли
```yml
# Создание структуры роли
$ ansible-galaxy role init myrole

# Поиск ролей в Ansible Galaxy
$ ansible-galaxy role search nginx

# Установка роли из Galaxy
$ ansible-galaxy role install geerlingguy.nginx

# Установка из requirements.yml
$ ansible-galaxy role install -r requirements.yml
```
#### Пример файла `requirements.yml`:
```yml
---
roles:
  - name: geerlingguy.nginx
    src: https://github.com/geerlingguy/ansible-role-nginx.git
    version: 3.0.0
  - name: geerlingguy.mysql
    version: 3.0.0
```
#### Использование роли в плейбуке
```yml
---
- name: Настройка веб-серверов
  hosts: webservers
  become: true

  roles:
    - role: geerlingguy.nginx
      vars:
        nginx_listen_port: 8080
    - role: mycustomrole

  # Задачи после ролей
  tasks:
    - name: Дополнительная настройка
      copy:
        src: custom.conf
        dest: /etc/nginx/conf.d/custom.conf
```

# 8. Переменные и факты `Facts`
#### Приоритет переменных (от низкого к высокому)
* role defaults
* inventory group_vars
* inventory host_vars
* playbook vars
* playbook vars_files
* playbook vars_prompt
* playbook vars (ключи)
* set_fact (во время выполнения)
* command line --extra-vars (самый высокий)
#### Сбор фактов о хостах
```yml
- name: Сбор информации о хостах
  hosts: all
  gather_facts: true   # По умолчанию true

  tasks:
    - name: Вывод информации
      debug:
        msg: |
          Hostname: {{ ansible_hostname }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          CPU: {{ ansible_processor_count }} x {{ ansible_processor_cores }} cores
          Memory: {{ ansible_memtotal_mb }} MB
          IP: {{ ansible_default_ipv4.address }}
```
#### Использование set_fact
```yml
- name: Установка переменной во время выполнения
  set_fact:
    app_version: "{{ app_version | default('1.0.0') }}"
    is_production: "{{ 'production' in inventory_hostname }}"
```

# 9. Условные операторы и циклы
#### Условия `when`
```yml
- name: Установка пакета только на Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_distribution == "Ubuntu"

- name: Установка на CentOS
  yum:
    name: nginx
    state: present
  when: ansible_distribution == "CentOS"

- name: Команда только если переменная определена
  command: /usr/bin/special_script
  when: special_var is defined and special_var

- name: Условие с несколькими критериями
  debug:
    msg: "Production server"
  when:
    - inventory_hostname in groups['production']
    - ansible_distribution_version >= "20.04"
```
#### Циклы
```yml
- name: Установка нескольких пакетов
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - curl
    - htop
  # Или с переменными:
  # loop: "{{ packages }}"

- name: Создание нескольких пользователей
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups | default('users') }}"
  loop:
    - { name: 'john', groups: 'sudo' }
    - { name: 'jane', groups: 'www-data' }
    - { name: 'bill' }

- name: Словари с dict2items
  debug:
    msg: "{{ key }} = {{ value }}"
  loop: "{{ {'a': 1, 'b': 2, 'c': 3} | dict2items }}"
  vars:
    key: "{{ item.key }}"
    value: "{{ item.value }}"
```

# 10. Шаблоны `Jinja2`
#### Базовые конструкции
```j2
{# Комментарий #}

# Переменные
ServerName {{ server_name }}
Port {{ nginx_port | default(80) }}

# Условия
{% if ssl_enabled %}
    listen 443 ssl;
    ssl_certificate {{ ssl_cert_path }};
    ssl_certificate_key {{ ssl_key_path }};
{% else %}
    listen 80;
{% endif %}

# Циклы
{% for domain in domains %}
    server_name {{ domain }};
{% endfor %}

# Фильтры
{{ ansible_memtotal_mb | filesizeformat }}        # 7.9 GB
{{ "Hello World" | lower }}                       # hello world
{{ config_file | basename }}                      # nginx.conf
{{ config_path | dirname }}                       # /etc/nginx

# Конкатенация
{{ "prefix_" + app_name }}
{{ ["/var", "log", app_name] | join('/') }}
```
#### Пример шаблона `nginx.conf.j2`
```j2
user {{ nginx_user }};
worker_processes {{ nginx_worker_processes }};
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections {{ nginx_worker_connections }};
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    keepalive_timeout {{ nginx_keepalive_timeout }};

    {% if nginx_vhosts %}
    {% for vhost in nginx_vhosts %}
    server {
        listen {{ vhost.listen_port | default(80) }};
        server_name {{ vhost.server_name }};
        root {{ vhost.root }};
        index {{ vhost.index | default('index.html index.htm') }};

        location / {
            try_files $uri $uri/ =404;
        }
    }
    {% endfor %}
    {% endif %}
}
```

# 11. Шифрование секретов `Vault`
Ansible Vault позволяет шифровать чувствительные данные.

#### Основные команды
```bash
# Создание зашифрованного файла
$ ansible-vault create secrets.yml

# Редактирование зашифрованного файла
$ ansible-vault edit secrets.yml

# Шифрование существующего файла
$ ansible-vault encrypt variables.yml

# Расшифровка файла
$ ansible-vault decrypt secrets.yml

# Просмотр содержимого
$ ansible-vault view secrets.yml

# Смена пароля
$ ansible-vault rekey secrets.yml
```
#### Использование `vault` в плейбуках
```bash
# Выполнение с вводом пароля
$ ansible-playbook playbook.yml --ask-vault-pass

# Использование файла с паролем
$ ansible-playbook playbook.yml --vault-password-file ~/.ansible/vault_pass.txt

# Использование нескольких vault
$ ansible-playbook playbook.yml --vault-id dev@vault-pass.txt --vault-id prod@vault-prod.txt
```
#### Структура `vault`-файла
```yml
---
# secrets.yml (зашифрованный файл)
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          663864396532363436303937653835396239343731646463316337376237393264303535623333
          6539333561653165373731633364393836313036373264370a646533393866376531316432343563
          32623539373664343237646135313765613261323662633733303031366535623834663438386264
          3433616365393634330a363561373534353136393838393639623438643064623666663163376239
          6636

api_key: "{{ vault_api_key }}"
```

# 12. Отладка и диагностика
#### Полезные команды
```bash
# Проверка синтаксиса плейбука
$ ansible-playbook playbook.yml --syntax-check

# Проверка соединения с хостами
$ ansible all -i inventory.ini -m ping

# Выполнение ad-hoc команд
$ ansible webservers -i inventory.ini -m apt -a "name=htop state=present" --become

# Сбор информации о хостах (facts)
$ ansible webservers -i inventory.ini -m setup

# Тестирование переменных
$ ansible localhost -m debug -a 'var=groups' --extra-vars "@vars.yml"

# Валидация инвентаря
$ ansible-inventory -i inventory.ini --list
$ ansible-inventory -i inventory.ini --graph
```
#### Отладочные модули в плейбуках
```yml
- name: Отладка переменной
  debug:
    var: ansible_distribution

- name: Отладка с сообщением
  debug:
    msg: "Current host is {{ inventory_hostname }}"

- name: Вывод всех переменных (осторожно!)
  debug:
    var: hostvars[inventory_hostname]

- name: Пауза для ручной проверки
  pause:
    prompt: "Press Enter to continue or Ctrl+C to abort"

- name: Проверка с assert
  assert:
    that:
      - ansible_distribution == "Ubuntu"
      - nginx_port is defined
    fail_msg: "This playbook only works on Ubuntu with nginx_port defined"
    success_msg: "All checks passed"
```

## В этом руководстве были представлены основы Ansible, включая инструменты для управления хостами, конфигурации и установки пакетов, копирования файлов и управления службами, а также использование ролей для управления плейбуками и задачами.
