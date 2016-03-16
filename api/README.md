# Концепция
Аутентификация осуществляется на основе HMAC алгоритма. Что это значит в двух словах.
На Клиенте и на Сервере есть одинаковый секретный ключ, которые они не передают по сети.
С этого ключа Клиент подписывает свой запрос, а Сервер проверяет эту подпись.
Клиент передаёт на Сервер только свой access_id. 
На Сервере хранится таблица соответствий access_id => secret_token.

http://www.thebuzzmedia.com/designing-a-secure-rest-api-without-oauth-authentication/

## Модульность
Каждое специфичное API будет частью гема, реализующего эту функциональность.
Каждый гем содержит документацию к API в редейме или доп. документе.

## Версионирование
Предлагается применить погемовое версионирование API.
При работе с API в урле обязательно указывается версия.
Маршруты и права конкретного API будут располагаться в конкретном геме.
В каждом геме, на ряду с конкретными маршрутами описывающими конкретные версии, будет возможность вызвать ресуср без указании версии, в таком случае будет использована последняя версия гемового апи.

Устаревшие версии будут закрываться, а клиенту при запросе будет возвращаться специальный код `410`. Это реализуется в маршрутах, с помощью обработки всех экшинов устаревшей версии, с помощью спец контроллера `Apress::Api::DeprecatedVersionsController`.

пример урла:
`http://www.pulscen.ru/api/v2/products`
`http://ekb.pulscen.ru/api/v1/products`
`http://krepika.pulscen.ru/api/v2/products`
`http://www.krepika.ru/api/v1/products`
`http://www.krepika.ru/api/products` -> `http://www.krepika.ru/api/v2/products` (т.к. посл. v2)

## Возможности
- Авторизация с помощью access_id/secret_token (HMAC)
- Авторизация с помощью session_id (будучи авторизованным на портале)
- Через логин, пароль и device_id получить secret_token, access_id и refresh_token
- Через access_id/refresh_token получить secret_token и refresh_token

Авторизация через session_id не рекомендуется к использованию и сделана для плавного перехода мобильного приложения на новое API и для обеспечения работы через API JavaScript приложений (например ЕТИ)

## Формат ответа
- Ответ формируется в формате JSON
- В ответе должен всегда присутствовать root элемент. примеры:
```json
{
  rubric: {id: 123}
}

{
  rubrics: [
    {id: 123},
    {id: 124}
  ]
}
```
- постраничная наваигация должна возвращаться в заголовках ответа `X-Total-Count X-Total-Pages`

## Разработка и отладка
Предполагается возможность легкой разработки и отладки через браузер и curl. Для этого нужно сделать:
- все токены и основные параметры читать как из заголовков, так и через параметры
- в тестовом окружении, при наличии специального параметра `check_signature=0`, не проверять подпись запроса

## Плюсы подхода
- универсальность. будут одинаково взаимодействовать наши внутренние сервисы и внешние клиенты
- легкость реализации на клиенте
- не нужно передавать логи/пароль (безопасность)
- не нужно хранить сессию, между запросами
- независимая авторизация на разных устройствах

# Реализация
## Gem `apress-api`
### DB
*api_clients*

id - integer
device_id - string
access_id - string
secret_token - string
secret_token_expire_at - datetime
refresh_token - string
refresh_token_expire_at - datetime
user_agent - string
created_at - datetime
updated_at - datetime

### Rails

