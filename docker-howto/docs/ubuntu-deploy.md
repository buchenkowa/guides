Разворачивание проектов apress в докере на OC Ubuntu
=========================================================

Процесс разворачивания окружение можно разделить на следующие этапы:

1. [Установка docker](#1-Установка-docker)

2. [Установка docker-compose](#2-Установка-docker-compose)

3. [Установка dip](#3-Установка-dip)

4. [Клонирование проекта из репозитория](#4-Клонирование-проекта-из-репозитория)

5. [Скачивание дампов базы данных проекта](#5-Скачивание-дампов-базы-данных-проекта)

6. [Запуск ssh агента](#6-Запуск-ssh-агента)

7. [Запуск dns сервера](#7-Запуск-dns-сервера)

8. [Запуск nginx proxy](#7-Запуск-nginx-proxy)

9. [Первичная инициализация проекта](#8-Первичная-инициализация-проекта)


1\. Установка docker
--------------------

* Устанавливать нужно docker community edition (docker-ce) следуя алгоритму, приведенному в 
  [официальной документации](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/).

* После установки **обязательно** нужно добавить своего пользователя в группу `docker`, чтобы была возможность работать
  с докером без `sudo`:

  ```bash
    sudo groupadd docker
    sudo usermod -aG docker `echo $USER`
  ```

  Чтобы изменения вступили в силу необходимо выполнить полный выход из системы текущим пользователем, либо
  просто перезагрузиться:

  ```bash
    sudo reboot 
  ```


2\. Установка docker-compose
----------------------------

После успешной установки docker-ce, необходимо установить docker-compose:

```bash
  sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$$(uname -s)-$$(uname -m)" -o /usr/local/bin/docker-compose;
  sudo chmod +x /usr/local/bin/docker-compose;
```


3\. Установка dip
-----------------

* dip - это CLI утилита, которая упрощает работу с apress проектами. Инициализацию, запуск, остановку, миграции и пр. 
  нужно будет выполнять с помощью этой утилиты.

  [Инструкция](https://github.com/bibendi/dip/releases) по установке предполагает, что директория `/usr/local/bin/` 
  доступна для записи не из под рута. По умолчанию ОС Ubuntu не предоставляет таких разрешений и с точки зрения 
  безопасности лучше так не делать.

  Поэтому установку `dip` можно выполнить следующим способом:

  ```bash
	  sudo apt-get install libyaml-0-2 2>/dev/null
    curl -L https://github.com/bibendi/dip/releases/download/2.0.0/dip-`uname -s`-`uname -m` > ~/dip
    sudo mv ~/dip /usr/local/bin/dip
    sudo chmod +x /usr/local/bin/dip
  ```


4\. Клонирование проекта из репозитория
---------------------------------------

Если ОС Ubuntu только что установлена, то по умолчанию может не быть git'a. Для его установки: 

```bash
  sudo apt-get install git
```

Клонирование репозитория (PROJECT=blizko|pulscen|lookmart):

```bash
  git clone https://github.com/abak-press/PROJECT
```


5\. Скачивание дампов базы данных проекта
-----------------------------------------

В инструкции по запуску из докера для каждого проекта указано откуда и какие дампы нужно взять. 
инструкции по скачиванию дампов:

* [Pulscen](https://github.com/abak-press/pulscen/tree/develop/docker#postgresql)
* [Blizko](https://github.com/abak-press/blizko/tree/develop/docker#postgresql)
* [Lookmart](https://github.com/abak-press/lookmart/tree/develop/docker#postgresql)

Если находимся за абаковской сетью, необходим доступ для скачивания дампов с https://ekb.dumps.railsc.ru/

- [Пример задачи на получение доступа](https://jira.railsc.ru/browse/SERVER-3820)

6\. Запуск ssh агента
---------------------

* ssh агент, по умолчанию, ожидает, что в директории `~/.ssh/` находится сгенерированная пара rsa ключа, приватный 
  (id_rsa) и публичный (id_rsa.pub). Имена файлов важны! Также необходимо добавить публичную часть rsa ключа в Ваш 
  гитхаб аккаунт. 

  Созданием нужного rsa ключа:

  ```bash
    ssh-keygen -f ~/.ssh/id_rsa -t rsa
  ```

  - При создании ключа можно задать `passphrase` - тогда ее обязательно нужно запомнить

  - Инструкция по добавлению ssh ключа в гитхаб аккаунт находится
    [здесь](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/).

  После создания и добавления ключа на гитхаб стартуем ssh агент:

  ```bash
    dip ssh add
  ```

  - Если для ключа задана `passphrase` ее необходимо будет ввести. Если запуск прошел успешно, то в консоли появится
  сообщение вида `Identity added: ${путь к приватному ключу} (${путь к приватному ключу})`


* Также, имеется возможность явно указать ssh агенту какую rsa-пару использовать:

  ```bash
    dip ssh add -k RSA_KEY
  ```

  - `RSA_KEY` - абсолютный путь к приватному ключу. Если в пути есть пробелы, то путь необходимо поместить в кавычки.
                Публичная часть ключа должна находится в одном каталоге с приватной. Имя публичной части ключа должно 
                совпадать с приватной, с добавлением в конец `.pub`

  Приимер:

    ```bash
      dip ssh add -k '/home/user/my rsa/rsa_key'
    ```

    в данном случае ssh агент ожидает, что в каталоге `/home/user/my rsa/` находится как приватная часть ключа 
    (`rsa_key`) так и публичная (`rsa_key.pub`).

  

7\. Запуск dns сервера 
----------------------

Для запуска dns сервера, инициализации и запуска с нужными настройками:

```bash
  dip dns up
  echo address=/docker/127.0.0.1 | sudo tee -a /etc/NetworkManager/dnsmasq.d/dnsmasq.conf;\
  echo export DOCKER_TLD=localhost | tee -a ~/.bashrc;\
  sudo service network-manager restart;
```

8\. Запуск nginx proxy
----------------------

Для запуска nginx proxy сервера, инициализации и запуска с нужными настройками:

```bash
  dip nginx up;
```


9\. Первичная инициализация проекта
----------------------------------

Для первичной инициализации, необходимо из корневой директории проекта выполнить:

```bash
  dip provision
```

После инициализации проекта, можно стартовать сервер:

```bash
  dip rails s
```

Проект будет доступен из браузера по адресу:  `www.PROJECT.docker`, где `PROJECT=blizko|pulscen|lookmart`.

Подробная инструкция по командам управления проектом (миграции, старт сервера, resque, база и тд) описано в проектной
инструкции:

* [Pulscen](https://github.com/abak-press/pulscen/tree/develop/docker)
* [Blizko](https://github.com/abak-press/blizko/tree/develop/docker)
* [Lookmart](https://github.com/abak-press/lookmart/tree/develop/docker)
