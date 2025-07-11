[[Python]]
[[Фреймворк Django. Работа с проектами]]
### View-функция: обработчик запроса

View-функция — это обычная функция с некоторыми особенностями. Задача такой функции — принять на вход информацию о запросе и вернуть ответ пользователю.

![[Планирование адресов и конвертеры путей#^bd463e]]


Первым аргументом view-функция должна принимать объект запроса — **HttpRequest.** В этом объекте хранится информация из полученного запроса.

А возвращать view-функция должна объект ответа — **HttpResponse.** Первым аргументом в этот объект передаётся строка — текстовое содержимое, которое будет отображено в браузере пользователя в ответ на запрос.


```python
...

# Первый аргумент view-функции принимает экземпляр класса HttpRequest.
def product_list(request):
    # Здесь может быть любой код, который выполняется для подготовки ответа.
    ...
    # Создаём экземпляр класса HttpResponse с текстом,
    # который будет передан в браузер.
    return HttpResponse('Страница со списком товаров')
```



View-функции хранят отдельно, в файле _views.py_. Если проект разделён на приложения, в каждом приложении создают собственный файл _views.py_, по той же логике, что и файлы _urls.py_.


## HttpRequest: **о**бъект запроса

При получении запроса Django создаёт экземпляр объекта **HttpRequest**; этот объект содержит все данные о запросе: запрошенный URL, тип запроса и многое другое.

Самые востребованные атрибуты объекта **HttpRequest**:

- `HttpRequest.path` — относительный адрес запроса (без доменного имени). Например, при запросе `http://www.acme.not/catalog/` в свойстве `HttpRequest.path` будет содержаться строка `'catalog/'`.
- `HttpRequest.method` — метод полученного запроса. В зависимости от типа запроса одна и та же view-функция может вести себя по-разному. Например, при GET-запросе — отдавать запрошенную информацию, а при POST-запросе — записывать что-то в базу данных.

Детально познакомиться с объектом **HttpRequest** можно [в документации Django](https://docs.djangoproject.com/en/3.2/ref/request-response/#django.http.HttpRequest).


## HttpResponse: объект ответа

View-функции должны возвращать объект **HttpResponse;** этот объект содержит статус-код ответа сервера, код возвращаемой HTML-страницы и другую информацию, необходимую браузеру для отрисовки страницы. Разработчик может управлять содержимым ответа, создавая объект HttpResponse с теми или иными параметрами.

Детально познакомиться с объектом **HttpResponse** можно [в документации Django](https://docs.djangoproject.com/en/3.2/ref/request-response/#django.http.HttpResponse).


### Пример работы алгоритма открытия страницы:

## От запроса до ответа

Отследим ход обработки запроса к главной странице `www.acme.not`. Получив запрос, Django первым делом смотрит в корневой _urls.py_ и находит там маршрут для главной страницы. К этому маршруту подключены адреса из приложения **homepage**:


```
# acme_project/urls.py
from django.urls import include, path

urlpatterns = [    
    path('', include('homepage.urls')),
    ...
] 
```

Запрос передаётся в _urls.py_ приложения **homepage**:


```python
# homepage/urls.py
from django.urls import path

from . import views

urlpatterns = [
    # Главная страница.
    path('', views.index),
] 
```

Вызывается view-функция `index()`, возвращающая объект класса **HttpResponse** с текстом `'Главная страница'`:


```python
# homepage/views.py
# Класс HttpResponse нужно импортировать в код из модуля django.http.
from django.http import HttpResponse

def index(request):    
    return HttpResponse('Главная страница') 
```

Браузер преобразует этот объект в страницу:


![[Pasted image 20250424232310.png]]


### Переменные из URL

Если в шаблоне адреса использована переменная — то view-функция должна принимать эту переменную.

Допустим в файле _catalog/urls.py_ для обработки запросов вида `http://acme.not/catalog/kangaroo/` подготовлен такой шаблон адреса:


```python
# catalog/urls.py
from django.urls import path

from . import views

urlpatterns = [
    path('<slug:category>/', views.product_category),
    ...
] 
```

В шаблоне адреса используется переменная `category`; эту переменную должна принимать на вход view-функция `product_category()`, которая вызывается для этого шаблона:


```python
# catalog/views.py
from django.http import HttpResponse

def product_category(request, category):    
    return HttpResponse(f'Категория {category}') 
```

Если в функции `path()` в шаблоне адреса вы применяете переменную (как в примере: `path('<slug:category>/', views.product_category)`), то view-функция должна **обязательно** ожидать на вход эту переменную, даже если вы не планируете использовать её в функции.
