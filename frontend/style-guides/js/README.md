JS стайл гайд
=============

  - [Правила именования](#Правила-именования)

## Правила именования

  * Все переменные, названия методов, модулей и папок должны иметь понятные, простые и лаконичные имена 
    без сокращений.

    :warning: Плохо: 

    ```js
    // company_rf/submit_btn.js
    
    function showSubmitBtn() {...}

    function hideSubmitButtonOnCompanyRegistrationForm() {...}
    ```

    :heavy_check_mark: Хорошо:

    ```js
    // company_registration_form/submit_button.js
    
    function showSubmitButton() {...}

    function hideSubmitButton() {...}
    ```

  * Для именования папок и файлов используем `snake_case`. 

  * Для именования переменных и методов используем `camelCase`.

  * Для именования констант используем `SNAKE_UPPER_CASE`.

  * К имени приватного метода или переменной добавляем в начало префикс `_`.

  * Максимальное количество аргументов функции - 3, если нужно больше - применяем паттерн `facade` 
    (объявляем в качестве единственного аргумента функции объект, в котором размещаем нужные нам данные).

    :warning: Плохо: 

    ```js
    function foo(a, b, c, d) {
    }
    ```

    :heavy_check_mark: Хорошо:

    ```js
    function foo(facade) {
      //внутри объекта facade размещаем нужные нам данные
      facade.a();
    }
    ```

  * Имена методов должны содержать глагол, который явно идентифицирует действие, которое совершает данный метод.

     :warning: Плохо: 

    ```js
    function _listener() {...}

    function _formValidator() {...}

    function _data() {...}
    ```

    :heavy_check_mark: Хорошо:

    ```js
    function _addListeners() {...}

    function _validateForm() {...}

    function _sendData() {...}
    ```

    **Исключение** - методы с приставками `to` и `is`. Например `toString`, `isChecked`.
   
