[[Питон Практикум]]

Правила для Яндекс практикума

### HTML-шаблоны

1. Отступ перед вложенными элементами — два пробела:

```html
 <ul>
   <li>элемент списка</li>
 </ul>
```


2. Одиночные теги, например `<br>`, не следует закрывать (не надо так: `<br/>`).
3. Названия атрибутов пишутся в двойных кавычках:
    
    `class="attr-not-single-quotes"`.
    
4. Имена классов и идентификаторов пишутся строчными буквами, слова в них разделяются дефисом `-`. Например: `my-class-not-underscore`.
5. Если тег с атрибутами длиннее 79 символов, нужен перенос строки. Новая строка отбивается двумя пробелами от начала строки родительского тега.


```html
 <div
   id="my-id"
   class="my-class-long-name"
   >
   <p>
     Lorem Ipsum Dolor Sit....
   </p>
 </div>
```


Если необходимо избежать лишнего пробела в текстовой части, возможно и такое написание:

```html
 <a
   id="my-id"
   class="my-class-long-name"
   >Lorem Ipsum Dolor Sit...</a>
```


### Django-шаблоны

Код шаблонов оформляется согласно [Coding style](https://docs.djangoproject.com/en/dev/internals/contributing/writing-code/coding-style/#template-style).

1. Вокруг переменных ставятся одиночные пробелы:

```html
 {{ foo }}
```

2. Не ставятся пробелы между переменной и фильтрами:

```html
 {{ name|lower }}
```

3. Теги Django по значимости идентичны HTML-тегам. Отступ перед вложенными элементами — два пробела:

```html
 {% extends "base.html" %}
 {% load static %}
 {% block css %}
 {% endblock %}
 
 {% block content %}
   <div> <!-- Может быть разный уровень вложенности -->
     {% for post in object_list %}    
       <div class="single-post"> 
         <p>Текст поста</p>
       </div>
     {% endfor %}
   </div>
 {% endblock %}
```


