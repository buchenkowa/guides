Добавление сторонних зависимостей
=================================

Решение добавлять в кодовую базу любые сторонние зависимости (библиотеки, компоненты, утилиты) должно быть обязательно
согласовано с руководителем отдела. Необходимость добавления новой зависимости должна быть обоснована, также, 
предварительно, следует убедиться, что в кодовой базе организации, в каком-либо репозитории нет данной (или похожей)
функциональности. 

Если, где-то в кодовой базе организации уже присутствует функциональность, которая может решить поставленную задачу, то 
следует рассмотреть возможность использовать данную функциональность. Если данная функциональность еще не предполагает 
общего использования, то, предварительно, ее необходимо преобразовать в универсальную функциональность, в соответствии
с [правилами написания универсальной функциональности](../universal-code-rules/README.md).

Если, в существующей кодовой базе есть функциональность, позволяющая решить вашу задачу без добавления сторонней
зависимость лишь отчасти, то необходимо проанализировать какой вариант решения будет наиболее эффективным - доработка
существующей функциональности, либо добавление новой зависимости. 