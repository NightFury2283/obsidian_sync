
[[Питон Практикум]]
[[Фреймворк Django. Работа с проектами]]
[[HTML_CSS - фронтенд]]

Пока что форма в проекте выглядит не очень презентабельно. Дизайн — это отдельная специальность, но презентовать пользователям свой проект лучше в аккуратном виде. Привести в порядок внешний вид страницы можно без больших трудозатрат, подключив к проекту фреймворк Bootstrap; на этот раз подключим его с помощью специальной библиотеки.

Фреймворк Bootstrap содержит [набор стилей для форм](https://getbootstrap.com/docs/5.3/forms/overview/#overview); применим их, чтобы страница выглядела прилично.


![[Pasted image 20250513120321.png]]


Чтобы не мучиться и не писать HTML-код отдельно для каждого элемента формы, воспользуемся готовым решением — библиотекой **django-bootstrap5**, предназначенной для интеграции Bootstrap в Django-проекты.

В активированном виртуальном окружении установите библиотеку:

```bash
pip install django-bootstrap5==22.2
```


В настройках проекта пропишите в список установленных приложений `INSTALLED_APPS` библиотеку, неважно где — вверху, внизу или посередине списка.

```python
INSTALLED_APPS = [
    ...
    'django_bootstrap5',
    ...
]
```


Теперь нужно подключить библиотеку django_bootstrap5 к тем HTML-шаблонам, где она будет использоваться; для этого в начало шаблона добавляется тег `{% load django_bootstrap5 %}`.

У библиотеки есть ряд собственных тегов для шаблонов Django. Для оформления формы требуется только два специальных тега:

- `{% bootstrap_form form %}` — для вывода полей формы с Bootstrap-оформлением, которое при необходимости можно модифицировать. В этом теге переменная `form` указывается без методов `as_ul` или `as_p`.
- `{% bootstrap_button button_type="submit" content="Oтправить" %}` — тег для отрисовки кнопки.

В базовом шаблоне необходимо подключить CSS-стили Bootstrap, для этого в начало шаблона _base.html_ добавляется тег `{% bootstrap_css %}`.

Подробное описание библиотеки django-bootstrap5 есть [в документации](https://django-bootstrap5.readthedocs.io/en/latest/index.html).


## Сделаем красиво

В _base.html_ замените тег `<link>` на тег библиотеки django_bootstrap5. Код должен получиться таким:

```html
<!DOCTYPE html>
<html lang="ru">
  <head>
    <meta charset="UTF-8">
    <title>Проект ACME</title>
    <!-- Подключаем библиотеку django_bootstrap5.
      Это нужно сделать во всех шаблонах,
      где будут применяться теги этой библиотеки. -->
    {% load django_bootstrap5 %}
    <!-- Загружаем файл с CSS-стилями по умолчанию.
      Это делается только в базовом шаблоне. -->
    {% bootstrap_css %}
  </head>
  <body>
    {% include "includes/header.html" %}
    <div class="container pt-5">
      {% block content %}{% endblock %}
    </div>
  </body>
</html>
```


Откройте страницу с формой и убедитесь, что ничего не сломалось: страница должна открываться точно в том же виде, что и раньше.

Теперь можно добавить в код шаблона _birthday.html_ теги библиотеки django_bootstrap5:


```html
<!-- templates/birthday/birthday.html -->
{% extends "base.html" %}
<!-- Подключаем библиотеку django_bootstrap5. -->
{% load django_bootstrap5 %}

{% block content %}
  <form>
    <!-- Выводим поля формы с помощью специального тега. -->
    {% bootstrap_form form %}
    <!-- Добавляем кнопку отправки данных. -->
    {% bootstrap_button button_type="submit" content="Отправить" %}
  </form>
  {% with data=request.GET %}
    ...
```


![[Pasted image 20250514094250.png]]


Добавим форме границу и заголовок. В Bootstrap есть [блок под названием «карточка»](https://getbootstrap.com/docs/5.2/components/card/) (_card_), он отвечает нашим требованиям. Его и применим.

Измените код в шаблоне _birthday.html_:


```html
<!-- templates/birthday/birthday.html -->
...
{% block content %}
  <div class="card">
    <div class="card-header">
      Калькулятор ожидания дня рождения
    </div>
    <div class="card-body">
      <form>
        {% bootstrap_form form %}
        {% bootstrap_button button_type="submit" content="Отправить" %}
      </form>
    </div>
  </div>
  {% with data=request.GET %}
...
```


Вокруг формы появилась внешняя граница, а над формой — название:

![[Pasted image 20250514095208.png]]


Некрасиво, если форма занимает всю ширину страницы; для управления шириной блоков в Bootstrap есть специальный набор классов.

[Фреймворк Bootstrap виртуально делит ширину любого блока на 12 колонок](https://getbootstrap.com/docs/5.2/layout/columns/#column-wrapping). Чтобы **вложенный** блок занимал половину родительского — вложенному блоку нужно присвоить класс `col-6` («блок занимает шесть колонок из двенадцати»).

Добавим блоку с классом `card` класс `col-4`:

```html
...
{% block content %}
  <div class="card col-4">
    <div class="card-header">
      Калькулятор ожидания дня рождения
    </div>
...
```


![[Pasted image 20250514095433.png]]


