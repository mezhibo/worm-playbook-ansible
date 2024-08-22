**Основная часть**

1. Подготовьте свой inventory-файл prod.yml.

2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает vector. Конфигурация vector должна деплоиться через template файл jinja2. От вас не требуется использовать все возможности шаблонизатора, просто вставьте стандартный конфиг в template файл. Информация по шаблонам по ссылке. не забудьте сделать handler на перезапуск vector в случае изменения конфигурации!

3. При создании tasks рекомендую использовать модули: get_url, template, unarchive, file.

4. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.

5. Запустите ansible-lint site.yml и исправьте ошибки, если они есть.

6. Попробуйте запустить playbook на этом окружении с флагом --check.

7. Запустите playbook на prod.yml окружении с флагом --diff. Убедитесь, что изменения на системе произведены.

8. Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.

9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги. Пример качественной документации ansible playbook по ссылке. Так же приложите скриншоты выполнения заданий №5-8

10. Готовый playbook выложите в свой репозиторий, поставьте тег 08-ansible-02-playbook на фиксирующий коммит, в ответ предоставьте ссылку на него.


**Решение**

Создадим 2 центос машины в Яндекс Клауд, и опишем их в inventory файле, сразу прописа учетку для доступа и ключ для подключения

![alt text](https://github.com/mezhibo/worm-playbook-ansible/blob/2fcd305fe13380b3d1ca688cbbc3cffc87cd6217/IMG/3.jpg)


Допишем в плейбук код, разворачивающий Vector и создадим jinja шаблон для Vector

Дописываем код в site.yml

```
- name: Install Vector
  hosts: vector-01
  pre_tasks:
    - name: Get Vector distrib
      ansible.builtin.get_url:
        url: "https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-x86_64-unknown-linux-musl.tar.gz"
        dest: "./vector-{{ vector_version }}-{{ vector_architecture }}-unknown-linux-gnu.tar.gz"
    - name: Create Vector directory
      become: true
      ansible.builtin.file:
        path: /etc/vector
        state: directory
  handlers:
    - name: Unarchive Vector package 
      ansible.builtin.command: "tar -xf ./vector-{{ vector_version }}-{{ vector_architecture }}-unknown-linux-gnu.tar.gz -C /ect/vector"
  tasks:
    - name: Template file
      become: true
      ansible.builtin.template:
        src: vector.toml.j2
        dest: /etc/vector/vector.toml
        mode: '0644'
    - name: Run Vector
      become: true
      ansible.builtin.shell: /etc/vector/vector-x86_64-unknown-linux-gnu/bin/vector --config /etc/vector/vector.toml &
  tags: vector
```

Созадаем жинжа шаблон 

![alt text](https://github.com/mezhibo/worm-playbook-ansible/blob/f7770b05af51efe8feb808f6f4b89871341a408b/IMG/4.jpg)


Запускаем плейбук с ключом --check


![alt text](https://github.com/mezhibo/worm-playbook-ansible/blob/f7770b05af51efe8feb808f6f4b89871341a408b/IMG/1.jpg)


И теперь выполняем плейбук и проверяем что он идемпотентен с помощью ключа --diff


![alt text](https://github.com/mezhibo/worm-playbook-ansible/blob/f7770b05af51efe8feb808f6f4b89871341a408b/IMG/2.jpg)


И вешаем тэг 08-ansible-02-playbook как того просит задание 

![alt text](https://github.com/mezhibo/worm-playbook-ansible/blob/69959f2d016db68b92c41480b896316b67e62bc5/IMG/5.jpg)


[ССЫЛКА ФАЙЛЫ ПРОЕКТА](https://github.com/mezhibo/mnt-homeworks/tree/3832537e2f3b9728258bc4fb1cd088485f35de21/08-ansible-02-playbook/playbook)


