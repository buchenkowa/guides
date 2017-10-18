# Правила работы со sphinx

В разделе представлены требования при разработке кода, взаимодействующего со sphinx.

## Термины

Поле - полнотекстовое поле, представлющее собой проиндексированное содержимое документа. Поле позволяет быстро найти докменты по заданным ключевым словам.
Атрибут - дополнительные значения, ассоциированные с каждым документов в индексе. Атрибут служит для сортировки и фильтрации списка документов при запросе.

## Добавление новых атрибутов в индекс

Новые атрибуты в индекс сфинкса добавляются через миграцию.
Добавлять атрибуты в test environment не нужно, так как перед прогоном тестов запускается полная переиндексация сфинкса, и все необходимые атрибуты и поля уже есть.
Каждый атрибут добавляется внутри блока begin - rescue.
На добавление атрибутов в дисковый индекс требуется увеличенный таймаут, по-умолчанию выставлен 5 секунд, необходимо в миграции перенастроить в 60 секунд.

Пример миграции добавления двух атрибутов в сфинкс:

```ruby
class AddPricesToProductIndex < ActiveRecord::Migration
  # Добавляем атрибуты в product_core (дисковый индекс), product_rt0, product_rt1 (Real-time индексы)
  INDICES = %w(product_core product_rt0 product_rt1).freeze
  # Таймаут чтения
  TIMEOUT = 60.seconds

  # Текущая конфигурация сфинкс
  def sphinx_config
    @sphinx_config ||= ThinkingSphinx::Configuration.instance.tap { |config| config.mysql_read_timeout = TIMEOUT }
  end

  # Клиент для работы с сфинкс
  def sphinx
    @sphinx ||= sphinx_config.mysql_client
  end

  def up
    # Пропускаем test environment
    return if Rails.env.test?

    INDICES.each do |index|
      begin
        sphinx.write("ALTER TABLE #{index} ADD COLUMN price FLOAT")
      rescue => e
        # Если заданный атрибут уже имеется (только для production среды) то пропускаем, иначе кидаем исключение
        if Rails.env.production? && !Rails.env.staging? && !error.message.include?('attribute already in schema')
          raise e
        end
      end
    end

    # Добавляем второй атрибут
    INDICES.each do |index|
      begin
        sphinx.write("ALTER TABLE #{index} ADD COLUMN discount_price FLOAT")
      rescue => e
        if Rails.env.production? && !Rails.env.staging? && !error.message.include?('attribute already in schema')
          raise e
        end
      end
    end
  end

  def down
    # Пропускаем test environment
    return if Rails.env.test?

    INDICES.each do |index|
      sphinx.write("ALTER TABLE #{index} DROP COLUMN price")
      sphinx.write("ALTER TABLE #{index} DROP COLUMN discount_price")
    end
  end
end
```

### Некоторые детали реализации добавления атрибутов

Во время добавления атрибутов в индекс сфинк, запросы в этот индекс невозможны из-за блокировки записи. Соответственно, в большой дисковый индекс добавление атрибута задержит запросы в него на несколько секунд.
Вновь добавленный атрибут для каждого документа имеет значение 0.
