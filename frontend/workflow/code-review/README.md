Код ревью
=========

* [Колесо проверки реквестов](code-review-wheel/README.md)

* [Порядок проверки пулл реквеста](#Порядок-проверки-пулл-реквеста)

* [Когда звать фронтового руководителя](#Когда-звать-фронтового-руководителя)

## Порядок проверки пулл реквеста

:warning: Раздел является дополнением к [общим правилам компании](../../../code-review/README.md) :warning:

Для оптимизации проверок и пропускания наименьшего количества ошибок, во врине у фронтендеров организована
круговая проверка PR. [Колесо проверки реквестов](code-review-wheel/README.md).

Если PR задевает **меньше 25 строк**, достаточно получить **только 1 ок от утверждающего**.

Срок первой реакции проверяющих и утверждающих - **24 часа**.

**В тех случаях, когда бэкэнд-разработчик задевает вьюхи или другие фронтовые части,
схема проверки следующая:**
- Проверяющий - один из младших фронтэнд разработчиков (по ситуации).
- Утверждающий - в зависимости от проекта старший фронтенд разработчик.

**Список понятий:**

- *Автор реквеста* - тут всё понятно.

- *Проверяющий* - тот, кто проверяет первый раз. Находим ошибки, вызываем утверждающего.
Ждать, пока автор их исправит, не нужно. Ваша часть сделана.

- *Утверждающий* - проверяет ещё раз, если ошибки есть - пишет в реквесте, что ошибки можно править.
Если нет, то апрувает реквест.

- *Запасной* - заменяет только *утверждающего.*

**(!) Если кто-то из проверяющих не может посмотреть**, передайте реквест любому фронтенд-разработчику,
который не входит в список утверждающих.


## Когда звать фронтового руководителя

В ряде случаев возникает необходимость звать руководителя группы фронт-энд разработки в реквесты.

- **Задачи после проектирования, где, помимо ОКа, есть коммент, что надо звать в реквест.**
Если есть мнение, что задача достаточно сложная/важная и руководитель группы фронтов оставил коммент,
что его надо позвать в реквест по этой задаче.

- **Любые задачи с крупным рефакторингом**
Необходимость проверки таких реквестов возникает потому, что сам по себе крупный рефакторинг не проектировался
(иначе бы попал под пункт 1. Это, в свою очередь, создает вопрос правильно ли он сделан, так как нет смысла убирать
один тех.долг, чтобы создать другой.

- **Если в реквесте спорный вопрос, который не могут решить на протяжении двух дней**
Чтобы не тормозить задачу еще дольше, руководитель приходит и решает спор.

- **Если задача делалась значительно дольше времени, чем планировалось (+2 дня).**
Если задача делалась значительно дольше времени, чем планировалось, значит задача не такая простая, какой её оценили
на этапе планирования. В таком случае решение попадает в определенную зону риска. Задачу могли плохо реализовать,
либо она настолько сложная, что ей требовалось проектирование.

- **Любое внедрение новых решений, которые каким-то образом не были спроектированы**
Сюда входит внедрение любых решений, которые “сделают разработку лучше/проще/дешевле”. Любое добавление каких-то
сторонних компонентов итд итп. Любое изменение межпроектных стилей/кода.

По умолчанию на проверку реквестов, соответствующих перечисленным требованиям, зовёт разработчик, сделавший реквест.
Но также может призвать и тот, кто обладает правами мержа в соответствуеющий репозиторий.