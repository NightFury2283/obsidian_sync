
[[Python]]
[[Питон Практикум]]
[[Фреймворк Django. Работа с проектами]]


Обычно в проекте есть какие-то сущности, которые в Django мы могли бы назвать моделями; есть объекты, которые пользователь может просматривать в списке или по отдельности; эти объекты можно создавать, редактировать и удалять (иногда эти операции доступны только администратору проекта, а иногда — всем пользователям). Такими объектами могут быть товары в интернет-магазине, статьи в энциклопедии или в документации, описания отдельных продуктов или услуг.


![[Pasted image 20250527145859.png]]


Эти классы называются **Class-Based Views**, **CBV** — представления на основе классов. **Представлениями** (_views_) называют классы и функции, отвечающие за обработку запроса и генерацию ответа.

**Class-Based Views** позволяют избавиться от рутинной работы и создавать стандартные представления, просто наследуясь от встроенных классов: в Django встроены CBV для самых различных целей.


## Общие принципы применения CBV

Как и view-функции, CBV предназначены для обработки запроса и подготовки ответа. Хранят CBV, как и view-функции, в директории приложения в файле _views.py_.

В зависимости от задачи CBV наследуются от встроенных классов, например:

- для обработки запроса на получение списка объектов разработчик наследует свой CBV от встроенного класса `ListView`;
- для отображения отдельного объекта CBV наследуют от встроенного класса `DetailView`;
- для управления формой, предназначенной для создания объектов, пользовательский CBV наследуют от встроенного класса `CreateView`.

Имена пользовательским CBV традиционно дают по схеме `Имя_моделиИмя_CBV`, например — `BirthdayListView` или `BirthdayCreateView`.

Создание класса для отображения **списка объектов** модели `Birthday` будет выглядеть так:


```python
# views.py
from django.views.generic import ListView


class BirthdayListView(ListView):
    ...
```


Class-Based Views точно так же, как и view-функции, передают словарь контекста в определённый шаблон; шаблоны работают так же, как и в случае с view-функциями.

В файле `urls.py` есть отличие в синтаксисе:


```python
# any_app/urls.py
...
urlpatterns = [
    # Так маршрут связывают с view-функцией, это знакомо:
    path('list/', views.app_list, name='list'),

    # А вот так маршрут связывают с CBV: имя_класса.as_view()
    path('list/', views.AppListView.as_view(), name='list'),
]
...
```


Изменим приложение **birthday** — заменим view-функции на CBV.

## ListView — класс для отображения списка объектов

Посмотрим, как при использовании CBV будет выглядеть код, который постранично выводит список людей с их днями рождения.

Импортируйте в файл _birthday/views.py класс_ ListView и создайте CBV `BirthdayListView` — этот класс заменит view-функцию `birthday_list()`:


```python
# birthday/views.py
...
from django.views.generic import ListView
...

# Наследуем класс от встроенного ListView:
class BirthdayListView(ListView):
    # Указываем модель, с которой работает CBV...
    model = Birthday
    # ...сортировку, которая будет применена при выводе списка объектов:
    ordering = 'id'
    # ...и даже настройки пагинации:
    paginate_by = 10
```


Теперь перенастроим маршрутизацию: в файле _birthday/urls.py_ в маршруте с `name='list'` замените вызов view-функции `birthday_list` на обращение к методу `as_view()` класса `BirthdayListView`. Этот метод есть у всех встроенных CBV и наследуется в пользовательских классах.


```python
# birthday/urls.py
...
urlpatterns = [
    ...
    path('list/', views.BirthdayListView.as_view(), name='list'),
    ...
```



Всё! Больше никаких изменений делать не надо.

Запустите проект.

В классе `BirthdayListView` не описано обращение к БД, не указан шаблон, не вызван `Paginator`, не объявлен словарь `context` — и, несмотря на это, всё работает!

Как всегда — никакой магии: все стандартные операции спрятаны под капот класса `ListView` и его наследника — `BirthdayListView`.

**Где обращение к БД?**

Под капотом. Класс знает, с какой моделью работать (`model = Birthday`), знает, чего от него ждут (вывести список объектов), и знает, как отсортировать этот список. Этого ему достаточно, чтобы выполнить свою работу.

**Где пагинатор?**

Под капотом. Есть список объектов, есть количество записей на страницу — этого достаточно. Остальное спрятано с глаз долой.

**Где указано имя шаблона?**

Под капотом. Имя шаблона можно не указывать, если он назван по определённым правилам: наследники класса `ListView` ищут шаблон `<название приложения>/<название модели>_list.html`; шаблон, подготовленный для списка, назван именно так: _birthday/birthday_list.html_.

**Где словарь** `context` **и что в нём?**

