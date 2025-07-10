
![[Pasted image 20250619183251.png]]

- **`gso`** – настройки входа через Google (запрос токена и email).
    
- **`googleSignInClient`** – клиент для запуска процесса авторизации.

### **`rememberNavController()` и `remember` в Jetpack Compose**


## **`rememberNavController()`**

Это создание и сохранение объекта `NavController` для управления навигацией между экранами.

- **`NavController`** – "мозг" навигации. Он:
    
    - Хранит стек экранов (историю переходов).
        
    - Обрабатывает команды (`navigate()`, `popBackStack()`).

- **`rememberNavController()`** – хелпер-функция, которая создаёт `NavController` и сохраняет его при перекомпозиции.

## **`remember` — ключевой механизм Compose**

**Что это?**  
Функция для сохранения значений между перекомпозициями (перерисовками UI).

**Как работает?**

var count by remember { mutableStateOf(0) }

- При первом вызове `remember` вычисляет значение (например, `mutableStateOf(0)`).
    
- При последующих перекомпозициях возвращает **то же самое значение**, а не вычисляет заново.


### **`rememberLauncherForActivityResult`**

**Что это?**  
Механизм для запуска активности (например, Google Sign-In) и получения результата в Compose.


-**`ActivityResultContracts.StartActivityForResult()`** – стандартный контракт(правила взаимодействия) для запуска активности.


### **`CoroutineScope(Dispatchers.IO).launch`**

**Что это?**  
Запуск корутины в фоновом потоке для операций с БД/сетью.


### **`LaunchedEffect`**
![[Pasted image 20250619185627.png]]

**Что это?**  
Side-effect в Compose для запуска корутин при изменении ключей.

- **Когда вызывается**: При первом составлении (composition) и при изменении `key1`.

#### **`popUpTo`**

![[Pasted image 20250619185725.png]]

Удаляет экраны из стека навигации:



