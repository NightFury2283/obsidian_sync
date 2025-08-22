
[[Python]]
[[Фреймворк Django. Работа с проектами]]
[[Питон Практикум]]

# Unittest в Django: тестирование контента

План тестирования контента для проекта YaNews выглядит так:

1. Количество новостей на главной странице — не более 10.
2. Новости отсортированы от самой свежей к самой старой. Свежие новости в начале списка.
3. Комментарии на странице отдельной новости отсортированы от старых к новым: старые в начале списка, новые — в конце.
4. Анонимному пользователю не видна форма для отправки комментария на странице отдельной новости, а авторизованному видна.

Начнём по порядку: проверим, что количество новостей не превышает десяти.

Первым делом — фикстуры: нужно создать более десятка новостей, тогда можно будет понять, ограничивается ли их количество при выводе на главную.

Количество новостей, выводимых на страницу указывается в настройках проекта `settings.NEWS_COUNT_ON_HOME_PAGE`, поэтому для тестов создадим `(settings.NEWS_COUNT_ON_HOME_PAGE + 1)` новостей, чтобы их было заведомо больше, чем должно отобразиться на странице.

Первое же решение, которое приходит на ум — это создать цикл и на каждой итерации цикла создавать объекты.

```
# news/tests/test_content.py
from django.conf import settings
from django.test import TestCase

from news.models import News


class TestContent(TestCase):

    @classmethod
    def setUpTestData(cls):
        for index in range(settings.NEWS_COUNT_ON_HOME_PAGE + 1):
            News.objects.create(title=f'Новость {index}', text='Просто текст.') 
```

Или иначе:

```
    @classmethod
    def setUpTestData(cls):
        for index in range(settings.NEWS_COUNT_ON_HOME_PAGE + 1):
            news = News(title=f'Новость {index}', text='Просто текст.')
            news.save() 
```

При таком подходе на каждой итерации цикла будет отправляться запрос к БД. Это как минимум неэффективно; не страшно, если объектов всего чуть больше десяти; но что если нужно создать несколько сотен или тысяч объектов?

## Групповое создание объектов

