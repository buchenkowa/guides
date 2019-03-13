# План адаптации

## Этап 1

### Получить доступ к:
- https://jira.railsc.ru к закрепленному проекту
- https://conf.railsc.ru
- https://github.com/abak-press
- cгенерировать ssh ключ
- gems.railsc.ru для установки приватных гемов
- настроить доступ к тестовой площадке knife.railsc.ru (через логин github)

### Настройка окружения
- установить Ubuntu 16.04 или 18.04,
- установить докер для Ubuntu и развернуть приложение с проектом (blizko.ru, pulscen.ru, yapokupayu.ru) на докере https://github.com/abak-press/guides/blob/master/docker-howto/docs/make.md
- установить телеграмм https://conf.railsc.ru/pages/viewpage.action?pageId=27755101

## Этап 2

### Первое знакомство с проектом
- http://edx.railsc.ru/accounts/login?next=/courses/VRIN/intro/2015/courseware/897014f8b6d7419cad212e1e4117f0ef/66262248a5774e0f9d30d35fc6e19720/ (необходимо зарегистрироваться)

### Знакомство с соглашениями о разработке ВРИН:
- [правила оформления scss, js, порядок проверки pull requests](https://github.com/abak-press/guides/tree/master/frontend)
- [работа с исходным кодом и репозиторием проекта](https://github.com/abak-press/guides/tree/master/abak-flow)
- [оформление PR и code review](https://github.com/abak-press/guides/tree/master/code-review#%D0%A2%D1%80%D0%B5%D0%B1%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F-%D0%BA-%D0%BE%D1%84%D0%BE%D1%80%D0%BC%D0%BB%D0%B5%D0%BD%D0%B8%D1%8E-pr)  
- [оформление кода](https://github.com/abak-press/guides/tree/master/style)
- [общая документация](https://github.com/abak-press/guides)

### Знакомство с:
- [документация по SASS](http://sass-lang.com/)
- [документация-учебник по HAML](http://haml.info/)
- среда Rails – книга “Гибкая разработка в среде Rails”, главы 18, 21. (можно взять в отделе, есть в наличии электронный вариант книги)

Там же можно ознакомиться, но на это можно обратить внимание и на этапе выполнения реальных задач:
- с тем, что такое Gemfile
- как запускаются/откатываются миграции

### Либо освежить в памяти, либо прочитать о github, обращая внимание на:
- подключение к удаленному репозиторию
- создание/удаление/переименование веток
- push/pull
- создание коммита (--ammend)
- rebase – :warning: обязательно :warning:
- fetch
- cherry-pick – :warning: обязательно :warning:

[Ссылка для ознокомления](https://git-scm.com/book/ru/v1/%D0%92%D0%B2%D0%B5%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5-%D0%9E%D1%81%D0%BD%D0%BE%D0%B2%D1%8B-Git): https://git-scm.com/book/ru/v1/%D0%92%D0%B2%D0%B5%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5-%D0%9E%D1%81%D0%BD%D0%BE%D0%B2%D1%8B-Git

## Этап 3

### Объяснение:
- порядка выполнения задач
- взаимодействия с участниками команды - МР, ПМ, тестировщиками
- этапы и статусы задач
- что такое SCRUM, спринт ([хотя бы в общих чертах](https://ru.wikipedia.org/wiki/Scrum))
- желательно объяснить что такое фокус-фактор

**Ссылка на source репозитория abak-press** http://sources.railsc.ru (регистрация через github)

**Знакомство с проектом по задачам. Задачи от простого к сложному, от исправлений багов до полноценной разработки.**

**Во время третьего этапа, когда кандидат будет делать самостоятельно задачи станет понятно, что еще можно/нужно подтянуть, на что обратить внимание.**
