Структура проекта
=================

Структура папок:

```
app
  /actions
    __tests__/
  /actionTypes
  /components
    __tests__/
    /connectors
    /containers
    /propTypes
    /styles
    /views
  /constants
  /propTypes
  /reducers
    __tests__/
  /sagas
    __tests__/
  /selectors
    __tests__/
  /store
  /utils
    __tests__/
```


## 1. Actions, action types, reducers

Сущности библиотеки [redux](https://redux.js.org/). Хранятся отдельно от компонентов.

Action types находятся в папке `actionTypes`. Обычно сгруппированы в файлы по категориям.

Пример:
```javascript
export const ORDER_CONVERT_APPEAL = 'ORDER_CONVERT_APPEAL';
```

Actions хранятся в папке `actions`. Обычно сгруппированы в файлы по категориям. Импортируют actionTypes. В качестве аргумента принимают конкретные данные, например, id.

Пример:
```javascript
export function setFiles(loaderId, files) {
 return {
   type: SET_FILES,
   payload: { loaderId, files },
 };
}
```

Reducers хранятся в папке `reducers`. Обычно сгруппированы в файлы по категориям. Импортируют actionTypes. Реализуют стандартный паттерн со `switch` по типу action.

Пример:
```javascript
export default function textZones(state = {}, action) {
 switch (action.type) {
   case ORDERS_SET_TEXT_ZONE:
     return {
       ...state,
       [action.payload.textZone.slug]: action.payload.textZone,
     };
   default:
     return state;
 }
}
```


## 2. Sagas

Сущности библиотеки [redux-saga](https://redux-saga.js.org/). Хранятся в папке `sagas`. Представляют собой 
[функции-генераторы]( https://developer.mozilla.org/ru/docs/Web/JavaScript/Guide/Iterators_and_Generators).
Используются для связи приложения со внешним миром. В основном содержат логику отправки запросов по сети и обработки
 ответа. Обычно сгруппированы в файлы по категориям. В корневой саге `rootSaga.js` должны быть запущены все саги, 
нужные приложению.

Типичная структура саги:
* взять что-то из store
* отправить запрос
* обработать ответ и позвать action с полученными данными

Как и в случае с обычными функциями, код можно организовать с выделением вспомогательных саг.
Внутри функции-генератора не забываем пользоваться `try-catch` для отлова ошибок от сервера и не только.

Пример:

```javascript
// это вспомогательная функция
export function* fetchTextZone(action) {
 const slug = action.payload.slug;

 try {
   const res = yield call(api.get, `/text_zones/${slug}`);
   yield put(textZonesActions.setTextZone(res.data.text_zone));
 } catch (error) {
   if (error.response && error.response.status === 404) {
     yield put(textZonesActions.setTextZone({ slug, content: null }));
   }
 }
}

// эта сага будет использована в корневой, т.к. она слушает action
export default function* textZones() {
 yield takeEvery(actionTypes.ORDERS_GET_TEXT_ZONE, fetchTextZone);
}
```


## 3. Компоненты /components

Реализуются с помощью библиотеки [react](https://reactjs.org/).
Логически выделенная часть интерфейса приложения. Один компонент приложения представляет собой папку в `app/components` 
и может содержать один или несколько React-компонентов. Внутри `app/components` не используются подпапки. Компонент 
импортируется только через индексный файл, другие компоненты ничего не знают о его структуре.


## 4. Вспомогательные утилиты /utils

Содержит файлы с часто используемыми вспомогательными функциями, а также утилиты для тестирования.

Пример:

```javascript
export function bytesToMbytesString(amount) {
 return `${amount / 1e6} Мб`;
}
```


## 5. Селекторы /selectors

Созданные с помощью [reselect](https://github.com/reactjs/reselect) функции для извлечения данных из store.

Пример:

```javascript
const getReminds = state => state.order.reminds.reminds;

export const getActualReminds = createSelector(
 getReminds,
 reminds => reminds.filter(({ status }) => ['active', 'overdue', 'deferred'].includes(status)),
);
```


## 1.6. Константы /constants

Константы, используемые приложением.

Пример:
```javascript
export const DEFAULT_PER_PAGE = 25;
```


## 7. Store /store

Создание и конфигурация хранилища данных для [redux](https://redux.js.org/).


## 8. Prop types /propTypes

Общие для приложения [prop types](https://reactjs.org/docs/typechecking-with-proptypes.html), используемые разными компонентами.
