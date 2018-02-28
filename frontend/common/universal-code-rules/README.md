Правила написания универсальной функциональности
================================================

Если вам кажется, что вашу бизнес-задачу можно решить с помощью написания универсальной функциональности, которая будет
полезна не только вам и может быть применена для решения широкого класса задач организации, то в процессе работы 
необходимо учитывать следующие принципы:

* Знания по универсальной функциональности не должны быть формализованы только в виде идеи и кода одного человека. Это 
  позволит снизить [bus factor](https://en.wikipedia.org/wiki/Bus_factor). Все важные знания должны быть зафиксированы
  в [базе знаний организации](https://github.com/abak-press/guides).

* Для принятия универсального функционала в кодовую базу организации необходимо выполнить 4 стадии:

  1. Проектирование с обязательным ревью руководителя отдела и всех заинтересованных лиц;
  2. Разработка;
  3. Документация с примерами;
  4. Код ревью с руководителем отдела и всех заинтересованных лиц.

  Если в ходе работы вы не придерживались данных принципов, но в итоге у вас появилась какая-то универсальная
  функциональность, или в существующем коде есть что-то, что может быть полезно другим, то для выделения этого 
  в универсальный модуль также необходимо выполнить эти 4 стадии. До тех пор функциональность не может считаться 
  универсальной и рекомендоваться к общему применению.

* На этапе оценки задачи, если вы считаете что ваше решение можно выполнить в виде универсального компонента, вы должны
  довести это до **руководителя отдела**, **ПМа**, **заказчиков** и заложить на это соответствующее время.