Сам словарь генерируется и передаётся под капотом. А содержит этот словарь объект страницы `page_obj` — это дефолтное название, под которым класс `ListView` передаёт объект страницы в шаблон.



Т.Е МЫ ЗАМЕНИЛИ ВОТ ЭТО:

```python
def birthday_list(request):

    # Получаем все объекты модели Birthday из БД.

    birthdays = Birthday.objects.order_by('id')

    paginator = Paginator(birthdays, 2)

    page_number = request.GET.get('page')

    page_obj = paginator.get_page(page_number)

    # Передаём их в контекст шаблона.

    context = {'page_obj': page_obj}

    return render(request, 'birthday/birthday_list.html', context)
```



Впрочем, при желании можно задать собственное название шаблона в атрибуте `template_name`, а содержимое словаря контекста можно описать явным образом с помощью метода `get_context_data()`.


документации; вот, например, [раздел, посвящённый ListView](https://docs.djangoproject.com/en/3.2/ref/class-based-views/flattened-index/#listview). А при регулярной работе с CBV вся необходимая информация довольно быстро запоминается.


## Класс CreateView — создание объектов модели

Сейчас в приложении **birthday** создание новых объектов организовано так:

- на основе модели `Birthday` посредством класса `ModelForm` создаётся форма;
- форма передаётся в шаблон;
- данные, отправленные через форму, передаются в объект формы для валидации и сохранения.

Все эти операции выполняет view-функция `birthday()`.

Для создания объектов в Django есть встроенный класс `CreateView`; напишем класс, который будет отвечать за создание объектов вместо функции `birthday()`, и посмотрим, какую выгоду это принесёт. Саму функцию `birthday()` пока что не удаляйте.

Создайте в файле _birthday/views.py_ класс `BirthdayCreateView`, наследник `CreateView`:


```python
# birthday/views.py
...
from django.views.generic import CreateView, ListView
from django.urls import reverse_lazy
...


class BirthdayCreateView(CreateView):
    # Указываем модель, с которой работает CBV...
    model = Birthday
    # Этот класс сам может создать форму на основе модели!
    # Нет необходимости отдельно создавать форму через ModelForm.
    # Указываем поля, которые должны быть в форме:
    fields = '__all__'
    # Явным образом указываем шаблон:
    template_name = 'birthday/birthday.html'
    # Указываем namespace:name страницы, куда будет перенаправлен пользователь
    # после создания объекта:
    success_url = reverse_lazy('birthday:list')
```


Как и в классе `ListView`, в описании класса `CreateView` необязательно указывать имя шаблона, но тогда шаблон должен называться по схеме `имя-модели_form.html`, то есть в нашем проекте имя шаблона должно быть _birthday_form.html_.

В приложении **birthday** шаблон называется иначе, так что его название нужно указать в явном виде через атрибут `template_name`.

View-функция `birthday()`, которая сейчас отвечает за создание новых объектов, использует форму `BirthdayForm`. Класс `CreateView` позволяет «сэкономить» и на форме: этот класс сам может создать форму на основе модели; нет необходимости описывать форму отдельно.

В атрибуте `fields` класса `CreateView` (точно так же, как в классе `ModelForm`) указывают, какие поля модели должны быть представлены в форме; в `fields` можно указать перечень полей, а можно использовать значение `__all__`.

Теперь нужно подправить настройки в _birthday/urls.py_:


```python
# birthday/urls.py
...
urlpatterns = [
    path('', views.BirthdayCreateView.as_view(), name='create'),
    ...
```


Всё работает: Django отрисовал форму и готов обработать её.

Однако форма не так хороша, как была: к форме, которую создал `CreateView`, не подключён виджет с датой — календарь не отображается, формат введённой даты не проверяется. Пользователю придётся самому угадать, в каком формате вводить дату.

Все остальные поля отображаются и работают без проблем: поэкспериментируйте, попробуйте передать валидные и невалидные данные, посмотрите, сохраняется ли отправленная информация.

Такой способ хорош, чтобы быстро и без лишних трудозатрат создать простую работающую форму, например — для прототипа какого-то сервиса.

Однако наш проект — не прототип, а серьёзный действующий сервис. И форма ему требуется полноценная, с виджетом-календарём и контролем имён: «участников The Beatles не пускать!».

Проверка имён и виджет-календарь настроены в форме `BirthdayForm`, а в форме, созданной классом `BirthdayCreateView`, их нет.

Класс `CreateView` может создать собственную форму, но может использовать форму, созданную отдельно, через класс `ModelForm`.

Применим эту полезную возможность и подключим форму `BirthdayForm` к классу `BirthdayCreateView`: для этого вместо атрибута `fields` нужно указать атрибут `form_class`; значением этого атрибута будет `BirthdayForm`.


```python
# birthday/views.py

class BirthdayCreateView(CreateView):
    model = Birthday
    # Указываем имя формы:
    form_class = BirthdayForm
    template_name = 'birthday/birthday.html'
    success_url = reverse_lazy('birthday:list')
```


Всё работает: Django отрисовал форму и готов обработать её.

Однако форма не так хороша, как была: к форме, которую создал `CreateView`, не подключён виджет с датой — календарь не отображается, формат введённой даты не проверяется. Пользователю придётся самому угадать, в каком формате вводить дату.

Все остальные поля отображаются и работают без проблем: поэкспериментируйте, попробуйте передать валидные и невалидные данные, посмотрите, сохраняется ли отправленная информация.

Такой способ хорош, чтобы быстро и без лишних трудозатрат создать простую работающую форму, например — для прототипа какого-то сервиса.

Однако наш проект — не прототип, а серьёзный действующий сервис. И форма ему требуется полноценная, с виджетом-календарём и контролем имён: «участников The Beatles не пускать!».

Проверка имён и виджет-календарь настроены в форме `BirthdayForm`, а в форме, созданной классом `BirthdayCreateView`, их нет.

Класс `CreateView` может создать собственную форму, но может использовать форму, созданную отдельно, через класс `ModelForm`.

Применим эту полезную возможность и подключим форму `BirthdayForm` к классу `BirthdayCreateView`: для этого вместо атрибута `fields` нужно указать атрибут `form_class`; значением этого атрибута будет `BirthdayForm`.

```python
# birthday/views.py

class BirthdayCreateView(CreateView):
    model = Birthday
    # Указываем имя формы:
    form_class = BirthdayForm
    template_name = 'birthday/birthday.html'
    success_url = reverse_lazy('birthday:list')
```


Форму, созданную через `ModelForm`, можно настроить гораздо гибче, чем форму, которую создаёт класс `CreateView`. Зато `CreateView` позволяет сгенерировать форму гораздо быстрее и проще: не приходится описывать кучу разных классов и функций, один класс заменяет собой и форму, и view-функцию по её обработке.

Свойство `success_url` в классе `CreateView` отвечает за переадресацию после успешного создания объекта.

Путь для редиректа указывается в функции `reverse_lazy()`: она, как и функция `reverse()`, возвращает строку с URL нужной страницы. Однако `reverse_lazy()` срабатывает только при непосредственном обращении к CBV во время работы веб-сервера, а не на этапе запуска проекта, когда импортируются все классы. В момент запуска проекта карта маршрутов может быть ещё не сформирована, и использование обычного `reverse()` вызовет ошибку.

Запустите проект и проверьте, что после создания нового объекта вас переадресует на страницу со списком объектов; в конце этого списка должен появиться новый объект.

## Класс UpdateView — редактирование объекта

Здесь всё рутинно, скучно и предсказуемо: класс `UpdateView` — это практически копия класса, создающего объекты, только родительский класс другой — `UpdateView`.


```python
# birthday/views.py
...
from django.views.generic import CreateView, ListView, UpdateView
...


class BirthdayUpdateView(UpdateView):
    model = Birthday
    form_class = BirthdayForm
    template_name = 'birthday/birthday.html'
    success_url = reverse_lazy('birthday:list')
```


Измените маршрут `edit` в _birthday/urls.py_ — и класс заработает.


## Миксины

В классах создания и редактирования объектов есть повторяющийся код: в них используются одинаковые атрибуты; убрать дублирование можно с помощью класса-миксина (от английского _mix in_ — «вмешивать, подмешивать»).

**Миксинами** называют вспомогательные классы, с помощью которых в наследуемый класс можно добавить необходимые атрибуты и методы, которые хранит этот класс-миксин. Когда какой-то класс наследуется от другого (например, когда `BirthdayCreateView` наследуется от `CreateView`) — к родительскому классу можно «подмешать» миксин; в результате наследуемый класс унаследует атрибуты и методы от обоих классов.


```python
# Создаём миксин.
class BirthdayMixin:
    model = Birthday
    form_class = BirthdayForm
    template_name = 'birthday/birthday.html'
    success_url = reverse_lazy('birthday:list')


# Добавляем миксин первым по списку родительских классов.
class BirthdayCreateView(BirthdayMixin, CreateView):
    # Не нужно описывать атрибуты: все они унаследованы от BirthdayMixin.
    pass


class BirthdayUpdateView(BirthdayMixin, UpdateView):
    # И здесь все атрибуты наследуются от BirthdayMixin.
    pass
```


## Класс DeleteView — удаление объекта

Продолжим рефакторинг проекта: заменим функцию `delete_birthday()` на CBV.

Для удаления объектов в Django есть класс `DeleteView`; унаследуем от него собственный класс `BirthdayDeleteView`, укажем в классе модель, имя шаблона и страницу для переадресации.


```python
# birthday/views.py
...
from django.views.generic import CreateView, DeleteView, ListView, UpdateView
...


class BirthdayDeleteView(DeleteView):
    model = Birthday
    template_name = 'birthday/birthday.html'
    success_url = reverse_lazy('birthday:list')
```


Поменяем маршрут в файле _birthday/urls.py_:


```python
# birthday/urls.py
...
path('<int:pk>/delete/', views.BirthdayDeleteView.as_view(), name='delete')
```


![[Pasted image 20250527204750.png]]


Всё работает, но страница удаления отличается от того, что было раньше: на странице не отображается id и содержимое удаляемой записи.

Чтобы вывести эти значения — шаблон обращается к объекту `form.instance`:

```html
<!-- birthday/birthday.html -->
...
{% with data=form.instance %}
   ...
   {{ data.first_name }} {{ data.last_name }}
   ...
```


Однако класс `DeleteView` не передаёт в шаблон объект `form` — при удалении форма с объектом не нужна на странице: требуется лишь пустая форма с кнопкой для отправки POST-запроса на удаление.

Создадим для страницы удаления объекта отдельный шаблон. Дадим ему имя, которое рекомендуется в [документации](https://docs.djangoproject.com/en/3.2/topics/class-based-views/generic-editing/#model-forms), — `имя-модели_confirm_delete.html`: _birthday_confirm_delete.html_. Как и в других CBV, в `DeleteView` можно не указывать имя шаблона, если его имя соответствует ожидаемому.

Раз уж у страницы удаления объекта теперь отдельный шаблон — немного улучшим его:

- на кнопке напишем «Удалить»;
- вместо заголовка «Калькулятор ожидания дня рождения» выведем информацию об удаляемом объекте.

Удаляемый объект доступен и в переменной `object`, и в переменной с названием модели — `birthday`. Для эксперимента обратимся к нему и через одну, и через другую переменную:


```html
<!-- birthday/birthday_confirm_delete.html -->
{% extends "base.html" %}
{% load django_bootstrap5 %}
{% block content %}
  <!-- Обращаемся через object -->
  <h1>Удалить запись {{ object.pk }}</h1>
  <div class="card col-4 m-3">
    <div class="card-header">
      <!-- Обращаемся через birthday -->
      {{ birthday.first_name }} {{ birthday.last_name }} - {{ birthday.birthday }}
    </div>
    <div class="card-body">
      <!-- В форме только csrf-токен и кнопка. Никакие другие поля тут не нужны.  -->
      <form method="post">
        {% csrf_token %}
        {% bootstrap_button button_type="submit" content="Удалить" %}
      </form>
    </div>
  </div>
{% endblock content %}
```


Теперь, когда есть шаблон с тем именем, которое ожидает класс `DeleteView`, то из описания класса можно удалить вызов шаблона; в классе останется всего две строки:

```python
# birthday/views.py
...

class BirthdayDeleteView(DeleteView):
    model = Birthday
    success_url = reverse_lazy('birthday:list')
```


Страница удаления объекта теперь выглядит вполне информативно:

![[Pasted image 20250527205213.png]]


## Больше миксинов!

Класс `BirthdayDeleteView` тоже можно описать через миксины, но в нём только два атрибута и миксин `BirthdayMixin` для него не подойдёт.

Можно скомпоновать код иначе:


```python
class BirthdayMixin:
    model = Birthday
    success_url = reverse_lazy('birthday:list')


class BirthdayCreateView(BirthdayMixin, CreateView):
    form_class = BirthdayForm
    template_name = 'birthday/birthday.html'


class BirthdayUpdateView(BirthdayMixin, UpdateView):
    form_class = BirthdayForm
    template_name = 'birthday/birthday.html'


class BirthdayDeleteView(BirthdayMixin, DeleteView):
    pass
```


Здесь есть место ещё для одного миксина: в него можно включить атрибуты `form_class` и `template_name`.

Создадим миксин `BirthdayFormMixin`, который будем «подмешивать» только к CBV создания и редактирования объекта.


```python
class BirthdayMixin:
    model = Birthday
    success_url = reverse_lazy('birthday:list')


class BirthdayFormMixin:
    form_class = BirthdayForm
    template_name = 'birthday/birthday.html'


class BirthdayCreateView(BirthdayMixin, BirthdayFormMixin, CreateView):
    pass


class BirthdayUpdateView(BirthdayMixin, BirthdayFormMixin, UpdateView):
    pass


class BirthdayDeleteView(BirthdayMixin, DeleteView):
    pass
```


