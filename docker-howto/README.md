apress-docker-howto
===================

![dir-image](images/apress-docker.png)

Руководство аккумулирует опыт работы с проектами *apress* через Docker.

* [Кому и зачем](#Кому-и-зачем)

* [Важно](#Важно)

* [Структура](#Структура)


Кому и зачем
-------------

**кому**

* Всем сотрудникам ВРИНа, кто хотел бы работать с проектами *apress* локально через Docker, но не знает с чего начать
* Всем сотрудникам ВРИНа, кто уже работает с докером, но столкнулся с какими-то трудностями
* Всем сотрудникам ВРИНа, кто хочет дополнить/добавить/исправить/сообщить о проблеме по данной теме

**зачем**

* собрать необходимую для быстрого старта проектов *apress* информацию в одном месте
* помочь/предложить вариант решения известных проблем
* предоставить возможность развернуть проект одной командой


Важно
-----

Cобранные заметки распространяются только на работу с проектами *apress* через
Docker в ОС Ubuntu.

Работоспособность решений проверена в следующих OC:

* Ubuntu 14.04
* Ubuntu 16.04
* Ubuntu 18.04


Структура
---------

* [README.md][0]    - общее описание руководства
* [makefile][1]     - файл для разворачивания проектов *apress* в докере одной командой
* [CHANGELOG.md][2] - лог изменений `makefile`
* [docs/][3]        - директория c документацией
  - [FAQ.md][4]             - сборник заметок по работе с проектами *apress* через докер, оформленные в стиле FAQ
  - [ubuntu-deploy.md][5]   - инструкция по развертыванию проектов *apress* в докере на Ubuntu  
  - [ubuntu-install.md][6]  - инструкция по установке ОС Ubuntu Linux
  - [make.md][7]            - инструкция по работе с makefile, для разворачивания проектов *apress* одной командой
* [images/][8]      - директория с изображениями

[0]: README.md
[1]: makefile
[2]: CHANGELOG.md
[3]: docs
[4]: docs/FAQ.md
[5]: docs/ubuntu-deploy.md
[6]: docs/ubuntu-install.md
[7]: docs/make.md
[8]: images/
