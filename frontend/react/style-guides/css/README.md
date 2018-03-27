CSS 
===

* при написании css-стилей реакт-компонентов правило о группировке стилей селектора в одну строчку не действует. Вместо
  этого все стили и вложенные конструкции пишем в столбик.

  
  :warning: legacy-style: 

  ```sass
  .wrapper { background: #fff; color: #000; cursor: default; margin: 5px; }

  .item { 
    display: block; float: left; width: 50px; height: 50px; border: solid black 1px;
    &:after { width: 0; height: 0; display: block; content: ''; clear: both; }
  }
  ```


  :heavy_check_mark: react-style:
  
  ```sass
  .wrapper { 
    background: #fff; 
    color: #000; 
    cursor: default; 
    margin: 5px; 
  }

  .item { 
    display: block;
    float: left; 
    width: 50px; 
    height: 50px; 
    border: solid black 1px;

    &:after { 
      width: 0; 
      height: 0; 
      display: block; 
      content: ''; 
      clear: both; 
    }
  }
  ```

  Пользуясь данным правилом, стоит помнить, что реакт-компоненты должны быть максимально декомпозированы, изолированы и
  не перегружены функциональностью или стилями. Лучше написать один маленький изолированный компонент с минимум-стилей,
  чем огромного универсального монстра с несколькими килобайтами css-a.
