Тестирование
============

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


## 1. Компоненты

Проверяем работу компонента изолированно от окружения.
Тестируется каждый React-компонент по отдельности.

### 1.1 Connectors

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

### 1.2 Containers

Проверяем результат рендера компонента, результаты выполнения методов.
Обращаем внимание на наличие условий в коде компонента, проверяем поведение в зависимости от условий.
При тестировании методов стараемся не залазить во внутреннее состояние компонента, ориентироваться на внешние 
признаки, например, поменялся результат рендера или вызвался action.

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

#### 1.3. Views

Проверяем результат рендера компонента.
Тестируем вывод разных сущностей по условиям, передачу props в другие компоненты, вывод других компонентов 
(иногда объединяем в один `it` - проверяем  с помощью `contains`).

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


#### 2. Actions

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


#### 3. Reducers

Для каждого типа action объединяем проверки в `describe`. Проверяем получившийся state. Проверяем также случай по 
умолчанию с пустым action.

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

#### 4. Sagas

Запускаем сагу с помощью библиотеки [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store) и проверяем 
результат выполнения - например, через store прошел какой-то action, вызвалась функция.

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
