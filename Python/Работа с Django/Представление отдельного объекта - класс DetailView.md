

Замена view-функций на CBV заметно облегчила код, но при рефакторинге приложение утратило важный сервис — теперь нигде не отображаются результаты подсчёта времени, оставшегося до дня рождения.

Создадим страницу, где будет отображаться отдельная запись; на этой же странице будем отображать и количество дней, оставшихся до дня рождения.

В Django есть специальный view-класс для отображения отдельных объектов: `DetailView`. Унаследуем от него собственный класс — `BirthdayDetailView`; отображать отдельные записи будем на страницах с адресом вида _birthday/<pk>/._

Начнём с маршрутизатора:

```python
# birthday/urls.py
from django.urls import path

from . import views

app_name = 'birthday'

urlpatterns = [
    path('', views.BirthdayCreateView.as_view(), name='create'),
    path('list/', views.BirthdayListView.as_view(), name='list'),
    path('<int:pk>/', views.BirthdayDetailView.as_view(), name='detail'),
    path('<int:pk>/edit/', views.BirthdayUpdateView.as_view(), name='edit'),
    path('<int:pk>/delete/', views.BirthdayDeleteView.as_view(), name='delete'),
]
```


## DetailView

Теперь очередь шаблона и CBV.

Опишем представление для отдельного объекта; для начала достаточно объявить класс и в нём указать модель.


```python
# birthday/views.py
...
from django.views.generic import (
    CreateView, DeleteView, DetailView, ListView, UpdateView
)
...


class BirthdayDetailView(DetailView):
    model = Birthday
```