Для одновременного создания нескольких объектов применяют метод [bulk_create()](https://docs.djangoproject.com/en/3.2/ref/models/querysets/#bulk-create):

```
    @classmethod
    def setUpTestData(cls):
        all_news = []
        for index in range(settings.NEWS_COUNT_ON_HOME_PAGE + 1):
            news = News(title=f'Новость {index}', text='Просто текст.')
            all_news.append(news)
        News.objects.bulk_create(all_news) 
```

Объекты, как и в прошлом примере, создаются в цикле; однако при этом не вызываются методы `save()` или `create()`: объекты хранятся в списке, в оперативной памяти.

Когда цикл завершён и все объекты созданы — вызываем метод `bulk_create()` и передаём в него список объектов.

Результат получится тот же, но при этом будет выполнен только один запрос к базе.

Тот же результат можно получить при помощи _list comprehension_:

```
    @classmethod
    def setUpTestData(cls):
        all_news = [
            News(title=f'Новость {index}', text='Просто текст.')
            for index in range(settings.NEWS_COUNT_ON_HOME_PAGE + 1)
        ]
        News.objects.bulk_create(all_news) 
```

А можно написать выражение, создающее объекты новостей внутри `bulk_create()`:

```
    @classmethod
    def setUpTestData(cls):
        News.objects.bulk_create(
            News(title=f'Новость {index}', text='Просто текст.')
            for index in range(settings.NEWS_COUNT_ON_HOME_PAGE + 1)
        ) 
```

В файл _news/tests/test_content.py_ импортируйте необходимые модули и создайте в нём класс `TestContent`. В методе `setUpTestData()` этого класса создайте одиннадцать объектов класса `News` — примените метод `bulk_create()` любым из описанных способов.

## Работа со словарём контекста в ответе

Следующий шаг — загрузить главную страницу и посчитать в ней количество постов.

Объекты, передаваемые в шаблон для отрисовки, хранятся в атрибуте `context` экземпляра класса `Response`. Список объектов из словаря `response.context` можно получить при помощи ключа `object_list`: `response.context['object_list']`. По умолчанию под ключом `object_list` хранятся объекты контекста.

Есть и другой ключ, он состоит из имени модели и окончания `_list`, в примере с моделью `News` — `response.context['news_list']`; под ним хранятся те же объекты, что и в `object_list`.

Вместо ключа с именем модели (`news_list`, в нашем примере) можно задать и любой другой ключ при помощи атрибута `context_object_name` в Class-Based View. Например, вот так:

```
# news/views.py
...

class NewsList(generic.ListView):
    """Список новостей."""
    model = News
    template_name = 'news/home.html'
    context_object_name = 'news_feed'

    ... 
```

Затем имя `news_feed` можно применять в тестах и в шаблонах.

Остаётся получить длину списка с объектами новостей и сравнить её с константой из настроек. Дополните код в файле _test_content.py_:

```
# news/tests/test_content.py
from django.conf import settings
from django.test import TestCase
# Импортируем функцию reverse(), она понадобится для получения адреса страницы.
from django.urls import reverse

from news.models import News


class TestHomePage(TestCase):
    # Вынесем ссылку на домашнюю страницу в атрибуты класса.
    HOME_URL = reverse('news:home')

    @classmethod
    def setUpTestData(cls):
        all_news = [
            News(title=f'Новость {index}', text='Просто текст.')
            for index in range(settings.NEWS_COUNT_ON_HOME_PAGE + 1)
        ]
        News.objects.bulk_create(all_news)

    def test_news_count(self):
        # Загружаем главную страницу.
        response = self.client.get(self.HOME_URL)
        # Код ответа не проверяем, его уже проверили в тестах маршрутов.
        # Получаем список объектов из словаря контекста.
        object_list = response.context['object_list']
        # Определяем количество записей в списке.
        news_count = object_list.count()
        # Проверяем, что на странице именно 10 новостей.
        self.assertEqual(news_count, settings.NEWS_COUNT_ON_HOME_PAGE) 
```

## Тестируем сортировку новостей

Переходим ко второму пункту плана: «Новости отсортированы от самой свежей к самой старой. Свежие новости в начале списка».

Модифицируем фикстуру создания новостей таким образом, чтобы у каждой новости была собственная уникальная дата.

Сейчас по умолчанию в качестве даты подставляется текущий день. Это указано в описании поля `date` модели `News`.

```
# news/models.py
...

class News(models.Model):
    ...
    date = models.DateField(default=datetime.today)

    class Meta:
        ordering = ('-date',)
        ... 
```

Установим для каждой новости в фикстуре собственную дату принудительно. Первым делом импортируем из библиотеки `datetime` классы `datetime` и `timedelta` (их имена написаны с маленькой буквы, но это классы).

```
from datetime import datetime, timedelta 
```

Класс `datetime` позволяет вызвать метод `today()`, возвращающий текущую дату, а при помощи класса `timedelta` (разница во времени) можно получить любую другую дату. Например, вот так:

```
from datetime import datetime, timedelta

# Текущая дата.
today = datetime.today()
# Вчера.
yesterday = today - timedelta(days=1)
# Завтра.
tomorrow = today + timedelta(days=1) 
```

Применим этот подход в фикстуре. Обновите код в файле _news/tests/test_content.py_:

```
# news/tests/test_content.py

# Импортируйте нужные классы. 
from datetime import datetime, timedelta
...


class TestHomePage(TestCase):
    HOME_URL = reverse('news:home')

    @classmethod
    def setUpTestData(cls):
        # Вычисляем текущую дату.
        today = datetime.today()
        all_news = [
            News(
                title=f'Новость {index}',
                text='Просто текст.',
                # Для каждой новости уменьшаем дату на index дней от today,
                # где index - счётчик цикла.
                date=today - timedelta(days=index)
            )
            for index in range(settings.NEWS_COUNT_ON_HOME_PAGE + 1)
        ]
        News.objects.bulk_create(all_news) 
```

Теперь в тесте можно получить даты новостей и убедиться, что новости отсортированы в нужном порядке:

- получим список дат из новостей,
- создадим второй такой же список и отсортируем его,
- сравним списки: если исходный список равен отсортированному — значит, новости отсортированы правильно.

Соберём список дат новостей, назовём его `all_dates` и создадим список `sorted_dates`, куда поместим результат сортировки списка `all_dates` — для этого применим функцию `sorted()`.

Новости должны быть отсортированы по убыванию даты — от самых свежих к самым старым. Чтобы выполнить такую сортировку, в функцию `sorted()` нужно передать аргумент `reverse=True`: он «перевернёт» отсортированный список (по умолчанию `sorted()` сортирует по возрастанию).

Сравним списки `all_dates` и `sorted_dates`. Списки можно сравнивать целиком: при этом сравниваются не только элементы, но и их положение в списке.

```
# news/tests/test_content.py
...

    def test_news_order(self):
        response = self.client.get(self.HOME_URL)
        object_list = response.context['object_list']
        # Получаем даты новостей в том порядке, как они выведены на странице.
        all_dates = [news.date for news in object_list]
        # Сортируем полученный список по убыванию.
        sorted_dates = sorted(all_dates, reverse=True)
        # Проверяем, что исходный список был отсортирован правильно.
        self.assertEqual(all_dates, sorted_dates) 
```

Этот тест похож на предыдущий (тестируем содержимое главной страницы, работаем со списком объектов), но принцип атомарности юнит-тестов требует, чтобы эти тесты были отдельными.

Если в одном тестирующем методе написать несколько assert-выражений, то при первой же провалившейся проверке выполнение конкретного теста прекратится, и до остальных assert-выражений программа просто не дойдёт. Поэтому здесь необходимы отдельные тесты: один тест — одна проверка.

Перенесите тест `test_news_order()` в файл _test_content.py_ и запустите: убедитесь, что всё работает корректно.

## Тестируем сортировку комментариев на странице новости

Для тестов, проверяющих отдельную страницу, лучше создать собственный тестовый класс. В нём понадобятся объекты, которые не нужны в других тестах, да и структура файла станет лучше: будет проще понять, где что тестируется.

Первым делом — подготовка данных. Комментарии отображаются на странице новости, значит, нужно создать

- новость,
- два комментария к ней,
- пользователя (кто-то же должен быть автором комментария).

При создании комментария с помощью методов `create()` или `bulk_create()` не удастся установить собственное значение для поля `created`; время создания комментария устанавливается автоматически, это задано в модели:

```
# news/models.py
...


class Comment(models.Model):
    ...
    created = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ('created',)
    
    ... 
```

Даже если передать в комментарии разные значения поля `created`, то в базу всё равно запишутся значения текущего времени. И с высокой вероятностью для двух объектов, создаваемых подряд, это время будет одинаковым: программный код работает быстро.

Применим другой подход. Создадим два комментария без применения `bulk_create()` в обычном цикле. На каждой итерации цикла после создания объекта изменим значение поля `created` через изменение атрибута и сохраним объект.

Необходимые объекты создадим в `setUpTestData()`. Адрес страницы с новостью получим через `reverse()`.

В адресе потребуется id новости, получить его можно будет только после создания экземпляра `News`.

```
# news/tests/test_content.py
...

# Импортируем функцию для получения модели пользователя.
from django.contrib.auth import get_user_model
...
# Дополнительно к News импортируем модель комментария.
from news.models import Comment, News

User = get_user_model()


class TestHomePage(TestCase):
    ...


class TestDetailPage(TestCase):

    @classmethod
    def setUpTestData(cls):
        cls.news = News.objects.create(
            title='Тестовая новость', text='Просто текст.'
        )
        # Сохраняем в переменную адрес страницы с новостью:
        cls.detail_url = reverse('news:detail', args=(cls.news.id,))
        cls.author = User.objects.create(username='Комментатор')
        # Запоминаем текущее время:
        now = datetime.now()
        # Создаём комментарии в цикле.
        for index in range(10):
            # Создаём объект и записываем его в переменную.
            comment = Comment.objects.create(
                news=cls.news, author=cls.author, text=f'Tекст {index}',
            )
            # Сразу после создания меняем время создания комментария.
            comment.created = now + timedelta(days=index)
            # И сохраняем эти изменения.
            comment.save()                  
```

Этот подход позволяет изменить время создания комментария. Но при запуске кода появится предупреждение Django:

```
RuntimeWarning: DateTimeField Comment.created received a naive datetime (2022-01-01 12:21:26.016576) while time zone support is active.
  warnings.warn("DateTimeField %s received a naive datetime (%s)" 
```

Смысл сообщения таков: в поле `created` передано время без указания часового пояса, хотя для этого поля поддерживается время с часовыми поясами.

Чтобы исправить ситуацию, можно воспользоваться встроенной функцией Django `timezone.now()`, которая импортируется из модуля `django.utils`.

Если в настройках проекта включено использование часовых поясов (а оно включено по умолчанию: `settings.USE_TZ = True`), то функция `timezone.now()` вернёт время с указанием часового пояса; если использование часовых поясов отключено — `timezone.now()` вернёт время без часового пояса.

Проверим: распечатаем результат вызова `datetime.now()`:

```
from datetime import datetime

print(datetime.now())
# Часовой пояс не указан:
# 2022-03-08 15:44:47.919437 
```

Напечатаем вызов `timezone.now()`:

```
from django.utils import timezone

print(timezone.now())
# Вместе с датой и временем передан и часовой пояс: "+00:00".
# 2022-03-08 12:44:47.919437+00:00 
```

По умолчанию время сохраняется в UTC (Всемирное координированное время, англ. _Coordinated Universal Time_), оно на 3 часа меньше, чем московское время. Работа с часовыми поясами в Django — это отдельная обширная тема, которая не касается напрямую тестирования, поэтому не будем в неё углубляться, при желании всё можно прочесть [в документации](https://docs.djangoproject.com/en/3.2/topics/i18n/timezones/).

Импортируйте `timezone` в код и замените строку `now = datetime.now()` на `now = timezone.now()`:

```
# news/tests/test_content.py
...
# Допишите новый импорт.
from django.utils import timezone
...

class TestDetailPage(TestCase):

    ...

    @classmethod
    def setUpTestData(cls):
        ...
        # Получите текущее время при помощи утилиты timezone.
        now = timezone.now()
        ... 
```

Теперь время передаётся в нужном формате, и при выполнении кода никаких предупреждений выводиться не будет.

Теперь можно писать тесты. Объекты комментариев хранятся в `response.context`, но их ещё нужно отыскать. Для этого можно заглянуть в шаблон _templates/news/detail.html._

В шаблоне видно, что комментарии можно получить методом `news.comment_set.all()`. Значит, нужно убедиться, что в словаре контекста есть ключ `news` (название ключа совпадает с названием модели). После этого получить из контекста объект `news` и методом `comment_set.all()` получить из него список комментариев.

Сортировку комментариев проверим примерно так же, как проверяли сортировку списка новостей: получим список всех временных меток комментариев, отсортируем их и убедимся, что отсортированный список идентичен исходному.

Разница в том, что сортировать комментарии надо в прямом порядке (комментарии должны отображаться в порядке «от старых к новым»), в то время как новости сортировались в обратном порядке.

```
# news/tests/test_content.py
...
    def test_comments_order(self):
        response = self.client.get(self.detail_url)
        # Проверяем, что объект новости находится в словаре контекста
        # под ожидаемым именем - названием модели.
        self.assertIn('news', response.context)
        # Получаем объект новости.
        news = response.context['news']
        # Получаем все комментарии к новости.
        all_comments = news.comment_set.all()
        # Собираем временные метки всех новостей.
        all_timestamps = [comment.created for comment in all_comments]
        # Сортируем временные метки, менять порядок сортировки не надо.
        sorted_timestamps = sorted(all_timestamps)
        # Проверяем, что временные метки отсортированы правильно.
        self.assertEqual(all_timestamps, sorted_timestamps) 
```

Добавьте тест в файл _test_content.py_ и проверьте, правильно ли он работает.

## Тест наличия формы в словаре контекста

Остался последний пункт плана. Для авторизованного пользователя на странице новости должна быть видна форма комментариев, а для анонимного — нет. Как именно рендерится HTML-форма и что там отображается — мы проверять не будем, но можем проверить, есть ли объект `form` в словаре контекста и относится ли этот объект к нужному классу.

Напишем два теста:

- первый тест проверит, что при запросе анонимного пользователя **форма не передаётся** в словаре контекста,
- второй тест проверит, что при запросе авторизованного пользователя **форма передаётся** в словаре контекста.

В тестах применим методы

- `assertNotIn()` — для проверки отсутствия объекта в словаре контекста,
- `assertIn()` — для проверки наличия объекта в словаре контекста,
- `assertIsInstance` — для проверки принадлежности объекта формы к нужному классу.

```
# news/tests/test_content.py

# Импортируем класс формы.
from news.forms import CommentForm


class TestDetailPage(TestCase):

    ...

    def test_anonymous_client_has_no_form(self):
        response = self.client.get(self.detail_url)
        self.assertNotIn('form', response.context)
        
    def test_authorized_client_has_form(self):
        # Авторизуем клиент при помощи ранее созданного пользователя.
        self.client.force_login(self.author)
        response = self.client.get(self.detail_url)
        self.assertIn('form', response.context)
        # Проверим, что объект формы соответствует нужному классу формы.
        self.assertIsInstance(response.context['form'], CommentForm) 
```

Перенесите код в файл _test_content.py_ и запустите тесты; убедитесь, что всё работает, как задумано.

## На заметку

- В словаре `response.context` можно изменить имя ключа, под которым хранится список объектов; в уроке приведён пример, как изменить `response.context['news_list']` на `response.context['news_feed']`. [Документация рассказывает об этом более детально](https://docs.djangoproject.com/en/3.2/topics/class-based-views/generic-display/#making-friendly-template-contexts).
- Получение связанных объектов (как в случае с получением комментариев из объекта `News`) подробно [описано в документации](https://docs.djangoproject.com/en/3.2/topics/db/queries/#related-objects).

## Домашнее задание

Напишите тесты контента в проекте YaNote. План тестирования у вас есть — продолжайте его реализовывать!

