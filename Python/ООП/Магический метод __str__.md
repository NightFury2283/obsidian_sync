
Теперь попробуйте вывести на печать объект класса `Phone`. Запустите код:

```
class Phone:

    line_type = 'проводной'

    def __init__(self, dial_type_value):
        self.dial_type = dial_type_value

    def ring(self):
        print('Дзззззыыыыыыыынь!')

    def call(self, phone_number):
        print(f'Звоню по номеру {phone_number}! Тип связи - {self.line_type}.')

    def dial_type_upgrade(self, new_dial_type):
        self.dial_type = new_dial_type


rotary_phone = Phone(dial_type_value='дисковый')

print(rotary_phone)
```

```
Результат

<__main__.Phone object at 0x7fa57b33ffd0>
```


Получилось не так красиво, как со строкой: Python вывел на экран имя класса и адрес в памяти, где сохранён объект. Так работает метод `__str__`, который по умолчанию есть у всех объектов — встроенных и пользовательских. Этот метод позволяет объектам «рассказывать» о себе в виде строк.


Вывод по умолчанию получается не самым понятным. Для своих классов вы можете переопределить этот метод, то есть указать, что конкретно объект должен «рассказать» о себе.

Например:

```
# Вот он - магический метод __str__ с пользовательским описанием.
    def __str__(self):
        return f'Это {self.line_type} телефон. Набор - {self.dial_type}.'
```

```
Результат

Это проводной телефон. Набор - дисковый.
```


Шпаргалка:


https://code.s3.yandex.net/Python-dev/cheatsheets/089-oop-osnovnye-printsipy-shpora/089-oop-osnovnye-printsipy-shpora.html