Теперь шаблон. Для начала выясним, какое имя шаблона ожидает увидеть класс `DetailView`. Проще всего будет заглянуть на [сайт, посвящённый CBV](https://ccbv.co.uk/projects/Django/3.2/django.views.generic.detail/DetailView/), — там есть вся необходимая информация:


Создайте шаблон с суффиксом __detail_ — он будет служить для отображения отдельного объекта модели `Birthday`:


```html
<!-- birthday/birthday_detail.html -->
{% extends "base.html" %}

{% block content %}
  ID записи: {{ object.id }}
  <hr>
  {% if birthday.image %}
    <div>
      <!-- Картинку сделаем побольше, чем на странице list/: высотой 200px -->
      <img src="{{ birthday.image.url }}" height="200">
    </div>
  {% endif %}
   <h2>Привет, {{ object.first_name }} {{ object.last_name }}</h2>      
  {% if birthday_countdown == 0 %}
    <p>С днём рождения!</p>
  {% else %}
    <p>Осталось дней до дня рождения: {{ birthday_countdown }}!</p>
  {% endif %}
{% endblock content %}
```


![[Pasted image 20250527210156.png]]


Работает! Но вот только не отрабатывает калькулятор, рассчитывающий количество дней.

Чтобы всё заработало — надо:

- получить словарь контекста и добавить в него ключ `birthday_countdown`,
- присвоить этому ключу значение, вычисленное функцией `calculate_birthday_countdown()`,
- для вычислений передать в `calculate_birthday_countdown()` дату рождения.

## Переопределение словаря контекста в DetailView

Чтобы дополнить или переопределить словарь контекста — применяют метод `get_context_data()`.

Переопределим этот метод, а в нём

- Получим словарь с контекстом из родительского метода `get_context_data()`:

```python
  context = super().get_context_data(**kwargs)
```

- Дополним словарь новым ключом; значением этого ключа будет вызов функции `calculate_birthday_countdown()`:

```python
  context['birthday_countdown'] = calculate_birthday_countdown(...)
  ```

В качестве аргумента в функцию нужно передать дату рождения — значение поля `birthday` объекта модели. Сам объект доступен в атрибуте `self.object`:


```python
  context['birthday_countdown'] = calculate_birthday_countdown(
      self.object.birthday
  )
```


В общем виде код получится такой:

```python
# birthday/views.py
class BirthdayDetailView(DetailView):
    model = Birthday

    def get_context_data(self, **kwargs):
        # Получаем словарь контекста:
        context = super().get_context_data(**kwargs)
        # Добавляем в словарь новый ключ:
        context['birthday_countdown'] = calculate_birthday_countdown(
            # Дату рождения берём из объекта в словаре context:
            self.object.birthday
        )
        # Возвращаем словарь контекста.
        return context
```

![[Pasted image 20250527210919.png]]


Чтобы пользователь мог попасть на отдельную страницу записи — добавьте в шаблон _birthday_list.html_ ссылку на страницу объекта, например, вот так:

```html
...
<div class="col-10">  
  <div>
    {{ birthday.first_name }} {{ birthday.last_name }} - {{ birthday.birthday }}<br>
    <a href="{% url 'birthday:detail' birthday.id %}">Сколько до дня рождения?</a>
  </div>
...
```


![[Pasted image 20250527211029.png]]


## Наводим порядок

Впереди предстоит ещё много работы с приложением **birthday**, и перед этим нужно немного «прибраться» в нём: убрать лишнее из файлов и упростить код.

Первым делом переименуйте шаблон _birthday.html_ в _birthday_form.html_: именно это название шаблона ожидается в CBV `CreateView` и `UpdateView`, а значит, из описания этих классов можно убрать атрибут `template_name`.

При этом миксин `BirthdayFormMixin` теряет смысл — вместо того чтобы упростить код, он раздувает его. Что ж, уберём из кода этот миксин и наследование от него, а атрибут `form_class = BirthdayForm` добавим в тело нужных классов.

Файл _views.py_ примет такой вид:


```python
# birthday/views.py
from django.views.generic import (
    CreateView, DeleteView, DetailView, ListView, UpdateView
)
from django.urls import reverse_lazy

from .forms import BirthdayForm
from .models import Birthday
from .utils import calculate_birthday_countdown


class BirthdayListView(ListView):
    model = Birthday
    ordering = 'id'
    paginate_by = 10


class BirthdayMixin:
    model = Birthday
    success_url = reverse_lazy('birthday:list')


class BirthdayCreateView(BirthdayMixin, CreateView):
    form_class = BirthdayForm


class BirthdayUpdateView(BirthdayMixin, UpdateView):
    form_class = BirthdayForm


class BirthdayDeleteView(BirthdayMixin, DeleteView):
    pass


class BirthdayDetailView(DetailView):
    model = Birthday

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['birthday_countdown'] = calculate_birthday_countdown(
            self.object.birthday
        )
        return context
```


Из шаблона _birthday_form.html_ нужно удалить всё лишнее — условия, связанные с удалением объекта, приветствие и вывод количества дней до дня рождения. Код шаблона должен стать примерно таким:

```html
<!-- birthday/birthday_form.html -->
{% extends "base.html" %}
{% load django_bootstrap5 %}

{% block content %}
  {% if "/edit/" in request.path %}
    <h1>Редактировать запись {{ form.instance.pk }}</h1> 
  {% else %}
    <h1>Создать запись</h1>
  {% endif %}
  <div class="card col-4">
    <div class="card-header">
      Калькулятор ожидания дня рождения
    </div>
    <div class="card-body">
      <form method="post" enctype="multipart/form-data">
        {% csrf_token %}
        {% bootstrap_form form %}
        {% bootstrap_button button_type="submit" content="Отправить" %}
      </form>
    </div>
  </div>
{% endblock content %}
```


## Редирект на страницу отдельной записи

После создания или редактирования записи пользователь перенаправляется на общий список записей, но это неудобно: чтобы убедиться, что запись создана или изменена, пользователю придётся отыскать её в списке — а список может оказаться довольно велик.

Но теперь в приложении появилась отдельная страница записи — и после создания или редактирования записи будет логично переадресовывать пользователя именно на неё.

Настроить переадресацию через атрибут CBV `success_url` не получится, ведь для этого надо динамически подставить в адрес id страницы. Есть два основных способа, которыми можно решить эту задачу:

1. Можно описать в CBV метод `get_success_url()`, который будет формировать нужную ссылку. Недостаток у этого метода в том, что во избежание дублирования кода придётся создать отдельный миксин и «подмешивать» его во все нужные CBV.
2. Другой вариант — описать в модели, с которыми работают CBV, метод `get_absolute_url()` — «получить абсолютный URL объекта». Тогда, если в CBV не указаны атрибут `success_url` или метод `get_success_url()`, которые обладают бо́льшим приоритетом, CBV будет обращаться именно к методу `get_absolute_url()`. И никаких дополнительных миксинов добавлять не потребуется. Именно такой способ предлагает использовать [документация](https://docs.djangoproject.com/en/3.2/topics/class-based-views/generic-editing/#model-forms).

Добавьте метод `get_absolute_url()` в модель `Birthday` в файле _birthday/models.py._



```python
# birthday/models.py
...
# Импортируем функцию reverse() для получения ссылки на объект.
from django.urls import reverse
...


class Birthday(models.Model):
    ...
    
    class Meta:
        ...

    def get_absolute_url(self):
        # С помощью функции reverse() возвращаем URL объекта.
        return reverse('birthday:detail', kwargs={'pk': self.pk})
```


Теперь можно убрать из классов `BirthdayCreateView` и `BirthdayUpdateView` атрибут `success_url`; после отправки формы пользователь будет переадресован на страницу конкретной записи. Значит, и миксин `BirthdayMixin` с атрибутом `success_url` тоже не особо нужен, его можно убрать.

Файл _views.py_ получится таким:


```python
# birthday/views.py
from django.views.generic import (
    CreateView, DeleteView, DetailView, ListView, UpdateView
)
from django.urls import reverse_lazy

from .forms import BirthdayForm
from .models import Birthday
from .utils import calculate_birthday_countdown


class BirthdayListView(ListView):
    model = Birthday
    ordering = 'id'
    paginate_by = 10


class BirthdayCreateView(CreateView):
    model = Birthday
    form_class = BirthdayForm


class BirthdayUpdateView(UpdateView):
    model = Birthday
    form_class = BirthdayForm


class BirthdayDeleteView(DeleteView):
    model = Birthday
    success_url = reverse_lazy('birthday:list')


class BirthdayDetailView(DetailView):
    model = Birthday

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['birthday_countdown'] = calculate_birthday_countdown(
            self.object.birthday
        )
        return context
```


## Преимущества CBV

- Меньшее количество кода.
- Бо́льшая скорость разработки.
- Меньшая вероятность ошибок: все стандартные CBV многократно протестированы разработчиками Django.
- Если по одному адресу нужно обрабатывать разные типы запросов, например GET и POST, как было в случаях с использованием форм, то можно разделить обработчики на разные методы, что делает структуру кода более чёткой и упрощает его понимание.
- Можно использовать все преимущества ООП, например создавать свои собственные классы и переиспользовать их в разных частях приложения.

Полный список Class-Based Views, встроенных в Django, можно посмотреть [в документации](https://docs.djangoproject.com/en/3.2/ref/class-based-views/). Помимо этого, есть [специальный сайт](https://ccbv.co.uk/projects/Django/3.2/), посвящённый Django CBV. Там удобная структура, перечислены все методы, атрибуты, родительские классы и потомки CBV и выложено много другой информации. Этот сайт рекомендован и в [документации Django](https://docs.djangoproject.com/en/3.2/ref/class-based-views/flattened-index/).

Код, написанный на CBV, выглядит намного проще и короче, чем те же представления, описанные во view-функциях.

CBV — мощный инструмент, но к нему надо привыкнуть: нужно знать или уметь быстро найти названия нужных атрибутов, методов, шаблонов; помнить, какие у атрибутов значения по умолчанию.

Но эти проблемы касаются любого инструмента, будь то фреймворк, библиотека или встроенный класс. И решения для этих проблем всегда под рукой: это поисковая система, документация и ваша память, в которой постепенно отложится всё необходимое и востребованное.

