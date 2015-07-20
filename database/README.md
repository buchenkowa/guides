# Требования к разработке БД
Раздел содержит т.н. tips & tricks & best practices по разработке баз данных и ни в коем случае не является руководством по их проектированию. Для изучения последнего рекомендуется

###### Литература
- Том Кайт: Oracle для профессионалов (2 тома)

В качестве основного хранилища данных в проектах организации выступает СУБД [Postgresql](http://www.postgresql.org/).

## Содержание
* [Общие положения](#Общие-положения)
    * [Инструменты разработки](#Инструменты-разработки)
    * [Использование моделей в миграциях](#Использование-моделей-в-миграциях)
* [Именование элементов](#Именование-элементов)
    * [Схемы](#Схемы)
    * [Таблицы](#Таблицы)
    * [Столбцы](#Столбцы)
    * [Индексы](#Индексы)
    * [Констрейнты на уникальность](#Констрейнты-на-уникальность)
    * [Внешние ключи](#Внешние-ключи)
* [Документирование](#Документирование)
* [Построение индексов](#Построение-индексов)
    * [Общие положения](#Общие-положения-1)
    * [Набор одиночных индексов VS составной индекс](#Набор-одиночных-индексов-vs-составной-индекс)
    * [Уникальные индексы](#Уникальные-индексы)
* [Констрейнты](#Констрейнты)
    * [FOREIGN KEY ON DELETE action](#foreign-key-on-delete-action)
    * [Режим отложенности (NOT DEFERRABLE / DEFERRABLE)](#Режим-отложенности-not-deferrable--deferrable)
* [Использование схем](#Использование-схем)
* [Выполнение миграций под нагрузкой](#Выполнение-миграций-под-нагрузкой)
    * [Создание и удаление индексов](#Создание-и-удаление-индексов)
    * [Создание констрейнта на уникальность](#Создание-констрейнта-на-уникальность)
    * [Ограничение времени выполнения блокирующих операций](#Ограничение-времени-выполнения-блокирующих-операций)
    * [Foreign Key c / на высоконагруженные таблицы](#foreign-key-c--на-высоконагруженные-таблицы)
    * [Создание столбца с условием NOT NULL DEFAULT](#Создание-столбца-с-условием-not-null-default)
* [Готовые решения типовых задач](#Готовые-решения-типовых-задач)
* [Другое](#Другое)
    * [SET [SESSION] / SET LOCAL](#set-session--set-local)
* [pgcompactor](#pgcompactor)

## Общие положения
Под разработкой базы данных подразумевается изменение ее структуры.

Любое изменение должно быть зафиксировано в виде [миграции](http://edgeguides.rubyonrails.org/active_record_migrations.html).

Миграции могут являться частью межпроектного кода, и потому применяемый в них SQL-синтаксис должен быть доступен во всех используемых проектами версиях Postgresql, другими словами, в самой младшей из них. Таковой является v9.2, и потому гайд будет ориентирован на нее.

:exclamation: установка расширений Postgresql не является частью разработки и должна производиться руками администраторов


#### Инструменты разработки

При решении простых типовых задач по разработке БД рекомендуется использовать ORM (ActiveRecord) [методы](http://api.rubyonrails.org/v3.1.0/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html) по взаимодействию с БД (или методы сторонних решений, таких как [foreigner](https://github.com/matthuhiggins/foreigner)) нежели SQL.

###### Пример
```ruby
# bad
execute "ALTER TABLE users ADD COLUMN nickname character varying(30);"

# good
add_column :users, :nickname, :string, :limit => 30
```

:exclamation: вопреки рекомендации приведенные в разделе примеры написаны на SQL с целью поддержания единой стилистики

###### Литература
- http://guides.rubyonrails.org/v3.1.0/migrations.html#migrations-are-classes

#### Использование моделей в миграциях

Различные подводные камни, связанные с использованием ActiveRecord моделей в миграциях, описаны в литературе ниже. Чтобы избежать подобных проблем, рекомендуется в миграциях отказаться от использования моделей и констант, определенных вне ее, в пользу SQL.

###### Пример
```ruby
# bad
Apress::Packets::Packet.find([PACKET_EMPTY, PACKET_SILVER_TEST]) do |packet|
  packet.is_paid = false
  packet.save!
end

# good
PACKET_EMPTY = 0
PACKET_SILVER_TEST = 6

execute <<-SQL
  UPDATE packets
  SET is_paid = false
  WHERE id IN (#{PACKET_EMPTY}, #{PACKET_SILVER_TEST})
SQL
```

###### Литература
- http://guides.rubyonrails.org/v3.1.0/migrations.html#using-models-in-your-migrations
- http://railsguides.net/change-data-in-migrations-like-a-boss/

## Именование элементов
#### Схемы

Название схемы должно отражать предметную область хранимых в ней структур и данных.

###### Пример

Компонент системы | Схема размещения структур и данных
-------|-------
Статистика посещений | statistics
Ночной импорт данных  | import
Акции | deals
РАССОЛ | mail_deliveries
Укроп | ukrop
СУНДУК | denormalization

#### Таблицы
Имя таблицы является производной от имени класса сущности, данные которой она хранит.

В нашем случае правило отражения имени класса на имя таблицы (маппинг) продиктовано используемой ORM - [ActiveRecord](https://github.com/rails/rails/tree/master/activerecord), и потому:

> имя таблицы = имя класса (без неймспейса) во множественном числе в нижнем регистре в [snake-case](https://ru.wikipedia.org/wiki/Snake_case)'е

###### Пример
Сущность  | Класс | Имя таблицы
------------- | ------------- | -------------
Компания  | Company | companies
Пользователь компании | CompanyUser | company_users
Пакет  | Apress::Packets::Packet | packets

Если в таблице хранятся данные разных (но имеющих общего предка) сущностей (см. [STI](http://api.rubyonrails.org/classes/ActiveRecord/Inheritance.html)), то таблица именуется на основе имени класса общего предка.

###### Пример
Сущность  | Класс | Имя таблицы
------------- | ------------- | -------------
Иконка  | Icon | images
Картинка новости | NewsImage | images

В некоторых случаях правило можно нарушить, явно указав в классе сущности имя соответствующей таблицы.

###### Пример
Сущность  | Класс | Имя таблицы
------------- | ------------- | -------------
Ассет Ckeditor | Ckeditor::Asset | ckeditor_assets
Очередь по очистке кеша компаний | CompanyCacheCleanQueue | company_cache_clean_queue

#### Столбцы

В зависимости от природы свойства, хранимого в колонке таблицы, можно выделить следующие рекомендации по именованию:

Свойство  | Пример именования | Комментарий
------------- | ------------- | -------------
Ссылка на другую сущность | rubric_id, company_id, parent_id | название сущности с постфиксом _id
Колличественное или качественное свойство сущности | position, content, size, previous_level | существительное или словосочетание
Временная метка события           | created_at, updated_at, deleted_at, actualized_at | глагол в прошедшем времени с постфиксом _at
Логическое свойство (булево) | is_active или active, is_deleted или deleted, has_not_inside_trait_values, company_site_messenger_is_visible, order_is_acquired | Если свойство выражается словом / словосочетанием, рекомендуется использовать префикс is

#### Индексы

Принято именовать индексы в соответствии с правилом, заложенным в метод ActiveRecord   [#add_index](http://apidock.com/rails/v3.1.0/ActiveRecord/ConnectionAdapters/SchemaStatements/add_index).

Правило: `index_table-name_on_col1_and_col2_and_..._colN`

###### Пример

```SQL
CREATE INDEX CONCURRENTLY index_company_users_on_user_id_and_company_id ON company_users USING btree (user_id, company_id);
```
или средствами ActiveRecord

```ruby
add_index :company_users, [:user_id, :company_id]
```
#### Констрейнты на уникальность

Правило: `uniq_table-name_on_col1_and_col2_and_..._colN`

###### Пример

```SQL
ALTER TABLE deals.offer_terms ADD CONSTRAINT uniq_offer_terms_on_offer_id UNIQUE(offer_id) DEFERRABLE INITIALLY DEFERRED;
```

#### Внешние ключи

Правило: `fk_table_column`, где
- table — имя ссылающейся таблицы
- column — имя столбца, являющегося ссылкой на запись из ссылаемой таблицы

###### Пример

```SQL
ALTER TABLE ONLY companies
    ADD CONSTRAINT fk_companies_moderated_by FOREIGN KEY (moderated_by) REFERENCES users(id) ON DELETE SET NULL DEFERRABLE;
```
:exclamation: гем `foreigner` по-умолчанию использует отличную от принятой схему именования.

## Документирование

Таблицы и столбцы таблиц необходимо снабжать информативными, понятными комментариями на русском языке. Выбор инструмента создания комментариев остается за вами.

###### Пример

Создание комментариев при помощи SQL
```SQL
COMMENT ON TABLE company_rubrics IS 'Привязки компаний к товарному рубрикатору';
COMMENT ON COLUMN calls.user_id IS 'Менеджер, совершающий звонок';
```

## Построение индексов
#### Общие положения
Следующие факторы играют важную роль в определении необходимости покрытия столбца индексом

Pro:
- столбец обладает высокой селективностью
- большое количество запросов с фильтрацией по столбцу
- таблица, которой принадлежит столбец, обладает большим количеством строк по факту или в ближайшей перспективе ( > 10Ка)

Cons:
- столбец обладает низкой селективностью
- малое количество запросов с фильтрацией по столбцу
- большой объем записи в таблицу, которой принадлежит столбец

При этом столбец в обязательном порядке покрывается индексом, если это:
- первичный ключ (primary key)
- внешний ключ (foreign key)

При создании нового индекса, необходимо выявить более ненужные индексы, и если таковые есть -- удалить.

###### Пример

```SQL
DROP INDEX CONCURRENTLY IF EXISTS index_users_on_email_ci;
CREATE UNIQUE INDEX CONCURRENTLY index_users_on_email_ci ON users (lower(email) varchar_pattern_ops);
```

При удалении индекса рекомендуется проверять его на существование.

###### Пример

```SQL
DROP INDEX CONCURRENTLY IF EXISTS "index_company_review_images_on_company_review_id";
```
#### Набор одиночных индексов VS составной индекс

При решении данного вопроса необходимо придерживаться рекомендаций, приведенных в [документации postgres](http://www.postgresql.org/docs/9.2/static/indexes-bitmap-scans.html):

>
If your workload includes a mix of queries that sometimes involve only column x, sometimes only column y, and sometimes both columns, you might choose to create two separate indexes on x and y, relying on index combination to process the queries that use both columns. You could also create a multicolumn index on (x, y). This index would typically be more efficient than index combination for queries involving both columns, but  it would be almost useless for queries involving only y, so it should not be the only index. A combination of the multicolumn index and a separate index on y would serve reasonably well. For queries involving only x, the multicolumn index could be used, though it would be larger and hence slower than an index on x alone. The last alternative is to create all three indexes, but this is probably only reasonable if the table is searched much more often than it is updated and all three types of query are common. If one of the types of query is much less common than the others, you'd probably settle for creating just the two indexes that best match the common types.

###### Пример

```SQL
CREATE INDEX CONCURRENTLY index_rubrics_on_lft_rgt_level ON rubrics USING btree (lft, rgt, cached_level);

CREATE INDEX CONCURRENTLY index_rubric_on_cached_level ON rubrics USING btree (cached_level);
```

#### Уникальные индексы

Вместо уникального индекса нужно использовать констрейнт на уникальность (там, где это возможно).

###### Пример

```SQL
ALTER TABLE deals.offer_regions ADD CONSTRAINT uniq_offer_regions_on_offer_id_and_region_id UNIQUE(offer_id, region_id) DEFERRABLE INITIALLY DEFERRED;
```

###### Литература
- http://www.postgresql.org/docs/9.2/static/indexes-unique.html

## Констрейнты
:exclamation: Структура БД должна **максимально жестко** контролироваться с помощью констрейнтов.

Следующие логические ограничения на значение в столбце в обязательном порядке должны быть отражены в виде констрейнта.

Логическое ограничение | Физическое ограничение (констрейнт)
---------------------------------- | -----------------------------------------------------
Значение (или набор) является идентификатором строки в таблице | PRIMARY KEY
Значение является ссылкой на запись в другой таблице | REFERENCES / FOREIGN KEY
Значение не может быть пустым | NOT NULL (при этом указывается дефолтное значение, если таковое имеет смысл)
Значение (или набор) должно быть уникальным | UNIQUE
Значение должно принадлежать заданному множеству | Реализуется путем создания множества в виде ENUM и указании его в качестве типа данных столбца
Значение является ограниченной  по длине строкой | указание variable-length with limit в качестве типа колонки

###### Пример
```SQL
-- создание первичного ключа
ALTER TABLE ONLY companies_without_domain ADD CONSTRAINT company_id_pkey PRIMARY KEY (company_id);

-- создание внешнего ключа
ALTER TABLE "user_subscribes" ADD CONSTRAINT "fk_user_subscribes_thematic_rubric_id" FOREIGN KEY (thematic_rubric_id) REFERENCES thematic_rubrics(id) ON UPDATE CASCADE ON DELETE CASCADE DEFERRABLE;

-- обеспечения наличия значения с указанием значения по-умолчанию
ALTER TABLE "statistics"."company_statistic_total_by_days" ALTER COLUMN yml_hits SET DEFAULT 0 NOT NULL;

-- обеспечения наличия значения
ALTER TABLE mail_deliveries.categories ALTER COLUMN category_type_id SET NOT NULL;

-- обеспечение уникальности значения
ALTER TABLE deals.offer_regions  ADD CONSTRAINT uniq_offer_regions_on_offer_id_and_region_id UNIQUE(offer_id, region_id)         DEFERRABLE INITIALLY DEFERRED;

-- ограничение множества значений
CREATE TYPE product_state AS ENUM
   ('pending',
    'accepted',
    'changed',
    'declined',
    'deleted',
    'undecided');
ALTER TABLE products ADD COLUMN state product_state DEFAULT 'pending'::product_state NOT NULL;

-- ограничение длины значения
ALTER TABLE users ADD COLUMN phone varchar(12);
```

###### Литература
http://www.postgresql.org/docs/9.2/static/ddl-constraints.html
http://www.postgresql.org/docs/9.2/static/datatype-character.html

#### FOREIGN KEY ON DELETE action
При создании констрейнта типа "внешний ключ" необходимо уделять внимание опции `ON DELETE action`.

Если действие, которое необходимо произвести с записями, ссылающимися на удаляемую строку, может быть реализовано на урове БД при помощи опции ON DELETE action (NO ACTION / RESTRICT / CASCADE / SET NULL / SET DEFAULT), то именно так оно и должно быть реализовано.

Иначе, разрешается описывать действие на уровне приложения при помощи определения before / after колбеков и / или указания опции `:dependent` при задании ассоциации в ссылаемой модели. При этом на уровне БД должно быть выставлено ON DELETE NO ACTION (по-умолчанию), либо ON DELETE RESTRICT.

###### Литература
- http://www.postgresql.org/docs/9.2/static/sql-createtable.html
- https://robots.thoughtbot.com/referential-integrity-with-foreign-keys

#### Режим отложенности (NOT DEFERRABLE / DEFERRABLE)
При создании констрейнта на уникальность или внешнего ключа необходимо уделять внимание и явно указывать режим отложенности констрейнта (по-умолчанию констрейнты не отлагаемые, NOT DEFERRABLE).

В каких случаях полезно создавать констрейнты отлагаемыми / отложенными, можно почерпать из литературы ниже.

Несмотря на то, что существует прием, позволяющий безболезненно "изменить" режим отложенности (только для внешнего ключа), при наличии сомнений рекомендуется создавать констрейнты с режимом `DEFERRABLE INITIALLY IMMEDIATE`.

###### Пример
```SQL
-- требуется отложенная проверка уникальности
ALTER TABLE "company_review_images" ADD CONSTRAINT "company_review_images_review_id_position_unique"
  UNIQUE ("company_review_id", "position") DEFERRABLE INITIALLY DEFERRED;

-- требуется отложенная проверка внешнего ключа
ALTER TABLE deals.offer_terms
  ADD CONSTRAINT offer_terms_offer_id_fk FOREIGN KEY (offer_id)
    REFERENCES deals.offers (id) MATCH SIMPLE
    ON UPDATE NO ACTION ON DELETE CASCADE
    DEFERRABLE INITIALLY DEFERRED;

-- требования на изначальную отложенность нет
ALTER TABLE seo_entities
  ADD CONSTRAINT fk_seo_entities_rubric_id FOREIGN KEY (rubric_id)
    REFERENCES rubrics (id) MATCH SIMPLE
    ON UPDATE NO ACTION ON DELETE NO ACTION DEFERRABLE INITIALLY IMMEDIATE;
```

###### Литература
- http://www.postgresql.org/docs/9.2/static/sql-set-constraints.html
- http://hashrocket.com/blog/posts/deferring-database-constraints
- http://www.depesz.com/2009/08/11/waiting-for-8-5-deferrable-uniqueness/
- http://sqlblog.com/blogs/alexander_kuznetsov/archive/2013/11/15/learning-postgresql-differences-in-implementation-of-constraints.aspx

## Использование схем

Структуры данных и данные системных компонент должны быть размещены в собственных схемах данных.

Тем самым обеспечивается дополнительная структуризация базы данных, простота выполнения дампа данных отдельных компонент и горизонтального масштабирования (путем переноса высоконагруженной схемы на отдельную машину).

## Выполнение миграций под нагрузкой
Данный раздел описывает известные приемы, позволяющие избежать длительных блокировок во время выполнения DDL-операций над БД.

###### Литература
- http://www.databasesoup.com/2013/11/alter-table-and-downtime-part-i.html
- http://www.databasesoup.com/2013/11/alter-table-and-downtime-part-ii.html
- http://blog.endpoint.com/2012/11/postgres-alter-column-problems-and.html

#### Создание и удаление индексов
Создание индекса на таблице приводит к блокировке записи в нее. Чтобы избежать блокировок, создавать и удалять индексы нужно конкурентно.

###### Пример

```SQL
CREATE INDEX CONCURRENTLY index_trait_rubrics_on_trait_id ON trait_rubrics;
DROP INDEX CONCURRENTLY IF EXISTS index_trait_rubrics_on_trait_id;
```

:exclamation: После выполнения конкурентного создания индекса, необходимо проверить его валидность.

Литература
 - http://www.postgresql.org/docs/9.2/static/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY

#### Создание констрейнта на уникальность
Во время создания констрейнта на уникальность, чтение и запись в таблицу заблокированы (что справедливо для большинства DDL-операций над таблицами).

Существует прием, позволяющий сократить длительность блокировок во время создания такого констрейнта и состоящий из двух шагов:

1. Конкурентное создание уникального индекса (не блокирующая операция)
2. Создание констрейнта на уникальность на основе существующего уникального индекса (блокирующая, но быстрая, поскольку индекс уже создан)

###### Пример

```SQL
CREATE UNIQUE INDEX CONCURRENTLY index_users_unique_on_phone ON users (phone);
ALTER TABLE users ADD CONSTRAINT uniq_users_on_phone UNIQUE USING INDEX index_users_unique_on_phone;
```
#### Ограничение времени выполнения блокирующих операций
Рекомендуется ограничивать максимальное время выполнения операций над таблицами, приводящим к блокировкам.

Для этого **внутри транзакции** необходимо выставить (с ключом LOCAL) адекватное значение параметра `statement_timeout` перед выполнением "опасных" операций.

###### Пример

```SQL
    BEGIN;
        SET LOCAL statement_timeout = '1s';

        -- locking stuff here
    END;
```

#### Foreign Key c / на высоконагруженные таблицы
Создание внешнего ключа требует взятия Access Exclusive Lock как на ссылающуюся, так и на ссылаемую таблицу на время создания и валидации констрейнта, что в случае объемных и высоконагруженных таблиц неизбежно приводит к длительным блокировкам на чтение и запись в обе таблицы.

При помощи опции `NOT VALID` можно создать внешний ключ, пропустив этап валидации констрейнта и снизив длительность блокировок. При этом валидность констрейнта гарантируется только для последующих за его созданием вставок и изменений в таблицу.

Валидацию можно произвести при помощи команды `VALIDATE CONSTRAINT`, дождавшись снижения нагрузки на БД.

###### Пример

```SQL
ALTER TABLE trait_products ADD CONSTRAINT fk_trait_products_trait_value_id FOREIGN KEY (trait_value_id) REFERENCES trait_values(id) NOT VALID;

ALTER TABLE trait_products VALIDATE CONSTRAINT fk_trait_products_trait_value_id;
```

Использование опции `NOT VALID` решает задачу безболезненного изменения режима отложенности констрейнта типа "внешний ключ" без нарушения условий констрейнта.

###### Пример
```SQL
-- ранее созданный внешний ключ с режимом отложенности 'DEFERRABLE INITIALLY IMMEDIATE'
ALTER TABLE trait_products ADD CONSTRAINT fk_trait_products_trait_value_id FOREIGN KEY (trait_value_id) REFERENCES trait_values(id) DEFERRABLE INITIALLY IMMEDIATE;

-- быстрое создание аналогичного констрейнта с нужным режимом отложенности 'DEFERRABLE INITIALLY DEFERRED'
ALTER TABLE trait_products ADD CONSTRAINT fk_deferred_trait_products_trait_value_id FOREIGN KEY (trait_value_id) REFERENCES trait_values(id) DEFERRABLE INITIALLY DEFERRED;

-- удаление более ненужного внешнего ключа
ALTER TABLE trait_products DROP CONSTRAINT fk_trait_products_trait_value_id;
```

###### Литература
- http://www.postgresql.org/docs/9.2/static/sql-altertable.html
- https://gocardless.com/blog/zero-downtime-postgres-migrations-the-hard-parts

#### Создание столбца с условием NOT NULL DEFAULT

Создание столбца путем выполнения
```SQL
 ALTER TABLE users ADD COLUMN hat_size TEXT NOT NULL DEFAULT 'L';
```
приводит к заполнению столбца `hat_size` дефолтным значением путем полной перезаписи таблицы. В это время чтение и запись в таблицу невозможны.

Чтобы избежать блокировок рекомендуется выполнять создание столбца с дефолтным значением следующим образом:

```SQL
BEGIN;
  -- создаем столбец
  ALTER TABLE users ADD COLUMN hat_size TEXT;

  -- указываем дефолтное значение
  ALTER TABLE users ALTER COLUMN hat_size SET DEFAULT 'L';
END;
```

Затем проставляем дефолтное значение / значения
 * небольшими пачками,
 * растягивая во времени,
 * вне транзакции,

что лучше всего оформить в виде rake-задачи с выводом прогресса.

```ruby
# пример rake-задачи
task :fill_hat_size_default_value, [:default_value, :batch_size, :sleep_time] => :environment do |_, args|
  default_value = args[:default_value] || raise('Не указано дефолтное значение')
  batch_size = (args[:batch_size] || 1000).to_i
  sleep_time = (args[:sleep_time] || 5).to_i

  records_cnt = User.where(:hat_size => nil).count
  raise 'Значение уже заполнено' if records_cnt.zero?

  batches_cnt = (records_cnt.to_f / batch_size).ceil
  progressbar = ProgressBar.create(:total => batches_cnt, :format => "%a %P% Processed: %c from %C")

  batches_cnt.times do
    User
      .where("id IN (SELECT id FROM users WHERE hat_size IS NULL LIMIT #{batch_size})")
      .update_all(:hat_size => default_value)

    progressbar.increment
    sleep(sleep_time)
  end

  progressbar.finish
end
```

и наконец

```SQL
-- создаем ограничение NOT NULL
ALTER TABLE users ALTER COLUMN hat_size SET NOT NULL;
```

###### Литература
- http://www.databasesoup.com/2013/11/alter-table-and-downtime-part-i.html
- https://github.com/jfelchner/ruby-progressbar

## Готовые решения типовых задач

- https://wiki.postgresql.org/wiki/Deleting_duplicates
- https://wiki.postgresql.org/wiki/Category:Administrative_Snippets

## Другое
#### SET [SESSION] / SET LOCAL
Выставление отличных от дефолтных настроек разрешено при жесткой необходимости, при этом
- изменение настроек *транзакции* разрешено для любого типа подключения,
- изменение настроек *сессии* разрешено **только для прямого** типа подключения.

#### Пример
Изменение настроек транзакции
```SQL
BEGIN;
  SET LOCAL enable_indexscan = FALSE;
  SET LOCAL enable_bitmapscan = FALSE;
  ... your stuff here
END; -- автоматический reset по окончании транзакции
```
Изменение настроек сессии (прямое подключение)
```SQL
SET enable_indexscan = FALSE;
SET enable_bitmapscan = FALSE;
... your stuff here
-- автоматический reset при закрытии подключения
```

Прямое подключение -- подключение, устанавливаемое с Postgresql. На каждое такое подключение создаются изолированные от остальных подключений серверный процесс и сессия (с дефолтными настройками, которые можно изменить), живущие до окончания подключения.

Непрямое подключение -- подключение, устанавливаемое с PgBouncer. PgBouncer держит пул предустановленных с Postgresql  (серверных) подключений, между которыми он распределяет команды из поступающих в него (клиентских) подключений. Закрытие клиентского подключения **не** приводит к закрытию серверного подключения.

На наших проектах PgBouncer работает в режиме `transaction pooling`, что на практике означает следующее:  команды одного клиентского подключения, не заключенные в одну транзакцию, могут быть выполнены в разных серверных подключениях, а, значит, и в разных сессиях. Следовательно, в худшем случае, выполнение команд
```SQL
SET statement_timeout = '1s';
ALTER TABLE products ...
RESET statement_timeout;
```
вне транзакции, приведет к тому, что время выполнения операции `ALTER TABLE` не будет ограничено сверху, а настройка сессии `statement_timeout` не будет сброшена.

###### Литература
- https://wiki.postgresql.org/wiki/PgBouncer
- http://www.depesz.com/2012/12/02/what-is-the-point-of-bouncing/
- http://www.postgresql.org/docs/9.2/static/sql-set.html
- http://www.postgresql.org/docs/9.3/static/sql-discard.html

## pgcompactor
pgcompactor -- инструмент борьбы с раздуванием таблиц.

###### Литература
- [Описание и примеры использования](https://conf.railsc.ru/display/DEV/pgcompactor)
- [Почему не жмется большая нагруженная таблица?](http://blog.endpoint.com/2015/01/a-few-postgresql-tricks.html)
- :exclamation: [Опасность использования ключа --reindex](https://conf.railsc.ru/display/DEV/pgcompactor?focusedCommentId=28901491#comment-28901491)
