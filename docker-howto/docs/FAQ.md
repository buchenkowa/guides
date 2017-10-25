FAQ
===

* [Как устанавливать гемы не из сети Абака?](#Как-устанавливать-гемы-не-из-сети-Абака)

* [При установке гемов появляется ошибка `Permission denied (publickey)..`](#При-установке-гемов-появляется-ошибка-permission-denied-publickey)

Как устанавливать гемы не из сети Абака?
----------------------------------------

1. Необходимо получить доступ к [gems.railsc.ru](http://gems.railsc.ru), создав задачу, по аналогии с 
[SERVER-3671](https://jira.railsc.ru/browse/SERVER-3671). Возможно, потребуется ОК вашего руководителя.

2. Далее необходимо запустить иницализацию следующим образом:

    ```bash
      APRESS_GEMS_CREDENTIALS=username:password dip provision
    ```

    username:password заменить на реальные логин и пароль, выданный Вам для доступа к apress-гемам.

3. Чтобы постоянно явно не передавать переменную окружения `APRESS_GEMS_CREDENTIALS` при инциализации, можно
прописать ее в `~/.bashrc` следующим образом:

    ```bash
      echo export APRESS_GEMS_CREDENTIALS=username:password >> ~/.bashrc
      . ~/.bashrc
    ```




При установке гемов появляется ошибка `Permission denied (publickey)..`
-----------------------------------------------------------------------

1. Убедиться, что в `~/.ssh/` создана пара rsa ключа, приватный (id_rsa) и публичный (id_rsa.pub)

2. Убедиться, публичная часть ключа добавлена в Ваш гитхаб аккаунт 
    ([инструкция](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/))

3. Перезапустить `ssh-агент`:

    ```bash
      dip ssh down
      dip ssh up
    ```
