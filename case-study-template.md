# Case-study оптимизации

## Актуальная проблема
В нашем проекте возникла серьёзная проблема.

Необходимо было обработать файл с данными, чуть больше ста мегабайт.

У нас уже была программа на `ruby`, которая умела делать нужную обработку.

Она успешно работала на файлах размером пару мегабайт, но для большого файла она работала слишком долго, и не было понятно, закончит ли она вообще работу за какое-то разумное время.

Я решил исправить эту проблему, оптимизировав эту программу.

## Формирование метрики
Для того, чтобы понимать, дают ли мои изменения положительный эффект на быстродействие программы я придумал использовать такую метрику:
Время выполнения.

## Гарантия корректности работы оптимизированной программы
Программа поставлялась с тестом. Выполнение этого теста в фидбек-лупе позволяет не допустить изменения логики программы при оптимизации.

## Feedback-Loop
Для того, чтобы иметь возможность быстро проверять гипотезы я выстроил эффективный `feedback-loop`, который позволил мне получать обратную связь по эффективности сделанных изменений за 1 секунду.

Вот как я построил `feedback_loop`:
1. Профилирование.
2. Нахождение точки роста.
3. Минимальная оптимизация.
4. вычислили метрику - оценили, как изменение повлияло на метрику
5. перестроили отчёт, убедились, что проблема решена
6. записали полученные результаты
7. закоммитились
8. перешли к следующей итерации

## Вникаем в детали системы, чтобы найти главные точки роста
Для того, чтобы найти "точки роста" для оптимизации я воспользовался:

1. ruby-prof flat
2. ruby-prof Graph
3. ruby-prof callstack


Вот какие проблемы удалось найти и решить

### Ваша находка №1
- ruby-prof flat <Class::Date>#parse
- убрал парсинг даты и два цикла
- обработка тестовых данных сократилась с 0.0012 до 0.0009
- <Class::Date>#parse перестала быть главной точкой роста

### Ваша находка №2
- ruby-prof flat Array#select
- Добавил group_by
- значительно ускорить обработку файла на 20000 строк - с ~10 секунд до ~1
- Array#select перестал быть главной точкой роста

### Ваша находка №3
- ruby-prof flat проверка уникальности
- добавил Set
- обработка 100000 занимает 1.54
- исправленная проблема перестала быть главной точкой роста

### Ваша находка №4
- ruby-prof-callstack each
- убрал вложенность нескольких циклов
- полный файл ~28 секунд
- each все еще главная точка роста, но уже уложились в бюджет

## Результаты
В результате проделанной оптимизации наконец удалось обработать файл с данными.
Удалось улучшить метрику системы до 28 секунд и уложиться в заданный бюджет.



## Защита от регрессии производительности
Для защиты от потери достигнутого прогресса при дальнейших изменениях программы (к сожалению сейчас нет времени написать.)
