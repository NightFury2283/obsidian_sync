[[Фреймворк Django. Работа с проектами]]
[[Python]]
[[API]]
[[Питон Практикум]]
## А что внутри?

В **Kittygram** описана простая модель — `Cat`. В ней есть три поля:

```python
 # kittygram/cats/models.py
class Cat(models.Model):
    name = models.CharField(max_length=16)
    color = models.CharField(max_length=16)
    birth_year = models.IntegerField(blank=True, null=True)

    def __str__(self):
        return self.name
```


В файле urls.py настроена обработка адреса `cats/`: запросы к этому URL направляются во view-функцию `cat_list()` приложения cats:

```python
# kittygram/kittygram/urls.py
from django.urls import path

from .views import cat_list

urlpatterns = [
    path('cats/', cat_list),
]
```


Файл views.py содержит единственную view-функцию `cat_list()`. Обратите внимание на декоратор `@api_view`, именно он настраивает обычную view-функцию для работы с API: например, в нём указываются типы запросов, которые должна обрабатывать view-функция. В следующем уроке мы разберёмся, что это за декоратор и чем ещё он хорош.


```python
# kittygram/cats/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

from .models import Cat
from .serializers import CatSerializer

# View-функция cat_list() будет обрабатывать только запросы GET и POST, 
# запросы других типов будут отклонены,
# так что в теле функции их можно не обрабатывать
@api_view(['GET', 'POST'])
def cat_list(request):
    if request.method == 'POST':
        serializer = CatSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    cats = Cat.objects.all()
    serializer = CatSerializer(cats, many=True)
    return Response(serializer.data)
```


Откройте в редакторе файл serializers.py, там описан сериализатор `CatSerializer`:

```python
# kittygram/cats/serializers.py

from rest_framework import serializers

from .models import Cat


class CatSerializer(serializers.ModelSerializer):
    class Meta:
        model = Cat
        fields = '__all__'
```


Сериализатор CatSerializer связывает модель и view-класс: он берёт информацию о типах данных из модели, указанной в поле `model`. Остаётся только указать в `fields` те поля, которые надо сериализовать, — и система заработает.


## Проверяем работу Kittygram

После развёртывания проект Kittygram полностью готов к работе. Запустите его и сделайте к нему запрос через Postman.

1. Запустите приложение Kittygram командой `python manage.py runserver`.
2. Запустите приложение Postman, в качестве целевого URL укажите `http://127.0.0.1:8000/cats/`.
3. Отправьте GET-запрос.

В ответ вернётся пустой JSON-объект. Это нормально, ведь в базе данных пока пусто:


![[Pasted image 20250721111644.png]]


## Первый котик в Kittygram

Чтобы создать в БД первую запись с котиком, выполните POST-запрос к проекту и передайте поля `name`, `color` и `birth_year`:

Обратите внимание, что поля нужно передавать в теле (англ. _body_) POST-запроса. А в Postman в соответствующем разделе должны быть выбраны опции **raw** и **JSON**.


![[Pasted image 20250721111844.png]]


Проверьте ответ: если запрос обработан корректно, должен вернуться статус-код 201 Created и JSON с только что созданным объектом.

Получилось? Отлично!

Сделайте ещё один POST-запрос, создайте запись для второго котика.

Повторно выполните GET-запрос на тот же URL, в ответ вам вернётся список из существующих в базе данных записей.


## Время делать ошибки: 400 Bad Request

Проверьте, что будет, если передать данные в неправильном формате: например, вместо целочисленного значения в поле `birth_year` передайте данные в формате `месяц.год`.

![[Pasted image 20250721112017.png]]

Обратите внимание на статус-код ответа: **400 Bad Request**. Поле `birth_year` ожидает данные в формате _Integer_ (целое число), а полученные в запросе данные можно привести только к float. В результате запрос не прошёл валидацию и запись не была создана.

Измените запрос: укажите год в формате _Integer_ и повторите попытку. На этот раз всё должно пройти гладко.

## Ошибка 405 Method Not Allowed

В декораторе view-функции `cat_list()` указано, что она должна обрабатывать запросы GET или POST: можно только создавать и запрашивать данные.

При попытке удалить запись методом DELETE вернётся сообщение об ошибке: **405 Method Not Allowed**.

```json
{
    "detail": "Method \"DELETE\" not allowed."
}
```


## Настройка полей в ответе

При запросе к API пользователю не понадобится id котика. Хорошо бы убрать это поле из JSON-ответа.

Список полей, передаваемых в ответ, описывается в сериализаторе, в поле `fields`. Вместо записи `fields = '__all__'` явно перечислите обрабатываемые поля.

Сделайте GET-запрос на эндпоинт `http://127.0.0.1:8000/cats/` и проверьте, что всё работает и ответ не изменился.

Уберите `id` из списка полей, и ваш API перестанет отправлять ненужные данные клиенту.

```python
# Обновлённый kittygram/cats/serializers.py
from .models import Cat
from rest_framework import serializers


class CatSerializer(serializers.ModelSerializer):
    class Meta:
        model = Cat
        fields = ('name', 'color', 'birth_year')
```

```python
...
{
    "name": "Леопольд",
    "color": "Рыжий",
    "birth_year": 1975
}
...
```


