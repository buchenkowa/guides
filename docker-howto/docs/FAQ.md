FAQ
===

* [Как устанавливать гемы не из сети Абака?](#Как-устанавливать-гемы-не-из-сети-Абака)

* [Ошибки при установке гемов](#Ошибки-при-установке-гемов)

  - [`Permission denied (publickey)...`][0]

  - [`Bundler::Fetcher::CertificateFailureError Could not verify the SSL certificate for https://rails-assets.org/.`][1]

[0]: #permission-denied-publickey
[1]: #bundlerfetchercertificatefailureerror-could-not-verify-the-ssl-certificate-for-httpsrails-assetsorg

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


Ошибки при установке гемов
--------------------------

### `Permission denied (publickey)...`

1. Убедиться, что в `~/.ssh/` создана пара rsa ключа, приватный (id_rsa) и публичный (id_rsa.pub)

2. Убедиться, публичная часть ключа добавлена в Ваш гитхаб аккаунт 
    ([инструкция](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/))

3. Перезапустить `ssh-агент`:

    ```bash
      dip ssh down
      dip ssh up
    ```

### `Bundler::Fetcher::CertificateFailureError Could not verify the SSL certificate for https://rails-assets.org/.`

Такое периодически происходит, когда заканчивается сертификат у `rails-assets.org`. В данной ситуации нужно просто
подождать, когда сертификат обновят. На это время в гемфайле нужно заменить ссылку на http://insecure.rails-assets.org
и сборка пройдет успешно.

последний раз такая проблема была в декабре 2017: https://github.com/tenex/rails-assets/issues/416
