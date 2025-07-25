
В функцию можно передавать не строку с плюсиком или минусиком, а выражение, которое ты хочешь выполнить! И тогда ты избавишься от проверки `if ... else`, и код станет проще и универсальнее.

Когда идёт разговор о том, чтобы «передать выражение в качестве аргумента», по сути аргументом передаётся не выражение, а его результат:

вместо if ... else
elif
elif, можно:

```
def some_function(arg):
    print(arg)


some_expression = 5 + 3
some_function(some_expression) 
```


В этом примере `some_expression` — это выражение, но при вызове `some_function()` аргументом передаётся не само выражение, а **вычисленный результат** этого выражения — число `8`.


Если же в аргументах нужно передать не `5 + 3`, а выражение, которое надо выполнить, например, `a + b`, и отдельно — значения для этого выражения: `a = 5`, `b = 3`, то нужно передать в функцию другую функцию.


### Передаём функцию как аргумент другой функции

Проведём эксперимент на примере операции сложения. Создадим отдельную функцию для сложения, передадим её в функцию-калькулятор. Внутри функции-калькулятора вызовем её и передадим в неё два аргумента — числа, над которыми требуется выполнить заданное действие.

В тех случаях, когда нужно **передать** функцию как объект, после названия функции скобки не ставятся. Если поставить скобки — функция будет **вызвана**.


```
# Объявляем функцию, которая принимает два числа и возвращает их сумму:
def operation_plus(a, b):
    return a + b


# Объявляем функцию-калькулятор, которая принимает функцию и два числа:
def calc(func, first, second):
    # Вызываем функцию, переданную в аргументах.
    # Передаём в неё два числа, которые тоже получены в аргументах.
    # Возвращаем результат работы полученной функции.
    return func(first, second)


# Вызываем функцию-калькулятор, передаём в неё функцию operation_plus()
# и два числа, которые надо сложить.
result = calc(operation_plus, 10, 15)  # Имя передаваемой функции - без скобок!

# Напечатаем результат:
print(result)
```


Чтобы не удлинять код, применим лямбда-функции.


### Лямбда-функции

Лямбда-функции — это безымянные, «анонимные» функции в Python. Для объявления такой функции используется ключевое слово `lambda`. Как и обычные функции, лямбда-функции могут принимать параметры.

В общем виде синтаксис объявления лямбда-функции выглядит так:

##### **!!! можно добавлять условия в формате (что-то сделать if условие else что-то сделать)**

```
lambda параметры_через_запятую: выполняемое_функцией_выражение 
```

Лямбда-функции очень похожи на обычные функции в Python. Например, обычную функцию с одним действием тоже можно объявить в одну строку:


```
def имя_функции(параметры_через_запятую): выполняемое_функцией_выражение 
```

Разница заключается в том, что:

- Обычные функции определяются с помощью ключевого слова `def`, после которого указывается имя. Лямбда-функции объявляются ключевым словом `lambda`, а имена им и вовсе не нужны.
- В лямбда-функциях можно использовать только одно выражение.
- Лямбда-функция автоматически возвращает значение того выражения, которое содержит; в лямбда-функциях не требуется ключевое слово `return`.

Выражениями в лямбда-функциях могут быть, например:

- арифметические и логические операции, такие как `a + b` и `a > b`;
- вызовы других функций, например `sum(a, b)`.

В приведённом коде определена лямбда-функция с параметрами `x` и `y`, она возвращает произведение аргументов.


```
x = lambda x, y: x * y
print(x(5, 4))

# 20 
```

Аргументы в лямбда-функцию можно передать сразу при объявлении — для этого нужно заключить функцию в скобки, а аргументы записать в той же строке, тоже в скобках:


```
x = (lambda  x, y: x * y)(2, 7)
print(x)

# Будет напечатано: 14 
```

Ту же запись можно сделать и без применения дополнительной переменной:


```
print((lambda  x, y: x * y) (2, 7))

# Будет напечатано: 14 
```


Чтобы код калькулятора не был громоздким, при вызове калькулятора мы первым аргументом передадим лямбда-функцию, которую опишем прямо при вызове функции.


```
def calc(func, first, second):
    return func(first, second)


# Первый аргумент при вызове calc() - это лямбда-функция с нужным выражением:
print(calc(lambda a, b: a + b, 5, 10))   # Складываем.
print(calc(lambda a, b: a * b, 30, 10))  # Умножаем.
print(calc(lambda a, b: a ** b, 30, 2))  # Возводим в степень. 
```


### Когда и где применять лямбда-функции

В тех случаях, когда нужна простая функция, которая не будет повторно использоваться, лучше всего подойдёт лямбда-функция.

Могут быть и другие основания для применения лямбда-функций:

- В функции выполняется понятная и простая операция, и функции не нужно название.
- Лямбда-функция делает код понятнее по сравнению с применением обычной функции.
- В коде нет именованной функции, которая выполняет необходимую операцию.


Пример применения лямбды-функции:

```
people = ['Антон', 'Соня', 'Коля', 'Женя', 'Тоня', 'Стёпа']

def say_to_all(func, sequence):
    for item in sequence:
        func(item)


# Этот вызов для каждого имени из списка должен напечатать
# строчку Привет, <имя>!
say_to_all(lambda item: print(f'Привет, {item}!'), people)
# Этот вызов для каждого имени из списка должен напечатать
# строчку До завтра, <имя>!
say_to_all(lambda item: print(f'До завтра, {item}!'), people)
```

Ещё один пример с добавление условия в лямбду-функцию:

```
people = ['Антон', 'Соня', 'Коля', 'Женя', 'Тоня', 'Стёпа']


def say_to_all(func, sequence):
    for item in sequence:
        func(item)


say_to_all(lambda item: print(f'Здравствуй, {item}!') if item[0] == 'С' else print(f'Привет, {item}!'), people)
```