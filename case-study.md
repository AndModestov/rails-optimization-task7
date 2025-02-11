### Шаг 1. Сбор DX-метрики

- Развернул Chronograf (репозиторий переехал https://github.com/influxdata/sandbox)
- Добавил и настроил Influxer
- Отправить данные из приложения в influxDB так и не вышло, т.к приложение разворачивается в докере,
  Chronograf - также. Спустя относительно продолжительное время так и не удалось настроить доступ к localhost:8086 из контейнера.
  (Errno::EADDRNOTAVAIL: Failed to open TCP connection to localhost:8086 (Cannot assign requested address - connect(2) for "localhost" port 8086))
- В тестовых целях опробовал отправку данных для DEV.to
- В качестве метрики оптимизации приложения будем использовать время прохождения тестового билда, без отправки в influxDB, к сожалению

### Шаг 2. Первый прогон тестов

- В проекте - 2806 examples
- На данный момент уже используется parallel_rspec.
- Текущее время прохождения ~ 3:50.

### Шаг 3. Common pitfalls fix

- Sidekiq используется в режиме fake
- Отключение DatabaseCleaner-а (удаление) и отключение логов ускорило прохождение тестов значительно ~ 1:54.

### Шаг 4. Профилирование - 1

- Запуск тестов в режиме `--profile` показал, что 10 slowest examples занимают 5% времени прохождения тестов.
  Большая часть тестов из списка выполняется около секунды, не думаю что оптимизация даст весомые результаты.
- Для примера сгенерировал несколько отчетов, оптимизировал по мелочи несколько тестов
- Также удалось найти несколько легаси тестов и удалить их, т.к функционал уже не используется и тестировать его нет смысла.
- Текущее время прохождение тестов ~ 1:42

### Шаг 5. Профилирование - 2

- RSpecDissect показал что большую часть времени занимают блоки let и before.
- Воспользуемся хелперами let_it_be и before_all от test-prof.
- Замена сломала порядка 50% тестов. Чтобы не тратить уйму времени - ради эксперимента отрефакторим несколько тестов.
- В рамках небольшого набора тестов время сократилось незначительно, но если сделать замену глобальной - эффект может быть ощутимым.

### Шаг 6. Профилирование - 3

- factory-prof был опробован, весомых результатов достигнуть не удалось
- factory-doctor показал Total wasted time: 00:06.314, весомого эффекта оптимизация не даст, но инструмент был опробован

### Выводы.

- Были опробованы многие функции test-prof, позволяющие профилировать и ускорять тесты
- Время прогона тестов было сокращено  c почти 4 минут до ~ 1:34 (~ в 2.5 раза).
- Потенциал сокращения еще есть, но требует временных вложений. Так что взял на заметку
