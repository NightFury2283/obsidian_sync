
Страницы, которые мы до сих пор создавали в проекте, — динамические, их содержимое зависит от состояния базы данных: если база пуста — страницы пусты; база наполнена — страницы наполнены; объект в БД изменился — содержимое страницы тоже изменилось.

Но помимо страниц с динамическим содержимым на сайте могут понадобиться и простые статичные страницы:

- контактная информация,
- правила и помощь,
- юридическая информация

...и множество других.

В Django есть view-класс **TemplateView**, который хорошо подходит для таких случаев. Этот класс по умолчанию обрабатывает только GET-запросы и возвращает страницу, сформированную на основе шаблона (он указывается в `template_name` — это обязательное свойство класса) и словаря контекста (его может и не быть).

В проекте **birthday** есть одна статичная страница — это главная страница проекта. Она описана в приложении **pages**; запросы к главной странице обрабатываются view-функцией, которая вызывает шаблон _pages/index.html_; в шаблон главной страницы даже не передаётся словарь контекста — ведь содержимое страницы встроено прямо в шаблон.

Это последняя view-функция, оставшаяся в проекте; заменим её на CBV **TemplateView**.

В файле _pages/views.py_ вместо view-функции `homepage()` опишем класс `Homepage`, унаследованный от **TemplateView**.

Единственный обязательный атрибут, который надо указать в классе, — это имя шаблона:

```python
# pages/views.py

# Импортируем класс TemplateView, чтобы унаследоваться от него.
from django.views.generic import TemplateView


class HomePage(TemplateView):
    # В атрибуте template_name обязательно указывается имя шаблона,
    # на основе которого будет создана возвращаемая страница.
    template_name = 'pages/index.html'
```


Теперь нужно внести изменения в описание маршрута: вместо вызова функции указываем метод `as_view()` класса `Homepage`.

```python
# pages/urls.py
from django.urls import path

from . import views

app_name = 'pages'

urlpatterns = [
    path('', views.HomePage.as_view(), name='homepage'),
]
```

Готово! Откройте главную страницу приложения, всё должно работать как раньше.

## Работа со словарём контекста в TemplateView

Из класса TemplateView можно передавать в шаблон любые переменные через словарь контекста. Отправим на главную страницу проекта ACME информацию о количестве дней рождения, опубликованных в приложении **birthday**. Для этого добавим в словарь контекста ключ `total_count`, в котором будет храниться количество объектов модели `Birthday`.

Добавление нового ключа выполняется через переопределение метода `get_context_data()`:

```python
# pages/views.py

from django.views.generic import TemplateView

from birthday.models import Birthday


class HomePage(TemplateView):
    template_name = 'pages/index.html'

    def get_context_data(self, **kwargs):
        # Получаем словарь контекста из родительского метода.
        context = super().get_context_data(**kwargs)
        # Добавляем в словарь ключ total_count;
        # значение ключа — число объектов модели Birthday.
        context['total_count'] = Birthday.objects.count()
        # Возвращаем изменённый словарь контекста.
        return context
```


Теперь надо вывести значение `total_count` в шаблон:

```html
<!-- pages/index.html -->
{% extends "base.html" %}

{% block content %}
  <h1>Проект ACME</h1>
  <p>
    Acme Corporation: <b>A</b> <b>C</b>ompany <b>M</b>ade <b>E</b>verything
  </p>
  <!-- Добавим строку "row" -->
  <div class="row">
    <!-- Добавим колонку "col-3" -->
    <div class="col-3">
      Количество записей о днях рождения<br />в нашем проекте:
    </div>
    <div class="col-9">
      <!-- Добавим колонку "col-9" и выведем значение переменной total_count.
           С помощью стиля Bootstrap отобразим его очень крупным шрифтом  -->
      <div class="display-1">{{ total_count }}</div>
    </div>
  </div>
{% endblock %}
```


![[Pasted image 20250527213430.png]]


С остальными особенностями этого класса можно познакомиться [в официальной документации](https://docs.djangoproject.com/en/3.1/ref/class-based-views/base/#django.views.generic.base.TemplateView).

А вот и [шпаргалка по теме «Представления: расширенные возможности»](https://code.s3.yandex.net/Python-dev/cheatsheets/033-django-class-based-views-shpora/033-django-class-based-views-shpora.html). Сохраните её в закладки, с ней будет проще двигаться дальше.
