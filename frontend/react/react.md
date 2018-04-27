#### 1. Структура проекта

Стек основных технологий:
* для компонентов интерфейса используем [react](https://reactjs.org/)
* для управления состоянием приложения используем [redux](https://redux.js.org/)
* для организации общения со внешним миром используем [redux-saga](https://redux-saga.js.org/)

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

##### 1.1. Actions, action types, reducers
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

##### 1.2. Sagas
Сущности библиотеки [redux-saga](https://redux-saga.js.org/). Хранятся в папке `sagas`. Представляют собой [функции-генераторы]( https://developer.mozilla.org/ru/docs/Web/JavaScript/Guide/Iterators_and_Generators).
Используются для связи приложения со внешним миром. В основном содержат логику отправки запросов по сети и обработки ответа. Обычно сгруппированы в файлы по категориям.
В корневой саге `rootSaga.js` должны быть запущены все саги, нужные приложению.

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

##### 1.3. Компоненты /components. см. [Компоненты](#Компоненты)

##### 1.4. Вспомогательные утилиты /utils
Содержит файлы с часто используемыми вспомогательными функциями, а также утилиты для тестирования.

Пример:
```javascript
export function bytesToMbytesString(amount) {
 return `${amount / 1e6} Мб`;
}
```

##### 1.5. Селекторы /selectors
Созданные с помощью [reselect](https://github.com/reactjs/reselect) функции для извлечения данных из store.

Пример:
```javascript
const getReminds = state => state.order.reminds.reminds;

export const getActualReminds = createSelector(
 getReminds,
 reminds => reminds.filter(({ status }) => ['active', 'overdue', 'deferred'].includes(status)),
);
```

##### 1.6. Константы /constants
Константы, используемые приложением.
Пример:
```javascript
export const DEFAULT_PER_PAGE = 25;
```

##### 1.7. Store /store
Создание и конфигурация хранилища данных для [redux](https://redux.js.org/).

##### 1.8. Prop types /propTypes
Общие для приложения [prop types](https://reactjs.org/docs/typechecking-with-proptypes.html), используемые разными компонентами.

#### 2. (#Компоненты)
Реализуются с помощью библиотеки [react](https://reactjs.org/).
Логически выделенная часть интерфейса приложения. Один компонент приложения представляет собой папку в `app/components` и может содержать один или несколько React-компонентов. Внутри `app/components` не используются подпапки. Компонент импортируется только через индексный файл, другие компоненты ничего не знают о его структуре.

##### 2.1. Структура компонента
Папка `SomeComponent`.

Cтруктура папки:
     `/__tests__ ` - тесты
содержит файлы `ИмяТестируемогоФайла.spec.js`
     `/styles` - стили
обычно один файл `имя-папки.scss`
     `/views` - "глупые" компоненты
принимают пропсы и по ним что-то отрисовывают: другие React-компоненты или разметку
не имеют состояния или lifecycle-методов
описываются функцией, function declaration или стрелочной
     `/containers` - компоненты-контейнеры
готовят данные для “глупых” компонентов
содержат в себе логику работы приложения, обработчики событий
имеют state и/или lifecycle-методы
описываются классом
     `/connectors` - компоненты-коннекторы
оборачивают другой компонент(контейнер или view) в [connect](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) из react-redux для работы со store и actions
Для того, чтобы достать данные из store, могут быть использованы селекторы, созданные с помощью [reselect](https://github.com/reactjs/reselect).
     `/propTypes`
обычно один файл index.js с prop types для всех views и containers этой папки
     `index.js` -  дефолтный экспорт React-компонента, который является точкой входа в наш компонент

В папках `views`, `containers`, `connectors` также содержится `index.js` с экспортом всех файлов данной папки.

##### 2.2. Оформление React-компонентов

Все компоненты используют jsx - синтаксис. Файл с React-компонентом имеет название в camel case и расширение js (не jsx). В конце названия добавляется его тип, например: `SomeComponentView.js`, `SomeComponentContainer.js`.
В начале файла находятся импорты всех зависимостей, затем код компонента.
Во views и containers после кода компонента находится присвоение propTypes и defaultProps.
В конце файла - дефолтный экспорт.

Все импорты - абсолютные в папке с приложением: `import { smth } from 'app/path/to/smth'` (не `import { smth } from ‘../../../path/to/smth’`).
Методы компонента, которые передаются в дочерние компоненты, объявляются через class properties, чтобы сохранить this. В render стрелочные функции стараемся не использовать.
Методы компонента, которые используются только внутри самого компонента, объявляются методами класса, чтобы не создавать новую функцию на каждый экземпляр класса.
Объекты, к которым есть обращение по нескольким ключам, деструктурируются.

Прммер деструктуризации:
```javascript
   const {
     props: {
       isFetching,
       userOrders,
       totalPages,
       activePage,
       location: { query },
     },
     handleSelect,
   } = this;
```

В компонентах не используется напрямую `dispatch`, все экшны должны быть обернуты в него через [connect](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options).
Размер одного компонента стараемся держать в рамках одного-двух экранов.
Если понадобился location или другие параметры урла, оборачиваем корневой компонент в [withRouter](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/api/withRouter.md) из react-router.

##### 2.3. Стили компонентов
Для стилей используется [sass](https://sass-scss.ru/) с синтаксисом scss.
В один из файлов c компонентами (обычно первый после коннектора в цепочке) подключается файл со стилями `styles/some-component.scss`. Он содержит стили для всех React-компонентов в папке.
Корневой элемент в верстке обычно имеет класс some-component, остальные добавляют к этому классу some-component-link. Для конструирования классов по условиям используется библиотека [classnames](https://github.com/JedWatson/classnames).

#### 3. Тестирование
Пишем юнит-тесты.
Библиотека для тестирования - [jest](https://facebook.github.io/jest/).
Дополнительная библиотека для тестирования React-компонентов - [enzyme](http://airbnb.io/enzyme/).
Библиотека для эмуляции redux store в тестах - [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store).
Для тестирования есть набор утилит в `app/utils`, облегчающих написание тестов.
В тестах не забываем сделать mock внешних зависимостей.

Файлы с тестами находятся в папке `__tests__/` и имеют имя `fileNameToTest.spec.js`.

Используем блоки `beforeAll`, `beforeEach`, группируем случаи с помощью `describe`.
В `describe` объявляем переменные, которые понадобятся в `it`-ах.
Если переменная нужна только в `before`-блоке, объявляем её там.
Внутри `before`-блока осуществляем подготовку данных.
`beforeEach` используем, если внутри какого-то `it` состояние данных изменяется.

Пример:
```javascript
describe(‘Some Component’, () => {
   let wrapper;

   beforeAll(() => {
    // не планируем обращаться к этой переменной снаружи
   const initialState = {
     someKey: ‘someValue’,
   };
   // эту переменную используем снаружи
     wrapper = prepareTestEnvironment({Component: SomeComponent, initialState});
   });

   it('should render <OtherComponent />', () => {
     expect(wrapper.find(OtherComponent)).toHaveLength(1);
   });
});
```

##### 3.1. Компоненты
Проверяем работу компонента изолированно от окружения.
Тестируется каждый React-компонент по отдельности.

###### 3.1.1 Connectors
Проверяем, что
 * коннектор оборачивает нужный компонент
 * передает ему в качестве props нужные данные из store
 * в props передаются ф-ии, которые действительно делают dispatch нужного action в store

Пример:
```javascript
 it('should render <FilterByStatusContainer />', () => {
   expect(wrapper.find(FilterByStatusContainer)).toHaveLength(1);
 });

 it('should pass userRole', () => {
   expect(wrapper.prop('userRole')).toBe(initialState.userRole);
 });

 it('should pass getOrdersList', () => {
   wrapper.prop('getOrdersList')();
   expect(store.getActions()).toEqual([mockGetOrdersListAction]);
 });
```

###### 3.1.2 Containers
Проверяем результат рендера компонента, результаты выполнения методов.
Обращаем внимание на наличие условий в коде компонента, проверяем поведение в зависимости от условий.
При тестировании методов стараемся не залазить во внутреннее состояние компонента, ориентироваться на внешние признаки - например:, поменялся результат рендера или вызвался action.

Пример:
```javascript
 it('should handle select', () => {
   wrapper.instance().handleSelect('789');
   expect(props.getOrdersList).toHaveBeenCalledWith({
     ...props.location.query,
     page: 1,
     abc: '789',
   });
 });

 it('should show error if quantityExceeded = true', () => {
   expect(wrapper.contains(
     <Error>Вы можете прикрепить не более {fileLoaderSettings.maxCount} файлов</Error>
   )).toBe(true);
 });
```

######  3.1.3. Views
Проверяем результат рендера компонента.
Тестируем вывод разных сущностей по условиям, передачу props в другие компоненты, вывод других компонентов (иногда объединяем в один `it` - проверяем  с помощью `contains`).

Пример:
```javascript
 it('should render <Loader>', () => {
   expect(wrapper.type()).toBe(Loader);
 });

 it('should render <OrdersTable>', () => {
   expect(wrapper.contains(
     <OrdersTable orderType={props.orderType} />
   )).toBe(true);
 });
```

##### 3.2. Actions
Проверяем, что по набору аргументов получается нужный action. Выделяем проверку каждого action в `describe`.

Пример:
```javascript
 describe('resetFiles', () => {
   it('should create action to reset files', () => {
     const loaderId = 'loader';
     const expectedAction = {
       type: RESET_FILES,
       payload: { loaderId },
     };

     expect(resetFiles(loaderId)).toEqual(expectedAction);
   });
 });
```

##### 3.3. Reducers
Для каждого типа action объединяем проверки в `describe`. Проверяем получившийся state. Проверяем также случай по умолчанию с пустым action.

Пример (случай по умолчанию):
```javascript
 describe('default', () => {
   it('should return initial state', () => {
     expect(targetActionsReducer(undefined, {})).toEqual(initialState);
   });
 });
```

Пример (обычный action):
```javascript
describe('SET_TARGET_ACTIONS', () => {
   it('should set target actions and isFetching', () => {
     const targetActions = [{ id: 1 }, { id: 2 }];
     action = {
       type: SET_TARGET_ACTIONS,
       payload: { targetActions },
     };

     expect(targetActionsReducer(commonState, action)).toEqual({
       data: targetActions,
       isFetching: false,
     });
   });
 });
```

##### 3.4. Sagas
Запускаем сагу с помощью библиотеки [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store) и проверяем результат выполнения - например, через store прошел какой-то action, вызвалась функция.

Пример:
```javascript
// подготовительная часть - запуск саги с использованием нашей утилиты prepareTestSagaEnvironment
 beforeAll(() => {
   const initialState = { userRole: 'seller' };
   const testSagaEnvironment = prepareTestSagaEnvironment(initialState);
   store = testSagaEnvironment.store;
   testSagaEnvironment.middleware.run(targetActionsSaga);
 });

 it('should set actions', () => {
   const response = { data: { target_actions: [{ id: 1 }, { id: 2 }, { id: 3 }] } };

// обязательно делаем mock сетевого запроса
   api.get.mockImplementation(() => response);
   const expectedActions = [setTargetActions(response.data.target_actions)];

   store.dispatch(getTargetActions(1));
// проверяем, что нужный action прошел через store
   expect(store.getActions()).toEqual(expect.arrayContaining(expectedActions));
 });
```

#### 4. Конфигурация приложения
В корне папке приложения есть набор конфигов.
`.babelrc` - конфиг для [babel](https://babeljs.io/docs/usage/babelrc/)
`.eslintrc.js` - конфиг для [eslint](https://eslint.org/docs/user-guide/configuring#configuration-file-formats)
`jest.config.js` - конфиг для [jest](https://facebook.github.io/jest/docs/en/configuration.html)
`lint-staged.config.js` - конфиг для [lint-staged](https://github.com/okonet/lint-staged#configuration)
`webpack.dev.config.js` - конфиг для [webpack](https://github.com/webpack/docs/wiki/configuration) для разработки
`webpack.prod.config.js` - конфиг для [webpack](https://github.com/webpack/docs/wiki/configuration) для продакшна


#### 5. Стайлгайд
Используется конфиг от [airbnb](https://github.com/airbnb/javascript) для [eslint](https://eslint.org/).
Некоторые правила переопределены/дополнены в `.eslintrc.js`.
Прекоммит-хук проверяет готовые к коммиту файлы, он автоматически исправит недочеты и выведет ошибки.

#### 6. Библиотеки

Код приложения:
[react](https://reactjs.org/) - реализация компонентов интерфейса
[redux](https://redux.js.org/) - управление состоянием приложения
[redux-saga](https://redux-saga.js.org/) - управление внешним взаимодействием приложения
[reselect](https://github.com/reactjs/reselect) - извлечение данных из store
[axios](https://github.com/axios/axios) - для запросов по сети
[classnames](https://github.com/JedWatson/classnames) -  для добавления классов по условиям
[lodash](https://lodash.com/) - библиотека вспомогательных функций
[momentjs](https://momentjs.com/) - для работы с датами и временем
[prop-types](https://github.com/facebook/prop-types) - для типизации пропов React-компонентов
[react-router](https://reacttraining.com/react-router/) - навигация внутри приложения

Для тестов
[jest](https://facebook.github.io/jest/) - фреймворк для тестирования
[enzyme](https://facebook.github.io/jest/) - утилиты для тестирования React-компонентов
[redux-mock-store](https://github.com/arnaudbenard/redux-mock-store) - для эмуляции redux store

Проверка кода
[eslint](https://eslint.org/) - проверка синтаксиса
[eslint-watch](https://github.com/rizowski/eslint-watch) - запуск eslint в фоновом режиме
[lint-staged](https://github.com/okonet/lint-staged) - запуск команд на staged файлах
[husky](https://github.com/typicode/husky) git-хуки

Сборка и установка
[yarn](https://yarnpkg.com/en/) - установка и запуск
[webpack]( https://github.com/webpack/docs) - сборка приложения в итоговые файлы для подключения
[babel](https://babeljs.io/) - компиляция в es5

Готовые компоненты интерфейса
rc-dialog, rc-dropdown, rc-select https://github.com/react-component
[react-datetime](https://github.com/YouCanBookMe/react-datetime)
[react-dropzone](https://github.com/react-dropzone/react-dropzone)
[react-input-mask](https://github.com/sanniassin/react-input-mask)
[react-loading](https://github.com/fakiolinho/react-loading)
[react-youtube](https://github.com/troybetz/react-youtube)
