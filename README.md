# Задача «TXT2HTML»     

Перед выполнением задания внимательно прочитайте:

- [О всех этапах проверки задания](https://github.com/urfu-2016/guides/blob/master/workflow/extra.md)
- [Как отправить пулл](https://github.com/urfu-2016/guides/blob/master/workflow/pull.md)
- [Как пройти тесты](https://github.com/urfu-2016/guides/blob/master/workflow/test.md)
- Правила оформления [javascript](https://github.com/urfu-2016/guides/blob/master/codestyle/js.md), [HTML](https://github.com/urfu-2016/guides/blob/master/codestyle/html.md) и [CSS](https://github.com/urfu-2016/guides/blob/master/codestyle/css.md) кода

## Основное задание

У нас есть интересный текст из блога Яндекса с комментариями.  
Предлагается превратить его в HTML документ в файле __index.html__.

Условия:

* Необходимо соблюдать семантику
* Необходимо использовать не менее 25 уникальных элементов
* В тексте упоминается алгоритм Флетчера – добавьте его в документ c помощью **разметки**:
<img src="https://img-fotki.yandex.ru/get/15487/32167648.0/0_13347f_349f618a_X5L" width="450">

Собственно текст:

```text
Блог компании Яндекс.

ЯНДЕКС.ПОЧТА: КАК МЫ ИЗМЕРЯЕМ СКОРОСТЬ ЗАГРУЗКИ И УЛУЧШАЕМ ЕЁ

Если ваш сайт медленно грузится, вы рискуете тем, что люди не оценят ни то,
какой он красивый, ни то, какой он удобный. Никому не понравится, когда все
тормозит. Мы регулярно добавляем в Яндекс.Почту новую функциональность,
иногда — исправляем ошибки, а это значит, у нас постоянно появляются новый код
и новая логика. Всё это напрямую влияет на скорость работы интерфейса.

Что мы измеряем

Этапы первой загрузки:
* подготовка;
* загрузка статики (HTTP-запрос и парсинг);
* исполнение модулей;
* инициализация базовых объектов;
* отрисовка.

Этапы отрисовки любой страницы:
* подготовка к запросу на сервер;
* запрос данных с сервера;
* шаблонизация;
* обновление DOM.

— «Ок, теперь у нас есть метрики, мы можем отправить их на сервер» - говорим мы
— «Что же дальше?» - вопрошаете вы
— «А давай построим график!» - отвечаем мы
— «А что будем считать?» - уточняете вы

Как вы знаете, медиана – это серединное, а не среднее значение в выборке.
Если у нас имеются числа 1, 2, 2, 3, 8, 10, 20, то медиана – 3, а среднее – 6,5.
В общем случае медиана отлично показывает, сколько грузится средний пользователь.

В случае ускорения или замедления медиана, конечно, изменится. Но она не может
рассказать, сколько пользователей ускорилось, а сколько замедлилось.

APDEX – метрика, которая сразу говорит: хорошо или плохо. Метрика
работает очень просто. Мы выбираем временной интервал [0; t], такой, что если
время показа страницы попало в него, то пользователь счастлив. Берем еще один
интервал, (t; 4t] (в четыре раза больше первого), и считаем, что если страница
показана за это время, то пользователь в целом удовлетворен скоростью работы,
но уже не настолько счастлив. И применяем формулу:

(кол-во счастливых пользователей + кол-во удовлетворенных / 2) / (кол-во всех).
Получается значение от нуля до единицы, которое, видимо, лучше всего показывает,
хорошо или плохо работает почта.

Как мы измеряем

Сейчас модуль обновления сам логирует все свои стадии, и можно легко понять
причину замедления: медленнее стал отвечать сервер либо слишком долго
выполняется JavaScript. Выглядит это примерно так:

this.timings['look-ma-im-start'] = Date.now();
this.timings['look-ma-finish'] = Date.now();

C помощью Date.now() мы получаем текущее время. Все тайминги собираются и при
отправке рассчитываются. На этапах разница между “end” и “start” не считается,
а все вычисления производятся в конце:

var totalTime = this.timings['look-ma-finish'] - this.timings['look-ma-im-start'];

И на сервер прилетают подобные записи:

serverResponse=50&domUpdate=60

Как мы ускоряем

Чтобы снизить время загрузки почты при выходе новых версий,
мы уже делаем следующее:

* включаем gzip;
* выставляем заголовки кэширования;
* фризим CSS, JS, шаблоны и картинки;
* используем CDN;

Мы подумали: «А что если хранить где-то старую версию файлов, а при выходе новой
передавать только diff между ней и той, которая сохранена у пользователя?»
В браузере же останется просто наложить патч на клиенте.

На самое деле эта идея не нова. Уже существуют стандарты для HTTP — например,
RFC 3229 «Delta encoding in HTTP» и «Google SDHC», — но по разным причинам они
не получили должного распространения в браузерах и на серверах.

Мы же решили сделать свой аналог на JS. Чтобы реализовать этот метод обновления,
начали искать реализации diff на JS. На популярных хостингах кода нашли
библиотеки:
- VCDiff
- google-diff-patch-match

Для окончательного выбора библиотеки нам нужно сравнить:

Библиотека      | IE 9          | Opera 12
----------      | ----          | --------
vcdiff          | 8             | 5
google diff     | 1363          | 76

После того как мы определились с библиотекой для диффа, нужно определиться с тем,
где и как хранить статику на клиенте.

Формат файла с патчами для проекта выглядит так:
[
    {
        "k": "jane.css",
        "p": [patch],
        "s": 4554
    },
    {
        "k": "jane.css",
        "p": [patch],
        "s": 4554
    }
]

То есть это обычный массив из объектов. Каждый объект — отдельный ресурс. У
каждого объекта есть три свойства. k — названия ключа в localStorage для этого
ресурса. p — патч для ресурса, который сгенерировал vcdiff. s — чексумма для
ресурса актуальной версии, чтобы потом можно было проверить правильность
наложения патча на клиенте. Чексумма вычисляется по алгоритму Флетчера.

Алгоритм Бройдена — Флетчера — Гольдфарба — Шанно (BFGS)
— итерационный метод численной оптимизации, предназначенный для
нахождения локального максимума/минимума нелинейного функционала
без ограничений.

Почему именно алгоритм Флетчера, а не другие популярные алгоритмы вроде:
CRC16/32 - алгоритм нахождения контрольной суммы, предназначенный для проверки
целостности данных
md5 - 128-битный алгоритм хеширования. Предназначен для создания «отпечатков»
или дайджестов сообщения произвольной длины и последующей проверки
их подлинности.

Потому что он быстрый, компактный и легок в реализации.

Итог

Фактически мы экономим 80-90% трафика. Размер загружаемой статитки в байтах:

Релиз	| С патчем     | Без патча
7.7.20  | 397          | 174 549
7.7.21  | 383          | 53 995
7.7.22  | 483          | 3 995

Автор: @doochik
С++ разработик
Электронная почта: (doochik@yandex-team.ru)
Компания: Яндекс

Комментарии (3):

- Mogaika (mogaika@yandex-team.ru) 30 ноября 2014 в 17:05

  А можете привести сравнение, на сколько быстрее грузится lite версия?

- JIguse (mrawesome@yandex.ru) 29 ноября 2014 в 21:30

  Спасибо за статью, познавательно. Здорово, что Яндекс делится некоторыми
  подробностями о внутренней работе сервисов.

- Brister (brist89@yandex-team.ru) 24 ноября 2014 в 13:13

  (кол-во счастливых пользователей + кол-во удовлетворенных / 2) / (кол-во всех).
  Получается значение от нуля до единицы, которое, видимо, лучше всего показывает,
  хорошо или плохо работает почта.

  наверное все-таки от 0.5 до 1

- alexeimois (test@yandex.ru) 22 ноября 2014 в 17:35

  Мы измеряем скорость загрузки с помощью Яндекс.Метрики:
  help.yandex.ru/metrika/reports/monitoring_timing.xml

© Яндекс, help@yandex.ru, Хохрякова, 10
```