#### Apress::Api::BaseController < ActionController::Metal
- `before_filter #authenticate`
  - берем из заголовков `Authorization` (желательно брать из параметров, для легкой работы через браузер и curl)
  - пример заголовка: `Authorization = APIAuth 'client access id':'signature'`
  - Ищем secret_token в таблице api_clients и проверяем signature.
  - Всю эту работу выполняет гем [api-auth](https://github.com/mgomes/api_auth), все подробности подписания читать там

#### Apress::Api::TokensController < BaseController
- `#create`
  - POST /api/v1/clients/:access_id/tokens/?refresh_token=
  - обновление refresh_token и secret_token
  - коды ответа: 404 - клиент не найден, 400 - токен не найден, 403 - токен протух
  - формат ответа:
  ```json
  {
    client: {
      id:, access_id:, device_id:, secret_token_expire_at:, refresh_token_expire_at:, user_agent:,
      updated_at:, created_at:, secret_token:, refresh_token:
    }
  }
  ```

### Dependencies 
api-auth, jbuilder


## Gem `apress-api-application`
Предлагается следующая схема.
Юзер в мобилке вводит логин/пароль, которые отправляются на Сервер (желательно по SSL, чтобы не светить secret_token). Вместе с этими данными передается device_id - идентификатор устройства.
Сервер находит User, находит соотв. запись в таблице api_clients, по user_id и device_id, если записи нет, то создает. В ответ возвращает access_id, secret_token, refresh_token и доп. информацию, если нужно.

*secret_token* - это секретный ключ, известный только клиенту и серверу, с помощью него подписывается запрос. Время жизни ограничено 1 час.

*device_id* - является не обязательным
Этот идентификатор нужен, для разделения авторизационных реквизитов между несколькими устройствами одного пользователя.

*refresh_token* - это авторизационный ключ для получения нового secret_token. Существует для того, что бы клиенты не хранили логин и пароль пользователя. Время жизни ограничено 1 неделя.

Также этот гем авторизует пользователя. Т.е. есть аутентификация, а есть авторизация.
Авторизация - это определение роли пользователя и его доступных прав.
Авторизация будет работать стандартная портальная.

Помимо 

### DB
Добавляет в таблицу `api_clients` столбец `user_id: :integer`

### Rails

### Apress::Api::Application::ClientsController
- `#create`
  - В идеале должен находится на отдельном SSL домене. POST `login.pulscen.ru/api/v1/clients`
  - Сюда приходят `{session: {login: "", password: "", device_id: ""}}`
  - Ведётся поиск по таблице api_users, по user_id/device_id
  - Если записи нет, то создаётся новая
  - Регенерируется secret_token, refresh_token
  - Возвращает клиенту access_id, secret_token, secret_token_expire_at, refresh_token, refresh_token_expire_at

### Apress::Api::Application::Extensions::Apress::Api::BaseController
- `before_filter #authenticate`
  - если пришёл заголовок с сессией, то авторизуем обычным способом через authlogic
  - иначе
  - super (авторизация по HMAC)
  - находит пользователя по user_id в таблице api_clients
  - выполняет авторизацию юзера

Т.е. при работе с каждым экшеном API будет возможность авторизации через access_id и session_id.

### Dependencies 
apress-api, apress-clearance, apress-application

# Общая схема работы
- ЕСЛИ **есть access_id и secret_token**, ТО:
  - запрашиваю любые урлы API, в заголовках передаю access_id, подписываю все запросы secret_token'ом
  - слежу за протуханием secret_token
- ЕСЛИ **secret_token** протух, ТО:
  - обновляю токены через POST /api/v1/clients/:access_id/tokens/?refresh_token=
- ЕСЛИ **refresh_token** протух, ТО:
  - авторизуюсь через POST /api/v1/clients (ClientsController#create), передаю login, password, device_id
  - в ответ возвращаются access_id, secret_token, secret_token_expire_at, refresh_token, refresh_token_expire_at
- ЕСЛИ **есть кука с сессией**, ТО:
  - то я javascript
  - запрашиваю любые урлы API, в куке передаю ид сессии.

# Дополнительно
- отказаться от драппера в пользу джейбилдера
- отпределять мобайл роль по юзер агенту
- маршруты должны жить на текущем домене (на всех компанейских + портальных)
- при логауте, нужно экспаирить secret_token и refresh_token

# Перевод на новое API мобильного приложения
Какое-то время мобильному приложению придется работать с двуми API параллельно. Новый функционал будем реализовывать в новом API, старый будет доживать в старом. Постепенно, старое API удалим.

# Дополнительные ресурсы
[Изучаем REST: Руководство по созданию RESTful сервиса](http://www.restapitutorial.ru/)   
[JSON API](http://jsonapi.org/)  
[GitHub API](https://developer.github.com/v3/)
