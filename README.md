# Автоматизация администрирования. Ansible

Подготовить стенд на Vagrant как минимум с одним сервером. На сервере используя Ansible необходимо развернуть nginx со следующими условиями:
- использовать модуль yum/apt
- конфигурационные файлы должны быть взяты из шаблона jinja2 с переменными
- после установки nginx должен быть в режиме enabled в systemd
- должен быть использован notify для старта nginx после установки
- сайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible


# Установка Ansible


Устанавливаем:
```
yum install ansible
```
Проверяем:  
```
ansible --version
```
# Настройка Ansible

Поднимаем vm командой ```vagrant up```.  
Узнаем параметры Vagrant с помощью команды ```vagrant ssh-config``` их нужно будет добавить в файл hosts.  

Создадим файл hosts, cодержимое файла:   
```
[web]
nginx ansible_host=127.0.0.1 ansible_port=2222 ansible_private_key_file=/home/iteco/ansible/ansible1/1/.vagrant/machines/nginx/virtualbox/private_key
```
Отредактируем файл ```ansible.cfg``` в ```/etc/ansible```

inventory - путь к моему hosts

remote_user = vagrant

host_key_checking = False - отменить проверку пароля

# Ansible и playbook

Проверим что Ansible может управлять нашим хостом. Сделать это
можно с помощью команды:
```
ansible -i /home/iteco/ansible/ansible1/inventory/hosts all -m ping
```

Проверяем параметры:
```
ansible -i /home/iteco/ansible/ansible1/inventory/hosts all -m systemd -a name=firewalld
```
Нас интересует только строка
```
"status": {
        ...
        "ActiveState": "inactive", 
        ...
```
Установим пакет epel-release на vm:
```
ansible -i /home/iteco/ansible/ansible1/inventory/hosts all -m yum -a "name=epel-release state=present" -b
```

Создадим Playbook, который будет устанавливать пакет epel-release, nginx.  
В файл добавили Tags. Теперь можно выполнить, например, только установку NGINX.  
Далее добавим шаблон для конфига NGINX и модуль, который будет копировать этот шаблон на хост, пропишем в Playbook необходимую нам переменную для того, чтобы NGINX слушал на порту 8080.
Теперь создадим handler и добавим notify к копированию шаблона. Теперь каждый раз когда конфиг будет изменяться - сервис перезагрузиться.  
Playbook файл можно посмотреть во вложении файл setup.yml  
Проверим работоспособность командой:  
```ansible-playbook setup.yml```
Результат мы видим на скрине во вложении.

# Роль
Все файлы расположены в директории "Role"
Для создания роли
```
ansible-galaxy init nginx
```

Разделяем наш playbook, перемещая всё по соответствующим директориям.
 - 1 В раздел handlers - перемещаем всё из блока handlers в playbook (в самом playbook перенесёное удаляем).

 - 2 В раздел tasks - перемещаем всё из блока tasks в playbook.

 - 3 В раздел templates перенесим содержимое текущего templates, выпилив существующую директорию.
 - 4 Раздел vars
```
---
# vars file for nginx
nginx_port: 8080
```

Остальные директории были удалены. Playbook принял следующий вид :
```
---
- name: Install and configure NGINX
  hosts: all
  become: true

  roles:
    - nginx
```

Проверим работоспособность командой:  
```ansible-playbook setup.yml```
