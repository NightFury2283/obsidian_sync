
[[Питон Практикум]]
[[Фреймворк Django. Работа с проектами]]

## Объединение условий

Объединить несколько условий в методе `.filter()` можно через запятую.

Чтобы view-функция получила объекты, которые соответствуют сразу двум условиям, её код должен быть примерно таким:


```python
# homepage/views.py
from django.shortcuts import render

from ice_cream.models import IceCream

def index(request):
    template = 'homepage/index.html'
    ice_cream_list = IceCream.objects.values(
        'id', 'title', 'description'
    ).filter(
        is_published=True, is_on_main=True  # Два в одном!
    )
    context = {
        'ice_cream_list': ice_cream_list,
    }
    return render(request, template, context)
```


## Q-объекты: запросы с операторами NOT, AND и OR

В запросах может быть недостаточно перечисления условий через запятую: иногда требуется составить более сложный комбинированный запрос. В Django ORM для этого применяют **Q-объекты**.

В **Q-объект** передаётся название поля, модификатор и значение для фильтрации, а сами объекты объединяются в запрос логическими операторами: `~` (NOT), `&` (AND) и `|` (OR):


![[Pasted image 20250511103626.png]]


Вот пример view-функции, которая выбирает объекты по двум условиям:

```python
# homepage/views.py
# Для применения Q-объектов их нужно импортировать:
from django.db.models import Q
from django.shortcuts import render

from ice_cream.models import IceCream

def index(request):
    template_name = 'homepage/index.html'
    ice_cream_list = IceCream.objects.values(
        'id', 'title', 'description'
    ).filter(
        # Делаем запрос, объединяя два условия
        # через Q-объекты и оператор AND:
        Q(is_published=True) & Q(is_on_main=True)
    )
    context = {
        'ice_cream_list': ice_cream_list,
    }
    return render(request, template_name, context)
```


## Примеры запросов с логическими операторами

**Логический оператор AND**

```python
# Вариант 1, через запятую в аргументах метода .filter():
IceCream.objects
.values('id')
.filter(is_published=True, is_on_main=True)

# Вариант 2, через Q-объекты:
IceCream.objects
.values('id')
.filter(Q(is_published=True) & Q(is_on_main=True))

# Вариант 3, дважды вызываем метод .filter();
# так обычно не пишут, но этот вариант тоже встречается:
IceCream.objects
.values('id')
.filter(is_published=True).filter(is_on_main=True)
```


**Логический оператор OR**

```python
# Можно так, через Q-объекты:
IceCream.objects
.values('id')
.filter(Q(is_published=True) | Q(is_on_main=True))

# А можно и так - более многословно, но зато без Q-объектов:
IceCream.objects.values('id').filter(is_published=True) 
| IceCream.objects.values('id').filter(is_on_main=True)
```

**Логический оператор NOT**

```python
# Лучше так:
IceCream.objects
.values('id')
.filter(Q(is_published=True) & ~Q(is_on_main=False))

# Но сработает и так:
IceCream.objects
.values('id')
.filter(is_published=True)
.exclude(is_on_main=False)
```

## Приоритет выполнения

Логические операторы имеют разный приоритет выполнения: оператор `NOT` имеет самый высокий приоритет и выполняется первым, следующий по приоритету — оператор `AND`, а самый последний — оператор `OR`. Для объединения условий в группу используются скобки `()`.


