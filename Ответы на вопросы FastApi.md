# Pydantic

## 1. Что такое Pydantic и для чего он используется?
Pydantic — библиотека для валидации данных и работы с моделями в Python. Используется для парсинга, сериализации и проверки данных (например, JSON, конфигураций, запросов/ответов API). Основан на аннотациях типов.

## 2. Как создать базовую модель данных с использованием Pydantic?
Используй класс `BaseModel`. Пример:

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    email: str
```

Теперь можно создавать объекты `User` с автоматической проверкой типов.

## 3. Как Pydantic осуществляет валидацию данных?
Pydantic автоматически проверяет типы данных на основе аннотаций. Если данные не соответствуют типам, выбрасывается исключение `ValidationError`. Можно добавлять кастомные валидаторы через декоратор `@validator`.
## 4. Чем отличается BaseModel от dataclass в Pydantic?
Вот таблица с основными различиями между `BaseModel`, `Dataclass` и гибридным подходом `Pydantic + Dataclass`:

| **Характеристика**            | **BaseModel**                          | **Dataclass**                     | **Pydantic + Dataclass**          |
|-------------------------------|----------------------------------------|-----------------------------------|-----------------------------------|
| **Основная цель**             | Валидация, парсинг, сериализация       | Упрощение создания классов данных | Упрощение + валидация             |
| **Валидация данных**          | ✅ Встроенная                          | ❌ Нет (если не использовать Pydantic) | ✅ Встроенная через Pydantic       |
| **Сериализация (`.dict()`, `.json()`)** | ✅ Встроенные методы                | ❌ Нет                             | ❌ Нет (только через Pydantic)     |
| **JSON Schema**               | ✅ Автоматическая генерация            | ❌ Нет                             | ❌ Нет (только через Pydantic)     |
| **Кастомные валидаторы**      | ✅ Через `@validator`                  | ❌ Нет                             | ✅ Через `@validator`              |
| **Вложенные модели**          | ✅ Поддержка                           | ❌ Нет                             | ✅ Поддержка                       |
| **Синтаксис**                 | Классический (`class User(BaseModel)`) | Упрощенный (`@dataclass`)         | Упрощенный (`@dataclass`)         |
| **Производительность**        | Выше накладные расходы                 | Минимальные накладные расходы     | Средние накладные расходы         |
| **Использование**             | API, конфигурации, сложные модели      | Простые контейнеры данных         | Гибридный подход: простота + валидация |

- **BaseModel**: Для сложных сценариев с валидацией, сериализацией и JSON Schema.
- **Dataclass**: Для простых контейнеров данных без валидации.
- **Pydantic + Dataclass**: Если нужен синтаксис `dataclass`, но с валидацией Pydantic.

Эта таблица поможет выбрать подходящий инструмент в зависимости от задачи.



## 5. Как задать типы данных для полей в Pydantic модели?
В Pydantic типы полей задаются через **аннотации типов**. Пример:

```python
from pydantic import BaseModel, EmailStr
from typing import List, Optional

class User(BaseModel):
    id: int
    name: str
    email: EmailStr
    age: Optional[int] = None
    friends: List[str] = []
```

**Ключевые моменты:**
- Базовые типы: `int`, `str`, `bool`.
- Сложные типы: `List`, `Dict`, `Optional`.
- Вложенные модели.
- Специальные типы: `EmailStr`, `PositiveInt`.
- Кастомные типы: `constr(min_length=3)`.
- `Optional` для необязательных полей.

## 6. Как использовать тип constr в Pydantic для ограничения длины строки?

Чтобы ограничить длину строки в Pydantic, используй тип `constr` (constrained string). Пример:

```python
from pydantic import BaseModel, constr

class User(BaseModel):
    username: constr(min_length=3, max_length=50)  # Строка от 3 до 50 символов
```

### Как это работает:
- `min_length`: Минимальная длина строки.
- `max_length`: Максимальная длина строки.

### Пример использования:
```python
user = User(username="abc")  # Валидно
user = User(username="ab")   # Ошибка: меньше 3 символов
user = User(username="a" * 51)  # Ошибка: больше 50 символов
```


## 7. Как можно задать значения по умолчанию для полей модели Pydantic?
Чтобы задать значения по умолчанию для полей модели Pydantic, просто присвой значение прямо в аннотации. Пример:

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str = "Anonymous"  # Значение по умолчанию
    age: int = 18            # Значение по умолчанию
    is_active: bool = True   # Значение по умолчанию
```

### Как это работает:
- Если поле не передано при создании объекта, используется значение по умолчанию.
- Если передано — используется переданное значение.

### Пример:
```python
user1 = User()  # Используются значения по умолчанию
print(user1)    # name='Anonymous' age=18 is_active=True

user2 = User(name="John", age=30)  # Переопределены name и age
print(user2)    # name='John' age=30 is_active=True
```


## 8. Как определить кастомный валидатор для поля модели в Pydantic?
Чтобы создать кастомный валидатор для поля модели в Pydantic, используй декоратор `@validator`. Пример:

```python
from pydantic import BaseModel, validator

class User(BaseModel):
    name: str
    age: int

    @validator('age')
    def check_age(cls, age):
        if age < 0:
            raise ValueError('Age must be positive')
        return age
```

### Как это работает:
1. Декоратор `@validator('age')` указывает, что метод `check_age` валидирует поле `age`.
2. Валидатор принимает значение поля и возвращает его (или измененное значение).
3. Если данные невалидны, выбрасывается `ValueError`.

### Пример использования:
```python
user = User(name="John", age=-5)  # Ошибка: Age must be positive
user = User(name="John", age=25)  # Валидно
```


## 9. В чем разница между @validator, @root_validator, @before, @after, @wrap, и @plain валидаторами?
В Pydantic есть несколько типов валидаторов, каждый из которых используется для разных сценариев. Вот их основные различия:

---

### 1. **`@validator`**
- **Назначение**: Валидация одного конкретного поля.
- **Когда использовать**: Когда нужно проверить или изменить значение одного поля.
- **Пример**:
  ```python
  from pydantic import BaseModel, validator

  class User(BaseModel):
      name: str

      @validator('name')
      def name_must_contain_space(cls, name):
          if ' ' not in name:
              raise ValueError('Name must contain a space')
          return name
  ```

---

### 2. **`@root_validator`**
- **Назначение**: Валидация всей модели или нескольких полей одновременно.
- **Когда использовать**: Когда нужно проверить или изменить несколько полей вместе.
- **Параметры**:
  - `pre=True`: Выполняется до стандартной валидации.
  - `pre=False` (по умолчанию): Выполняется после стандартной валидации.
- **Пример**:
  ```python
  from pydantic import BaseModel, root_validator

  class User(BaseModel):
      name: str
      age: int

      @root_validator
      def check_name_and_age(cls, values):
          if values['age'] < 18 and 'admin' not in values['name']:
              raise ValueError('Admins must be at least 18 years old')
          return values
  ```

---

### 3. **`@validator` с `pre=True`**
- **Назначение**: Валидация поля **до** стандартной валидации.
- **Когда использовать**: Когда нужно изменить или проверить значение поля до того, как Pydantic применит стандартные правила.
- **Пример**:
  ```python
  from pydantic import BaseModel, validator

  class User(BaseModel):
      age: int

      @validator('age', pre=True)
      def convert_age_to_int(cls, age):
          if isinstance(age, str):
              return int(age)
          return age
  ```

---

### 4. **`@validator` с `always=True`**
- **Назначение**: Валидация поля **всегда**, даже если значение не передано.
- **Когда использовать**: Когда нужно выполнить валидацию, даже если поле имеет значение по умолчанию.
- **Пример**:
  ```python
  from pydantic import BaseModel, validator

  class User(BaseModel):
      name: str = "Anonymous"

      @validator('name', always=True)
      def check_name(cls, name):
          if name == "Anonymous":
              raise ValueError('Name cannot be "Anonymous"')
          return name
  ```

---

### 5. **`@before_validator`**
- **Назначение**: Валидация поля **до** стандартной валидации (аналог `pre=True`).
- **Когда использовать**: То же, что и `@validator(pre=True)`.
- **Пример**:
  ```python
  from pydantic import BaseModel, before_validator

  class User(BaseModel):
      age: int

      @before_validator('age')
      def convert_age_to_int(cls, age):
          if isinstance(age, str):
              return int(age)
          return age
  ```

---

### 6. **`@after_validator`**
- **Назначение**: Валидация поля **после** стандартной валидации.
- **Когда использовать**: Когда нужно выполнить дополнительные проверки после стандартной валидации.
- **Пример**:
  ```python
  from pydantic import BaseModel, after_validator

  class User(BaseModel):
      age: int

      @after_validator('age')
      def check_age_range(cls, age):
          if age < 0:
              raise ValueError('Age must be positive')
          return age
  ```

---

### 7. **`@wrap_validator`**
- **Назначение**: Создание кастомного валидатора с полным контролем над процессом.
- **Когда использовать**: Когда нужно реализовать сложную логику валидации.
- **Пример**:
  ```python
  from pydantic import BaseModel, wrap_validator

  class User(BaseModel):
      age: int

      @wrap_validator('age')
      def custom_age_validator(cls, value, handler):
          if value < 0:
              raise ValueError('Age must be positive')
          return handler(value)
  ```

---

### 8. **`@plain_validator`**
- **Назначение**: Валидация без привязки к модели (используется редко).
- **Когда использовать**: Когда нужно создать валидатор, который не зависит от модели.
- **Пример**:
  ```python
  from pydantic import plain_validator

  @plain_validator
  def validate_age(age):
      if age < 0:
          raise ValueError('Age must be positive')
      return age
  ```

---

### Итоговая таблица:

| **Валидатор**         | **Назначение**                                                                 | **Когда использовать**                                                                 |
|------------------------|-------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| `@validator`           | Валидация одного поля.                                                        | Проверка или изменение значения одного поля.                                          |
| `@root_validator`      | Валидация всей модели или нескольких полей.                                   | Проверка или изменение нескольких полей вместе.                                       |
| `@validator(pre=True)` | Валидация поля до стандартной валидации.                                      | Изменение или проверка значения до стандартной валидации.                             |
| `@validator(always=True)` | Валидация поля всегда, даже если значение не передано.                    | Проверка поля, даже если оно имеет значение по умолчанию.                             |
| `@before_validator`    | Валидация поля до стандартной валидации (аналог `pre=True`).                  | То же, что и `@validator(pre=True)`.                                                  |
| `@after_validator`     | Валидация поля после стандартной валидации.                                   | Дополнительные проверки после стандартной валидации.                                  |
| `@wrap_validator`      | Создание кастомного валидатора с полным контролем.                            | Сложная логика валидации.                                                             |
| `@plain_validator`     | Валидация без привязки к модели.                                              | Редкие случаи, когда валидатор не зависит от модели.                                  |

Теперь ты знаешь, как и когда использовать каждый тип валидатора! 😊

## 10. Когда следует использовать @before валидатор и какие задачи он решает?
`@before` валидатор (или `@validator` с параметром `pre=True`) используется для выполнения валидации **до** стандартной проверки типов и правил Pydantic. Он решает задачи, связанные с предварительной обработкой данных, и может быть полезен в следующих сценариях:

---

### **Когда использовать `@before` валидатор?**
1. **Преобразование данных**:
   - Если нужно изменить формат данных перед стандартной валидацией (например, преобразовать строку в число).

2. **Предварительная очистка данных**:
   - Если нужно удалить лишние пробелы, привести строку к нижнему регистру или выполнить другие манипуляции.

3. **Ранняя проверка данных**:
   - Если нужно проверить данные до того, как Pydantic применит стандартные правила (например, проверить, что строка не пустая).

4. **Логика, зависящая от сырых данных**:
   - Если валидация зависит от исходного формата данных, который может измениться после стандартной проверки.

---

### **Примеры задач, которые решает `@before` валидатор**

#### 1. **Преобразование строки в число**
```python
from pydantic import BaseModel, validator

class User(BaseModel):
    age: int

    @validator('age', pre=True)
    def convert_age_to_int(cls, age):
        if isinstance(age, str):
            return int(age)  # Преобразуем строку в число
        return age
```

#### 2. **Очистка строки**
```python
from pydantic import BaseModel, validator

class User(BaseModel):
    name: str

    @validator('name', pre=True)
    def clean_name(cls, name):
        return name.strip().title()  # Удаляем пробелы и приводим к заглавным буквам
```

#### 3. **Ранняя проверка данных**
```python
from pydantic import BaseModel, validator

class User(BaseModel):
    email: str

    @validator('email', pre=True)
    def check_email_not_empty(cls, email):
        if not email:
            raise ValueError('Email cannot be empty')
        return email
```

#### 4. **Логика, зависящая от сырых данных**
```python
from pydantic import BaseModel, validator

class User(BaseModel):
    age: int

    @validator('age', pre=True)
    def check_age_format(cls, age):
        if isinstance(age, str) and not age.isdigit():
            raise ValueError('Age must be a number')
        return age
```

---

### **Как работает `@before` валидатор?**
1. **Порядок выполнения**:
   - `@before` валидатор выполняется **до** стандартной валидации Pydantic.
   - После него применяются стандартные правила (например, проверка типов).

2. **Возвращаемое значение**:
   - Валидатор должен вернуть значение, которое будет передано в стандартную валидацию.

3. **Ошибки**:
   - Если валидатор выбрасывает исключение, стандартная валидация не выполняется.

---

### **Пример использования**
```python
class User(BaseModel):
    age: int

    @validator('age', pre=True)
    def convert_age_to_int(cls, age):
        if isinstance(age, str):
            return int(age)
        return age

user = User(age="25")  # Валидно: строка "25" преобразована в число 25
print(user.age)  # 25
```

---

### **Итог**
Используй `@before` валидатор (`@validator` с `pre=True`), когда нужно:
- Преобразовать данные перед стандартной валидацией.
- Очистить или предварительно проверить данные.
- Выполнить логику, зависящую от сырых данных.


## 11. Как использовать @after валидатор и в каких случаях он применим?
`@after` валидатор (или `@validator` без `pre=True`, либо `@after_validator`) используется для выполнения валидации **после** стандартной проверки типов и правил Pydantic. Он решает задачи, связанные с дополнительной проверкой или обработкой данных, которые уже прошли стандартную валидацию.

---

### **Когда использовать `@after` валидатор?**

1. **Дополнительная проверка данных**:
    
    - Если нужно проверить данные после того, как Pydantic применил стандартные правила (например, проверить, что число находится в допустимом диапазоне).
        
2. **Логика, зависящая от валидированных данных**:
    
    - Если валидация зависит от данных, которые уже прошли стандартную проверку (например, сравнение двух полей).
        
3. **Изменение данных после валидации**:
    
    - Если нужно изменить значение поля после стандартной валидации (например, форматирование строки).

## 12. Что такое @wrap валидатор и как он позволяет обернуть процесс валидации?
`@wrap` валидатор (или `@wrap_validator`) — это мощный инструмент в Pydantic, который позволяет **полностью контролировать процесс валидации**. Он оборачивает стандартную логику валидации, предоставляя возможность выполнить дополнительные действия до или после неё.

---

### **Когда использовать `@wrap` валидатор?**
1. **Кастомная логика валидации**:
   - Если нужно добавить сложную логику, которая не укладывается в стандартные валидаторы.

2. **Изменение процесса валидации**:
   - Если нужно изменить или дополнить стандартную валидацию Pydantic.

3. **Обработка исключений**:
   - Если нужно перехватывать и обрабатывать ошибки валидации.

4. **Логирование или отладка**:
   - Если нужно логировать данные или выполнять отладку в процессе валидации.

---

### **Как работает `@wrap` валидатор?**
1. **Параметры**:
   - `value`: Значение поля, которое нужно валидировать.
   - `handler`: Функция, которая выполняет стандартную валидацию Pydantic.

2. **Логика**:
   - Вы можете выполнить действия до вызова `handler` (например, изменить значение).
   - Вызвать `handler`, чтобы применить стандартную валидацию.
   - Выполнить действия после вызова `handler` (например, изменить результат).

3. **Возвращаемое значение**:
   - Валидатор должен вернуть значение, которое будет присвоено полю.

---

### **Пример использования `@wrap` валидатора**

#### 1. **Добавление логики до и после валидации**
```python
from pydantic import BaseModel, wrap_validator

class User(BaseModel):
    age: int

    @wrap_validator('age')
    def custom_age_validator(cls, value, handler):
        print(f"Before validation: {value}")  # Логирование до валидации
        result = handler(value)  # Стандартная валидация
        print(f"After validation: {result}")  # Логирование после валидации
        return result
```

#### 2. **Изменение значения до валидации**
```python
from pydantic import BaseModel, wrap_validator

class User(BaseModel):
    age: int

    @wrap_validator('age')
    def convert_age_to_int(cls, value, handler):
        if isinstance(value, str):
            value = int(value)  # Преобразуем строку в число
        return handler(value)  # Стандартная валидация
```

#### 3. **Обработка исключений**
```python
from pydantic import BaseModel, wrap_validator, ValidationError

class User(BaseModel):
    age: int

    @wrap_validator('age')
    def handle_validation_errors(cls, value, handler):
        try:
            return handler(value)  # Стандартная валидация
        except ValidationError as e:
            print(f"Validation error: {e}")  # Логирование ошибки
            raise  # Повторно выбрасываем исключение
```

#### 4. **Кастомная логика валидации**
```python
from pydantic import BaseModel, wrap_validator

class User(BaseModel):
    age: int

    @wrap_validator('age')
    def custom_age_validator(cls, value, handler):
        if value < 0:
            raise ValueError('Age must be positive')  # Кастомная проверка
        return handler(value)  # Стандартная валидация
```

---

### **Итог**
`@wrap` валидатор позволяет:
- Полностью контролировать процесс валидации.
- Добавлять логику до или после стандартной валидации.
- Обрабатывать исключения и выполнять сложные проверки.

Используй его, когда стандартных валидаторов недостаточно для решения задачи! 😊
## 13. Как работает @plain валидатор и чем он отличается от других типов валидаторов?
### **Чем `@plain` валидатор отличается от других валидаторов?**

|**Характеристика**|**`@plain` валидатор**|**Другие валидаторы (`@validator`, `@root_validator`)**|
|---|---|---|
|**Привязка к модели**|Нет|Да (привязан к конкретной модели или полю)|
|**Доступ к другим полям**|Нет|Да (через `values` в `@root_validator`)|
|**Использование**|Независимая функция, может быть вызвана вручную|Автоматически вызывается при создании модели|
|**Пример использования**|Валидация данных вне контекста модели|Валидация данных в контексте модели|

`@plain` валидатор:

- Это независимая функция валидации, не привязанная к модели.
    
- Полезен для создания переиспользуемой логики валидации.
    
- Отличается от других валидаторов отсутствием привязки к модели и контексту.
## 14. Как можно использовать вложенные модели в Pydantic?
Вложенные модели в Pydantic позволяют создавать сложные структуры данных, где одна модель может содержать другую. Это полезно для работы с иерархическими данными, такими как JSON, API-запросы или конфигурации.

---

### **Как использовать вложенные модели?**

1. **Определение вложенной модели**:
   Создай отдельную модель, которая будет использоваться как поле в другой модели.

2. **Использование вложенной модели**:
   Укажи вложенную модель как тип поля в основной модели.

3. **Валидация и сериализация**:
   Pydantic автоматически валидирует и сериализует вложенные модели.

---

### **Пример использования вложенных моделей**

#### 1. **Простая вложенная модель**
```python
from pydantic import BaseModel

class Address(BaseModel):
    city: str
    street: str

class User(BaseModel):
    name: str
    age: int
    address: Address  # Вложенная модель

# Создание объекта
user = User(
    name="John",
    age=30,
    address={"city": "New York", "street": "5th Ave"}  # Вложенные данные
)

print(user)
# name='John' age=30 address=Address(city='New York', street='5th Ave')

print(user.address.city)  # Доступ к полю вложенной модели
# New York
```

#### 2. **Вложенные модели в списках**
```python
from typing import List
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

class Order(BaseModel):
    id: int
    items: List[Item]  # Список вложенных моделей

# Создание объекта
order = Order(
    id=1,
    items=[
        {"name": "Laptop", "price": 1200.0},
        {"name": "Mouse", "price": 25.0}
    ]
)

print(order)
# id=1 items=[Item(name='Laptop', price=1200.0), Item(name='Mouse', price=25.0)]
```

#### 3. **Многоуровневые вложенные модели**
```python
from pydantic import BaseModel

class Profile(BaseModel):
    bio: str
    website: str

class Address(BaseModel):
    city: str
    street: str

class User(BaseModel):
    name: str
    age: int
    address: Address
    profile: Profile  # Еще одна вложенная модель

# Создание объекта
user = User(
    name="Alice",
    age=25,
    address={"city": "Los Angeles", "street": "Sunset Blvd"},
    profile={"bio": "Software Engineer", "website": "https://example.com"}
)

print(user)
# name='Alice' age=25 address=Address(city='Los Angeles', street='Sunset Blvd') profile=Profile(bio='Software Engineer', website='https://example.com')
```

---

### **Преимущества вложенных моделей**
1. **Организация данных**:
   - Позволяют структурировать сложные данные.
2. **Валидация**:
   - Вложенные модели автоматически валидируются.
3. **Сериализация**:
   - Вложенные модели автоматически сериализуются в JSON или словарь.
4. **Повторное использование**:
   - Модели можно использовать в разных контекстах.

---

### **Пример сериализации вложенных моделей**
```python
# Сериализация в словарь
user_dict = user.dict()
print(user_dict)
# {
#   'name': 'Alice',
#   'age': 25,
#   'address': {'city': 'Los Angeles', 'street': 'Sunset Blvd'},
#   'profile': {'bio': 'Software Engineer', 'website': 'https://example.com'}
# }

# Сериализация в JSON
user_json = user.json()
print(user_json)
# {"name": "Alice", "age": 25, "address": {"city": "Los Angeles", "street": "Sunset Blvd"}, "profile": {"bio": "Software Engineer", "website": "https://example.com"}}
```

---

### **Итог**
Вложенные модели в Pydantic:
- Позволяют создавать сложные иерархические структуры данных.
- Автоматически валидируются и сериализуются.
- Упрощают организацию и повторное использование кода.

Используй их для работы с API, конфигурациями или любыми сложными данными! 😊
## 15. Как игнорировать или исключать поля из сериализации в Pydantic модели?
- **`exclude`**: Исключает указанные поля.
    
- **`exclude_unset`**: Исключает не заданные поля.
    
- **`exclude_defaults`**: Исключает поля со значениями по умолчанию.
    
- **`exclude_none`**: Исключает поля со значением `None`.
    
- **`Field(exclude=True)`**: Постоянно исключает поле из сериализации.
- 
```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    password: str

user = User(name="John", age=30, password="secret")

# Динамическое исключение
exclude_fields = {"password"}
user_dict = user.dict(exclude=exclude_fields)
print(user_dict)  # {'name': 'John', 'age': 30}
```
## 16. Как работают алиасы полей (Field(..., alias='other_name')) в Pydantic?
- **Алиасы** позволяют использовать разные имена для полей при загрузке данных.
    
- **`by_alias=True`** используется для сериализации с алиасами.
    
- **`allow_population_by_field_name`** позволяет загружать данные как по алиасу, так и по имени поля.
    
```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(..., alias="full_name")
    age: int

    class Config:
        allow_population_by_field_name = True

# Загрузка данных по имени поля
data = {"name": "John", "age": 30}
user = User(**data)

print(user.name)  # John
print(user.age)   # 30
```
## 18. Как работает метод dict() в Pydantic и какие параметры он поддерживает?
Метод `dict()` в Pydantic используется для преобразования модели в словарь Python. Он поддерживает несколько параметров, которые позволяют контролировать, какие данные включать в результат и как их форматировать.

---

### **Основные параметры метода `dict()`**

| **Параметр**          | **Описание**                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| `include`             | Включает только указанные поля.                                             |
| `exclude`             | Исключает указанные поля.                                                   |
| `by_alias`            | Использует алиасы полей вместо оригинальных имен.                           |
| `exclude_unset`       | Исключает поля, которые не были заданы при создании объекта.                |
| `exclude_defaults`    | Исключает поля, которые имеют значения по умолчанию.                        |
| `exclude_none`        | Исключает поля, которые имеют значение `None`.                              |
| `exclude_defaults`    | Исключает поля, которые имеют значения по умолчанию.                        |

---

### **Примеры использования**

#### 1. **Базовое использование**
```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str = None

user = User(name="John", age=30)

# Преобразование модели в словарь
user_dict = user.dict()
print(user_dict)  # {'name': 'John', 'age': 30, 'email': None}
```

#### 2. **Использование `include`**
```python
# Включаем только поле `name`
user_dict = user.dict(include={"name"})
print(user_dict)  # {'name': 'John'}
```


### **Комбинирование параметров**
Параметры можно комбинировать для более гибкого управления выводом.

#### Пример:
```python
# Исключаем поле `age` и используем алиасы
user_dict = user.dict(exclude={"age"}, by_alias=True)
print(user_dict)  # {'full_name': 'John'}
```

---
Метод `dict()` в Pydantic:
- Преобразует модель в словарь.
- Поддерживает параметры для контроля включаемых и исключаемых полей.
- Позволяет использовать алиасы и исключать не заданные поля или поля со значениями по умолчанию.
## Field
`Field` — это функция в Pydantic, которая позволяет **настраивать поля модели** с дополнительными параметрами, такими как описание, значения по умолчанию, алиасы, ограничения и многое другое. Она используется для более тонкого контроля над поведением полей модели.

---

### **Основные параметры `Field`**

| **Параметр**          | **Описание**                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| `default`             | Значение по умолчанию для поля.                                             |
| `alias`               | Альтернативное имя для поля (например, для работы с JSON).                  |
| `description`         | Описание поля (полезно для документации).                                   |
| `gt`, `ge`, `lt`, `le`| Ограничения на значение (больше, больше или равно, меньше, меньше или равно).|
| `min_length`, `max_length` | Ограничения на длину строки или коллекции.                             |
| `regex`               | Регулярное выражение для валидации строки.                                  |
| `exclude`             | Исключает поле из сериализации.                                             |
| `example`             | Пример значения для документации.                                           |

---

### **Примеры использования `Field`**

#### 1. **Значение по умолчанию**
```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(default="Anonymous")
    age: int = Field(default=18)

user = User()
print(user)  # name='Anonymous' age=18
```

#### 2. **Алиасы**
```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(..., alias="full_name")

user = User(full_name="John")
print(user.name)  # John
```

#### 3. **Ограничения на значение**
```python
from pydantic import BaseModel, Field, ValidationError

class User(BaseModel):
    age: int = Field(..., gt=0, lt=100)  # Возраст от 1 до 99

try:
    user = User(age=150)  # Ошибка: возраст должен быть меньше 100
except ValidationError as e:
    print(e)
```

#### 4. **Ограничения на длину строки**
```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(..., min_length=3, max_length=50)

user = User(name="John")
print(user)  # name='John'
```

#### 5. **Регулярное выражение**
```python
from pydantic import BaseModel, Field, ValidationError

class User(BaseModel):
    email: str = Field(..., regex=r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$")

try:
    user = User(email="invalid-email")  # Ошибка: неверный формат email
except ValidationError as e:
    print(e)
```

#### 6. **Описание и пример**
```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(..., description="Full name of the user", example="John Doe")
    age: int = Field(..., description="Age of the user", example=30)

print(User.schema())
# {
#   "title": "User",
#   "type": "object",
#   "properties": {
#     "name": {"description": "Full name of the user", "example": "John Doe", "title": "Name", "type": "string"},
#     "age": {"description": "Age of the user", "example": 30, "title": "Age", "type": "integer"}
#   },
#   "required": ["name", "age"]
# }
```

#### 7. **Исключение поля из сериализации**
```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str
    password: str = Field(..., exclude=True)  # Поле исключено из сериализации

user = User(name="John", password="secret")
print(user.dict())  # {'name': 'John'}
```

---

### **Итог**
`Field` в Pydantic:
- Позволяет настраивать поля модели с дополнительными параметрами.
- Поддерживает ограничения, алиасы, описания, примеры и многое другое.
- Упрощает валидацию и документацию данных.

Используй `Field` для более гибкого и мощного управления полями модели! 😊
## Когда нужен Field когда validtor

`Field` и `validator` в Pydantic решают разные задачи, и их использование зависит от того, что именно нужно сделать с данными. Вот основные различия и примеры, когда что использовать:

---

### **Когда использовать `Field`?**

`Field` используется для **настройки полей модели** и задания дополнительных параметров, таких как:
- Значения по умолчанию.
- Алиасы (альтернативные имена полей).
- Ограничения на значения (например, минимальное/максимальное значение, длина строки).
- Описания и примеры для документации.
- Исключение полей из сериализации.

#### Примеры:
1. **Ограничения на значение**:
   ```python
   from pydantic import BaseModel, Field

   class User(BaseModel):
       age: int = Field(..., gt=0, lt=100)  # Возраст от 1 до 99
   ```

2. **Алиасы**:
   ```python
   class User(BaseModel):
       name: str = Field(..., alias="full_name")
   ```

3. **Значение по умолчанию**:
   ```python
   class User(BaseModel):
       name: str = Field(default="Anonymous")
   ```

4. **Исключение поля из сериализации**:
   ```python
   class User(BaseModel):
       password: str = Field(..., exclude=True)
   ```

---

### **Когда использовать `validator`?**

`validator` используется для **кастомной логики валидации**, которая не может быть выражена через параметры `Field`. Например:
- Проверка нескольких полей вместе.
- Сложная логика валидации (например, проверка формата email).
- Изменение значений перед валидацией.

#### Примеры:
1. **Проверка нескольких полей**:
   ```python
   from pydantic import BaseModel, validator

   class User(BaseModel):
       password: str
       confirm_password: str

       @validator('confirm_password')
       def passwords_match(cls, confirm_password, values):
           if 'password' in values and confirm_password != values['password']:
               raise ValueError('Passwords do not match')
           return confirm_password
   ```

2. **Кастомная логика валидации**:
   ```python
   class User(BaseModel):
       age: int

       @validator('age')
       def check_age(cls, age):
           if age < 0:
               raise ValueError('Age must be positive')
           return age
   ```

3. **Изменение значения перед валидацией**:
   ```python
   class User(BaseModel):
       name: str

       @validator('name', pre=True)
       def format_name(cls, name):
           return name.title()  # Приводим имя к заглавным буквам
   ```

---

### **Основные различия**

| **Характеристика**     | **Field**                                    | **Validator**                                 |
| ---------------------- | -------------------------------------------- | --------------------------------------------- |
| **Назначение**         | Настройка полей (ограничения, алиасы и т.д.) | Кастомная логика валидации                    |
| **Когда использовать** | Для простых ограничений и настроек полей     | Для сложной логики, проверки нескольких полей |
| **Пример**             | `age: int = Field(..., gt=0)`                | `@validator('age')`                           |

---

### **Когда что использовать?**
- **Используй `Field`**, если нужно:
  - Задать ограничения на значение (например, минимальное/максимальное значение).
  - Указать значение по умолчанию.
  - Использовать алиасы для полей.
  - Добавить описание или пример для документации.

- **Используй `validator`**, если нужно:
  - Реализовать сложную логику валидации.
  - Проверить несколько полей вместе.
  - Изменить значение перед валидацией.

---

### **Пример совместного использования**
```python
from pydantic import BaseModel, Field, validator

class User(BaseModel):
    name: str = Field(..., min_length=3, max_length=50)  # Ограничение длины строки
    age: int = Field(..., gt=0, lt=100)  # Ограничение на возраст
    email: str

    @validator('email')
    def check_email(cls, email):
        if '@' not in email:
            raise ValueError('Invalid email format')
        return email
```

---

### **Итог**
- **`Field`** — для настройки полей и простых ограничений.
- **`validator`** — для кастомной логики валидации.

Используй их вместе, чтобы создавать мощные и гибкие модели данных! 😊

# FastAPI

## 19. Что такое FastAPI и для каких задач он предназначен?
**FastAPI** — современный фреймворк для создания быстрых и простых API на Python.  

### **Для чего?**
1. **RESTful API** и **микросервисы**.
2. **Асинхронные задачи** (базы данных, внешние API).
3. **Прототипирование** (быстрая разработка).

### **Плюсы:**
- Высокая **производительность** (как у Node.js/Go).
- **Автоматическая документация** (Swagger/ReDoc).
- Простота и **поддержка Pydantic** для валидации данных.
- **Асинхронность** для масштабируемости.

### **Пример:**
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```


## 20. Как использовать Depends для внедрения зависимостей в маршруты FastAPI?
В FastAPI **`Depends`** используется для внедрения зависимостей в маршруты. Это позволяет разделять логику, упрощать тестирование и повторно использовать код. Зависимости могут быть функциями, классами или даже другими зависимостями.

---

### **Как работает `Depends`?**
1. **Зависимость** — это функция или класс, который возвращает значение.
2. **`Depends`** автоматически вызывает зависимость и передает результат в маршрут.
3. Зависимости могут быть **вложенными** (одна зависимость может зависеть от другой).

---

### **Примеры использования `Depends`**

#### 1. **Простая зависимость (функция)**
```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Зависимость
def get_common_parameters():
    return {"message": "Hello, World!"}

# Маршрут с зависимостью
@app.get("/")
async def root(common_params: dict = Depends(get_common_parameters)):
    return common_params
```

#### 2. **Зависимость с параметрами**
```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Зависимость с параметрами
def get_user(name: str, age: int):
    return {"name": name, "age": age}

# Маршрут с зависимостью
@app.get("/user/")
async def read_user(user: dict = Depends(get_user)):
    return user
```

#### 3. **Зависимость как класс**
```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Зависимость как класс
class User:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

# Маршрут с зависимостью
@app.get("/user/")
async def read_user(user: User = Depends(User)):
    return {"name": user.name, "age": user.age}
```

#### 4. **Вложенные зависимости**
```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Первая зависимость
def get_db():
    return "Database connection"

# Вторая зависимость, зависящая от первой
def get_user(db: str = Depends(get_db)):
    return {"user": "John", "db": db}

# Маршрут с вложенными зависимостями
@app.get("/user/")
async def read_user(user: dict = Depends(get_user)):
    return user
```

#### 5. **Зависимость для аутентификации**
```python
from fastapi import FastAPI, Depends, HTTPException

app = FastAPI()

# Зависимость для аутентификации
def authenticate(token: str):
    if token != "secret":
        raise HTTPException(status_code=401, detail="Unauthorized")
    return {"user": "John"}

# Маршрут с аутентификацией
@app.get("/secure/")
async def secure_route(user: dict = Depends(authenticate)):
    return {"message": f"Hello, {user['user']}!"}
```

---

### **Преимущества использования `Depends`**
1. **Разделение логики**:
   - Зависимости позволяют выносить повторяющийся код (например, аутентификацию) в отдельные функции или классы.

2. **Повторное использование**:
   - Зависимости можно использовать в нескольких маршрутах.

3. **Тестируемость**:
   - Зависимости легко тестировать отдельно от маршрутов.

4. **Гибкость**:
   - Зависимости могут быть функциями, классами или даже асинхронными.

---

### **Итог**
- **`Depends`** используется для внедрения зависимостей в маршруты.
- Зависимости могут быть функциями, классами или вложенными.
- Это упрощает код, делает его более модульным и тестируемым.

Используй `Depends` для создания чистого и поддерживаемого кода в FastAPI! 😊
## 21. Чем отличается синхронная и асинхронная зависимость в FastAPI?
В FastAPI зависимости могут быть как **синхронными**, так и **асинхронными**. Разница между ними заключается в том, как они выполняются и как взаимодействуют с другими частями приложения. Вот основные различия:

---

### **1. Синхронная зависимость**
- **Определение**: Это обычная функция, которая выполняется синхронно (последовательно).
- **Когда использовать**:
  - Если зависимость выполняет быстрые операции (например, простые вычисления или доступ к памяти).
  - Если зависимость не требует ожидания (например, работа с CPU-bound задачами).

#### **Пример синхронной зависимости:**
```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Синхронная зависимость
def get_sync_data():
    return {"message": "Hello, World!"}

@app.get("/sync")
async def read_sync(data: dict = Depends(get_sync_data)):
    return data
```

---

### **2. Асинхронная зависимость**
- **Определение**: Это функция, объявленная с `async def`, которая может выполнять асинхронные операции (например, запросы к базе данных или внешним API).
- **Когда использовать**:
  - Если зависимость выполняет I/O-bound задачи (например, запросы к базе данных, HTTP-запросы).
  - Если нужно избежать блокировки основного потока выполнения.

#### **Пример асинхронной зависимости:**
```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Асинхронная зависимость
async def get_async_data():
    return {"message": "Hello, Async World!"}

@app.get("/async")
async def read_async(data: dict = Depends(get_async_data)):
    return data
```

---

### **Ключевые различия**

| **Характеристика**       | **Синхронная зависимость**                  | **Асинхронная зависимость**                  |
|--------------------------|--------------------------------------------|---------------------------------------------|
| **Определение**          | Обычная функция (`def`).                   | Асинхронная функция (`async def`).          |
| **Использование**        | Для быстрых операций (CPU-bound).          | Для операций с ожиданием (I/O-bound).       |
| **Блокировка потока**    | Может блокировать поток выполнения.        | Не блокирует поток выполнения.              |
| **Примеры задач**        | Простые вычисления, работа с памятью.      | Запросы к базе данных, HTTP-запросы.        |

---

### **3. Смешанное использование**
FastAPI позволяет использовать как синхронные, так и асинхронные зависимости в одном приложении. Например:

```python
from fastapi import FastAPI, Depends

app = FastAPI()

# Синхронная зависимость
def get_sync_data():
    return {"sync_message": "Hello, Sync World!"}

# Асинхронная зависимость
async def get_async_data():
    return {"async_message": "Hello, Async World!"}

@app.get("/mixed")
async def read_mixed(
    sync_data: dict = Depends(get_sync_data),
    async_data: dict = Depends(get_async_data)
):
    return {**sync_data, **async_data}
```

---

### **4. Когда что использовать?**
- **Синхронные зависимости**:
  - Для быстрых операций, которые не требуют ожидания.
  - Для CPU-bound задач (например, сложные вычисления).

- **Асинхронные зависимости**:
  - Для операций с ожиданием (например, запросы к базе данных, внешние API).
  - Для I/O-bound задач, чтобы избежать блокировки потока.

---

### **Итог**
- **Синхронные зависимости** — для быстрых операций без ожидания.
- **Асинхронные зависимости** — для операций с ожиданием, чтобы избежать блокировки потока.

Используй их в зависимости от задачи, чтобы сделать приложение более эффективным! 😊

## 22. Как создать и использовать BackgroundTasks в FastAPI?
Нет, **`BackgroundTasks` в FastAPI не использует отдельные потоки** по умолчанию. Вместо этого он выполняет задачи в том же потоке (или event loop), что и основное приложение, но **после отправки ответа клиенту**. Это важно понимать, чтобы избежать неправильных ожиданий.

---

### **Как работает `BackgroundTasks`?**

1. **Синхронные задачи**:
   - Если задача синхронная (обычная функция, `def`), она выполняется в основном потоке, но **после отправки ответа клиенту**.
   - Это может блокировать event loop, если задача долгая или CPU-bound.

2. **Асинхронные задачи**:
   - Если задача асинхронная (`async def`), она выполняется в том же event loop, что и основное приложение.
   - Это не блокирует event loop, так как асинхронные задачи могут "ожидать" (например, с помощью `await`).

3. **Отдельные потоки**:
   - По умолчанию `BackgroundTasks` **не использует отдельные потоки**. Если нужно выполнить CPU-bound задачи или задачи, которые блокируют поток, лучше использовать **`concurrent.futures.ThreadPoolExecutor`** или **`asyncio.to_thread`**.

---

### **Пример с синхронной задачей**

```python
from fastapi import FastAPI, BackgroundTasks
import time

app = FastAPI()

# Синхронная задача
def sync_task(message: str):
    time.sleep(5)  # Имитация долгой синхронной задачи
    print(f"Sync task completed: {message}")

@app.post("/sync-task/")
async def run_sync_task(message: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(sync_task, message)
    return {"message": "Sync task started in the background"}
```

- **Что происходит**:
  - Задача `sync_task` выполняется в основном потоке, но после отправки ответа клиенту.
  - Если задача долгая, она может блокировать event loop.

---

### **Пример с асинхронной задачей**

```python
from fastapi import FastAPI, BackgroundTasks
import asyncio

app = FastAPI()

# Асинхронная задача
async def async_task(message: str):
    await asyncio.sleep(5)  # Имитация долгой асинхронной задачи
    print(f"Async task completed: {message}")

@app.post("/async-task/")
async def run_async_task(message: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(async_task, message)
    return {"message": "Async task started in the background"}
```

- **Что происходит**:
  - Задача `async_task` выполняется в том же event loop, что и основное приложение.
  - Она не блокирует event loop, так как использует `await`.

---

### **Как выполнить задачу в отдельном потоке?**

Если нужно выполнить CPU-bound задачу или задачу, которая блокирует поток, используй **`asyncio.to_thread`** или **`concurrent.futures.ThreadPoolExecutor`**.

#### Пример с `asyncio.to_thread`:
```python
from fastapi import FastAPI, BackgroundTasks
import time
import asyncio

app = FastAPI()

# Синхронная задача, которая блокирует поток
def blocking_task(message: str):
    time.sleep(5)  # Имитация долгой блокирующей задачи
    print(f"Blocking task completed: {message}")

@app.post("/blocking-task/")
async def run_blocking_task(message: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(asyncio.to_thread, blocking_task, message)
    return {"message": "Blocking task started in the background"}
```

- **Что происходит**:
  - Задача `blocking_task` выполняется в отдельном потоке, что не блокирует event loop.

---

### **Итог**
- **`BackgroundTasks`** по умолчанию выполняет задачи в том же потоке (или event loop), что и основное приложение.
- Для **синхронных задач** это может блокировать event loop.
- Для **асинхронных задач** блокировки не происходит.
- Если нужно выполнить CPU-bound или блокирующую задачу, используй **`asyncio.to_thread`** или **`ThreadPoolExecutor`**.

Пример с `asyncio.to_thread`:

```python
from fastapi import FastAPI, BackgroundTasks
import time
import asyncio

app = FastAPI()

def blocking_task(message: str):
    time.sleep(5)
    print(f"Blocking task completed: {message}")

@app.post("/blocking-task/")
async def run_blocking_task(message: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(asyncio.to_thread, blocking_task, message)
    return {"message": "Blocking task started in the background"}
```

Теперь ты знаешь, как управлять фоновыми задачами в FastAPI! 😊
## 23. Что такое middleware в FastAPI и как его можно добавить в приложение?
**Middleware** в FastAPI — это слой, который обрабатывает запросы и ответы до того, как они достигнут маршрутов (endpoints) или после того, как они покинут маршруты. Middleware позволяет добавлять дополнительную логику, такую как аутентификация, логирование, обработка ошибок, CORS и многое другое.

---

### **Как работает Middleware?**
1. **Запрос**:
   - Middleware получает запрос до того, как он достигнет маршрута.
   - Может изменять запрос или выполнять дополнительные действия (например, логирование).

2. **Ответ**:
   - Middleware получает ответ после того, как он сгенерирован маршрутом.
   - Может изменять ответ или выполнять дополнительные действия (например, добавление заголовков).

---

### **Как добавить Middleware в FastAPI?**

Middleware добавляется с помощью метода `.add_middleware()` или декоратора `@app.middleware()`.

---

### **Пример 1: Логирование запросов и ответов**

```python
from fastapi import FastAPI, Request
import time

app = FastAPI()

# Middleware для логирования
@app.middleware("http")
async def log_middleware(request: Request, call_next):
    start_time = time.time()
    
    # Логируем запрос
    print(f"Request: {request.method} {request.url}")
    
    # Передаем запрос дальше
    response = await call_next(request)
    
    # Логируем ответ
    process_time = time.time() - start_time
    print(f"Response: {response.status_code}, Time: {process_time:.2f}s")
    
    return response

# Маршрут
@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```

- **Что происходит**:
  - Middleware логирует метод и URL запроса, а также время обработки и статус ответа.

---

### **Пример 2: Добавление заголовков в ответ**

```python
from fastapi import FastAPI, Request

app = FastAPI()

# Middleware для добавления заголовков
@app.middleware("http")
async def add_header_middleware(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Custom-Header"] = "Hello from Middleware"
    return response

# Маршрут
@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```

- **Что происходит**:
  - Middleware добавляет кастомный заголовок `X-Custom-Header` в каждый ответ.

---

### **Пример 3: Использование встроенного Middleware**

FastAPI предоставляет встроенные middleware для решения распространенных задач, таких как CORS.

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Добавляем CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Разрешаем все домены
    allow_credentials=True,
    allow_methods=["*"],  # Разрешаем все методы
    allow_headers=["*"],  # Разрешаем все заголовки
)

# Маршрут
@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```

- **Что происходит**:
  - Middleware добавляет поддержку CORS, чтобы разрешить запросы с других доменов.

---

### **Пример 4: Кастомный Middleware с использованием класса**

Middleware можно реализовать как класс, если требуется более сложная логика.

```python
from fastapi import FastAPI, Request

app = FastAPI()

# Кастомный Middleware как класс
class CustomMiddleware:
    def __init__(self, app):
        self.app = app

    async def __call__(self, scope, receive, send):
        # Логика до обработки запроса
        print("Before request")
        
        # Передаем запрос дальше
        await self.app(scope, receive, send)
        
        # Логика после обработки запроса
        print("After request")

# Добавляем Middleware
app.add_middleware(CustomMiddleware)

# Маршрут
@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```

- **Что происходит**:
  - Middleware логирует сообщения до и после обработки запроса.

---

### **Когда использовать Middleware?**
1. **Логирование**:
   - Логирование запросов и ответов.
2. **Аутентификация**:
   - Проверка токенов или других данных аутентификации.
3. **Обработка ошибок**:
   - Перехват и обработка исключений.
4. **CORS**:
   - Разрешение запросов с других доменов.
5. **Добавление заголовков**:
   - Добавление кастомных заголовков в ответы.

---

### **Итог**
- **Middleware** — это слой, который обрабатывает запросы и ответы.
- Можно добавлять через `.add_middleware()` или декоратор `@app.middleware()`.
- Используется для логирования, аутентификации, CORS и других задач.

Пример с логированием:

```python
from fastapi import FastAPI, Request
import time

app = FastAPI()

@app.middleware("http")
async def log_middleware(request: Request, call_next):
    start_time = time.time()
    print(f"Request: {request.method} {request.url}")
    response = await call_next(request)
    process_time = time.time() - start_time
    print(f"Response: {response.status_code}, Time: {process_time:.2f}s")
    return response

@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```

Теперь ты знаешь, как использовать Middleware в FastAPI! 😊
## 24. Как middleware в FastAPI может влиять на обработку запросов и ответов?
Middleware в FastAPI может:
1. **Изменять запросы**: Добавлять данные, проверять заголовки, логировать.
2. **Изменять ответы**: Добавлять заголовки, сжимать данные, логировать.
3. **Перехватывать ошибки**: Обрабатывать исключения до отправки ответа.
4. **Блокировать запросы**: Например, при неправильном хосте или отсутствии аутентификации.

Пример: Логирование запросов и ответов.

```python
@app.middleware("http")
async def log_middleware(request: Request, call_next):
    print(f"Request: {request.method} {request.url}")
    response = await call_next(request)
    print(f"Response: {response.status_code}")
    return response
``` 

Middleware — это мощный инструмент для добавления дополнительной логики в обработку запросов и ответов. 😊
## 25. Как использовать Pydantic модели для валидации запросов и ответов в FastAPI?
В FastAPI **Pydantic модели** используются для валидации и сериализации данных запросов и ответов. Это делает код чище, безопаснее и удобнее. Вот как это работает:

---

### **1. Валидация запросов**
Pydantic модели используются для валидации данных, передаваемых в запросе (например, в теле POST-запроса).

#### Пример:
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Модель для валидации запроса
class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

# Маршрут с валидацией запроса
@app.post("/items/")
async def create_item(item: Item):
    return item
```

- **Что происходит**:
  - FastAPI автоматически валидирует тело запроса с помощью модели `Item`.
  - Если данные не соответствуют модели, возвращается ошибка `422 Unprocessable Entity`.

---

### **2. Валидация ответов**
Pydantic модели также используются для валидации данных, возвращаемых в ответе.

#### Пример:
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Модель для валидации ответа
class User(BaseModel):
    username: str
    email: str
    age: int

# Маршрут с валидацией ответа
@app.get("/user/", response_model=User)
async def read_user():
    return {"username": "john", "email": "john@example.com", "age": 30}
```

- **Что происходит**:
  - FastAPI проверяет, что ответ соответствует модели `User`.
  - Если данные не соответствуют модели, возникает ошибка.

---

### **3. Валидация параметров запроса**
Pydantic модели можно использовать для валидации параметров запроса (например, query-параметров).

#### Пример:
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Модель для валидации query-параметров
class Filter(BaseModel):
    limit: int
    offset: int = 0

# Маршрут с валидацией query-параметров
@app.get("/items/")
async def read_items(filter: Filter):
    return {"limit": filter.limit, "offset": filter.offset}
```

- **Что происходит**:
  - FastAPI автоматически валидирует query-параметры с помощью модели `Filter`.

---

### **4. Вложенные модели**
Pydantic поддерживает вложенные модели, что полезно для сложных данных.

#### Пример:
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# Вложенная модель
class Address(BaseModel):
    city: str
    street: str

# Основная модель
class User(BaseModel):
    name: str
    address: Address

# Маршрут с вложенной моделью
@app.post("/user/")
async def create_user(user: User):
    return user
```

- **Что происходит**:
  - FastAPI валидирует вложенные данные (например, `address` внутри `user`).

---

### **5. Кастомные валидаторы**
Pydantic позволяет добавлять кастомные валидаторы для более сложной логики.

#### Пример:
```python
from fastapi import FastAPI
from pydantic import BaseModel, validator

app = FastAPI()

# Модель с кастомным валидатором
class Item(BaseModel):
    name: str
    price: float

    @validator('price')
    def price_must_be_positive(cls, value):
        if value < 0:
            raise ValueError('Price must be positive')
        return value

# Маршрут с кастомной валидацией
@app.post("/items/")
async def create_item(item: Item):
    return item
```

- **Что происходит**:
  - Если `price` отрицательный, возвращается ошибка `422 Unprocessable Entity`.

---

### **Итог**
- **Pydantic модели** в FastAPI используются для валидации запросов, ответов и параметров.
- Они делают код безопасным, чистым и удобным.
- Поддерживают вложенные модели и кастомные валидаторы.

Пример:

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float

@app.post("/items/")
async def create_item(item: Item):
    return item
```

Теперь ты знаешь, как использовать Pydantic модели в FastAPI! 😊
## 26. Как в FastAPI работает система маршрутов и как можно задавать параметры в пути?
В FastAPI система маршрутов (routing) позволяет определять, как приложение обрабатывает HTTP-запросы. Маршруты задаются с помощью декораторов, таких как `@app.get()`, `@app.post()`, и т.д. Параметры в пути (path parameters) позволяют динамически обрабатывать запросы в зависимости от значений, переданных в URL.

---

### **Как работают маршруты в FastAPI?**

1. **Декораторы**:
   - Маршруты определяются с помощью декораторов, например:
     - `@app.get("/path")` — для GET-запросов.
     - `@app.post("/path")` — для POST-запросов.
     - `@app.put("/path")` — для PUT-запросов.
     - `@app.delete("/path")` — для DELETE-запросов.

2. **Функции-обработчики**:
   - Каждый маршрут связан с функцией, которая обрабатывает запрос и возвращает ответ.

3. **Параметры в пути**:
   - Параметры в пути позволяют динамически обрабатывать запросы. Они указываются в фигурных скобках `{}` в URL.

---

### **Как задавать параметры в пути?**

Параметры в пути задаются с помощью фигурных скобок `{}` в URL. FastAPI автоматически извлекает значения и передает их в функцию-обработчик.

#### Пример 1: Простой параметр
```python
from fastapi import FastAPI

app = FastAPI()

# Маршрут с параметром в пути
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

- **Что происходит**:
  - Если запросить `/items/42`, FastAPI передаст `item_id=42` в функцию `read_item`.
  - Параметр `item_id` автоматически преобразуется в `int` (если возможно).

---

#### Пример 2: Несколько параметров
```python
from fastapi import FastAPI

app = FastAPI()

# Маршрут с несколькими параметрами
@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(user_id: int, item_id: int):
    return {"user_id": user_id, "item_id": item_id}
```

- **Что происходит**:
  - Если запросить `/users/1/items/42`, FastAPI передаст `user_id=1` и `item_id=42` в функцию `read_user_item`.

---

#### Пример 3: Параметр с типом данных
FastAPI автоматически валидирует параметры в пути на основе аннотаций типов.

```python
from fastapi import FastAPI

app = FastAPI()

# Параметр с типом данных
@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

- **Что происходит**:
  - Если запросить `/items/abc`, FastAPI вернет ошибку `422 Unprocessable Entity`, так как `abc` нельзя преобразовать в `int`.

---

#### Пример 4: Параметр с регулярным выражением
Можно использовать регулярные выражения для ограничения формата параметра.

```python
from fastapi import FastAPI

app = FastAPI()

# Параметр с регулярным выражением
@app.get("/items/{item_id:^\\d+$}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

- **Что происходит**:
  - Параметр `item_id` должен состоять только из цифр (`\d+`).

---

#### Пример 5: Параметр с типом `str` и значением по умолчанию
Если параметр не обязательный, можно задать значение по умолчанию.

```python
from fastapi import FastAPI

app = FastAPI()

# Параметр с типом `str` и значением по умолчанию
@app.get("/items/{item_id}")
async def read_item(item_id: str = "default"):
    return {"item_id": item_id}
```

- **Что происходит**:
  - Если запросить `/items/`, FastAPI вернет `{"item_id": "default"}`.

---

### **Как работает система маршрутов?**

1. **Сопоставление URL**:
   - FastAPI сопоставляет URL запроса с определенными маршрутами.
   - Если маршрут найден, вызывается соответствующая функция.

2. **Извлечение параметров**:
   - Параметры извлекаются из URL и передаются в функцию.

3. **Валидация**:
   - Параметры автоматически валидируются на основе аннотаций типов.

4. **Генерация документации**:
   - FastAPI автоматически генерирует документацию Swagger и ReDoc на основе маршрутов.

---

### **Итог**
- **Маршруты** задаются с помощью декораторов (`@app.get()`, `@app.post()` и т.д.).
- **Параметры в пути** указываются в фигурных скобках `{}` и автоматически передаются в функцию.
- FastAPI автоматически валидирует параметры и генерирует документацию.

Пример:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```


## 27. Как управлять состоянием сессии в FastAPI (например, использование state)?
`app.state` в FastAPI используется для хранения **глобального состояния приложения**, которое доступно всем маршрутам и middleware. Это полезно для хранения данных, которые должны быть доступны на протяжении всего жизненного цикла приложения, таких как подключения к базам данных, конфигурации, кэши или любые другие общие ресурсы.

---

### **Примеры реального применения `app.state`**

#### 1. **Хранение подключения к базе данных**
`app.state` часто используется для хранения подключения к базе данных, чтобы избежать повторного создания подключения для каждого запроса.

```python
from fastapi import FastAPI, Depends
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session

app = FastAPI()

# Создаем подключение к базе данных
DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Сохраняем подключение в app.state
app.state.db_engine = engine
app.state.db_session = SessionLocal

# Зависимость для получения сессии базы данных
def get_db():
    db = app.state.db_session()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
async def read_items(db: Session = Depends(get_db)):
    items = db.query(Item).all()
    return {"items": items}
```

- **Что происходит**:
  - Подключение к базе данных создается один раз при старте приложения и сохраняется в `app.state`.
  - Для каждого запроса создается новая сессия базы данных, которая автоматически закрывается после завершения запроса.

---

#### 2. **Хранение конфигурации**
`app.state` можно использовать для хранения конфигурации приложения, например, API-ключей или настроек.

```python
from fastapi import FastAPI

app = FastAPI()

# Сохраняем конфигурацию в app.state
app.state.config = {
    "api_key": "your-api-key",
    "max_requests": 100,
}

@app.get("/config/")
async def get_config():
    return {"config": app.state.config}
```

- **Что происходит**:
  - Конфигурация хранится в `app.state` и доступна всем маршрутам.

---

#### 3. **Кэширование данных**
`app.state` можно использовать для кэширования данных, чтобы избежать повторных вычислений или запросов к внешним API.

```python
from fastapi import FastAPI

app = FastAPI()

# Инициализируем кэш в app.state
app.state.cache = {}

@app.get("/cached-data/")
async def get_cached_data():
    if "data" not in app.state.cache:
        # Имитация долгого вычисления или запроса
        app.state.cache["data"] = {"result": "cached data"}
    return app.state.cache["data"]
```

- **Что происходит**:
  - Данные кэшируются в `app.state.cache` и используются для последующих запросов.

---

#### 4. **Хранение подключения к внешнему сервису**
`app.state` можно использовать для хранения подключения к внешнему сервису, например, Redis или RabbitMQ.

```python
from fastapi import FastAPI
import redis

app = FastAPI()

# Подключаемся к Redis и сохраняем подключение в app.state
app.state.redis = redis.Redis(host="localhost", port=6379, db=0)

@app.get("/redis/")
async def get_redis_data(key: str):
    value = app.state.redis.get(key)
    return {"key": key, "value": value}
```

- **Что происходит**:
  - Подключение к Redis создается один раз при старте приложения и сохраняется в `app.state`.
  - Все маршруты могут использовать это подключение для работы с Redis.

---

#### 5. **Хранение состояния приложения**
`app.state` можно использовать для хранения состояния приложения, например, счетчика запросов.

```python
from fastapi import FastAPI

app = FastAPI()

# Инициализируем счетчик запросов
app.state.request_count = 0

@app.get("/request-count/")
async def get_request_count():
    app.state.request_count += 1
    return {"request_count": app.state.request_count}
```

- **Что происходит**:
  - Счетчик запросов хранится в `app.state` и увеличивается при каждом запросе.

---

### **Преимущества использования `app.state`**
1. **Глобальный доступ**:
   - Данные доступны всем маршрутам и middleware.
2. **Эффективность**:
   - Ресурсы создаются один раз и используются повторно.
3. **Простота**:
   - Не нужно передавать данные через параметры функций.

---

### **Итог**
`app.state` в FastAPI используется для хранения глобального состояния приложения, такого как:
- Подключения к базам данных.
- Конфигурации.
- Кэши.
- Подключения к внешним сервисам.
- Состояние приложения.

Пример с подключением к базе данных:

```python
from fastapi import FastAPI, Depends
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session

app = FastAPI()

DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

app.state.db_engine = engine
app.state.db_session = SessionLocal

def get_db():
    db = app.state.db_session()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
async def read_items(db: Session = Depends(get_db)):
    items = db.query(Item).all()
    return {"items": items}
```


## 28. Как использовать декоратор @app.exception_handler для обработки исключений?
Декоратор **`@app.exception_handler`** в FastAPI позволяет перехватывать и обрабатывать исключения на уровне приложения. Это полезно для создания кастомных ответов на ошибки, таких как `404 Not Found` или `500 Internal Server Error`.

---

### **Как работает `@app.exception_handler`?**

1. **Перехват исключений**:
   - Декоратор `@app.exception_handler` перехватывает исключения определенного типа.
   - Можно перехватывать как встроенные исключения (например, `HTTPException`), так и кастомные.

2. **Кастомный ответ**:
   - В обработчике можно вернуть кастомный ответ (например, JSON с описанием ошибки).

---

### **Примеры использования**

#### 1. **Обработка `HTTPException`**
FastAPI автоматически генерирует `HTTPException` для ошибок, таких как `404 Not Found`. Можно перехватить это исключение и вернуть кастомный ответ.

```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse

app = FastAPI()

# Кастомный обработчик для HTTPException
@app.exception_handler(HTTPException)
async def http_exception_handler(request: Request, exc: HTTPException):
    return JSONResponse(
        status_code=exc.status_code,
        content={"message": f"Oops! {exc.detail}"},
    )

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 42:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item_id": item_id}
```

- **Что происходит**:
  - Если запросить `/items/42`, будет вызвано исключение `HTTPException`.
  - Обработчик вернет кастомный JSON-ответ: `{"message": "Oops! Item not found"}`.

---

#### 2. **Обработка кастомных исключений**
Можно создать собственное исключение и обработать его.

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

# Кастомное исключение
class CustomException(Exception):
    def __init__(self, message: str):
        self.message = message

# Обработчик для кастомного исключения
@app.exception_handler(CustomException)
async def custom_exception_handler(request: Request, exc: CustomException):
    return JSONResponse(
        status_code=400,
        content={"message": f"Custom error: {exc.message}"},
    )

@app.get("/custom-error/")
async def trigger_custom_error():
    raise CustomException(message="Something went wrong!")
```

- **Что происходит**:
  - Если запросить `/custom-error/`, будет вызвано кастомное исключение.
  - Обработчик вернет JSON-ответ: `{"message": "Custom error: Something went wrong!"}`.

---

#### 3. **Обработка всех исключений**
Можно перехватывать все исключения с помощью `Exception`.

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

# Обработчик для всех исключений
@app.exception_handler(Exception)
async def generic_exception_handler(request: Request, exc: Exception):
    return JSONResponse(
        status_code=500,
        content={"message": "Internal Server Error"},
    )

@app.get("/error/")
async def trigger_error():
    raise ValueError("Something went wrong!")
```

- **Что происходит**:
  - Если запросить `/error/`, будет вызвано исключение `ValueError`.
  - Обработчик вернет JSON-ответ: `{"message": "Internal Server Error"}`.

---

#### 4. **Обработка `RequestValidationError`**
FastAPI автоматически генерирует `RequestValidationError`, если данные запроса не проходят валидацию. Можно перехватить это исключение и вернуть кастомный ответ.

```python
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

app = FastAPI()

# Обработчик для RequestValidationError
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={"message": "Validation error", "details": exc.errors()},
    )

@app.post("/items/")
async def create_item(item: dict):
    return {"item": item}
```

- **Что происходит**:
  - Если отправить невалидные данные в `/items/`, будет вызвано исключение `RequestValidationError`.
  - Обработчик вернет JSON-ответ с деталями ошибки.

---

### **Преимущества использования `@app.exception_handler`**
1. **Кастомные ответы**:
   - Можно возвращать кастомные JSON-ответы или HTML-страницы.
2. **Централизованная обработка ошибок**:
   - Все исключения обрабатываются в одном месте.
3. **Гибкость**:
   - Можно перехватывать как встроенные, так и кастомные исключения.

---

### **Итог**
- **`@app.exception_handler`** позволяет перехватывать и обрабатывать исключения на уровне приложения.
- Можно обрабатывать встроенные исключения (например, `HTTPException`, `RequestValidationError`) и кастомные.
- Обработчик возвращает кастомный ответ (например, JSON).

Пример с `HTTPException`:

```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(HTTPException)
async def http_exception_handler(request: Request, exc: HTTPException):
    return JSONResponse(
        status_code=exc.status_code,
        content={"message": f"Oops! {exc.detail}"},
    )

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 42:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item_id": item_id}
```

Теперь ты знаешь, как использовать `@app.exception_handler` для обработки исключений в FastAPI! 😊

## 29. Что такое lifespan события в FastAPI и как их использовать?
**Lifespan события** в FastAPI — это механизм для выполнения кода при **запуске** и **завершении** работы приложения. Они позволяют управлять ресурсами, такими как подключения к базам данных, кэши или внешние сервисы, которые должны быть инициализированы до начала работы приложения и корректно закрыты при его завершении.

---

### **Как работают lifespan события?**

1. **Запуск приложения**:
   - Код, указанный в lifespan, выполняется **до** того, как приложение начнет обрабатывать запросы.
   - Это идеальное место для инициализации ресурсов (например, подключения к базе данных).

2. **Завершение работы приложения**:
   - Код, указанный в lifespan, выполняется **после** завершения работы приложения.
   - Это идеальное место для освобождения ресурсов (например, закрытия подключений).

---

### **Как использовать lifespan события?**

Lifespan события реализуются с помощью **асинхронного контекстного менеджера**. Для этого используется объект `lifespan`, который передается в `FastAPI`.

---

### **Пример использования**

#### 1. **Инициализация и закрытие подключения к базе данных**
```python
from fastapi import FastAPI
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Создаем подключение к базе данных
DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

app = FastAPI()

# Lifespan события
@app.on_event("startup")
async def startup():
    print("Starting up...")
    # Инициализация ресурсов (например, подключение к базе данных)
    app.state.db_engine = engine
    app.state.db_session = SessionLocal

@app.on_event("shutdown")
async def shutdown():
    print("Shutting down...")
    # Освобождение ресурсов (например, закрытие подключения к базе данных)
    app.state.db_engine.dispose()

@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```

- **Что происходит**:
  - При запуске приложения выполняется `startup`, где инициализируется подключение к базе данных.
  - При завершении работы приложения выполняется `shutdown`, где закрывается подключение.

---

#### 2. **Использование `lifespan` (начиная с FastAPI 0.95.0)**
Начиная с версии 0.95.0, рекомендуется использовать объект `lifespan` вместо `@app.on_event`.

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Создаем подключение к базе данных
DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Lifespan менеджер
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Код, выполняемый при запуске
    print("Starting up...")
    app.state.db_engine = engine
    app.state.db_session = SessionLocal
    yield
    # Код, выполняемый при завершении
    print("Shutting down...")
    app.state.db_engine.dispose()

app = FastAPI(lifespan=lifespan)

@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```

- **Что происходит**:
  - Код до `yield` выполняется при запуске приложения.
  - Код после `yield` выполняется при завершении работы приложения.

---

### **Преимущества использования lifespan событий**
1. **Инициализация ресурсов**:
   - Можно инициализировать подключения к базам данных, кэши, внешние сервисы и т.д.
2. **Освобождение ресурсов**:
   - Можно корректно закрыть ресурсы при завершении работы приложения.
3. **Централизованное управление**:
   - Весь код, связанный с жизненным циклом приложения, находится в одном месте.

---

### **Когда использовать lifespan события?**
- Для инициализации и закрытия подключений к базам данных.
- Для запуска и остановки фоновых задач.
- Для управления кэшем или другими ресурсами.

---

### **Итог**
- **Lifespan события** позволяют выполнять код при запуске и завершении работы приложения.
- Можно использовать `@app.on_event("startup")` и `@app.on_event("shutdown")` или объект `lifespan`.
- Это идеальное место для управления ресурсами, такими как подключения к базам данных.

Пример с `lifespan`:

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@asynccontextmanager
async def lifespan(app: FastAPI):
    print("Starting up...")
    app.state.db_engine = engine
    app.state.db_session = SessionLocal
    yield
    print("Shutting down...")
    app.state.db_engine.dispose()

app = FastAPI(lifespan=lifespan)

@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```



## 30. В чем разница между lifespan и событиями startup и shutdown?
|**Характеристика**|**`startup` и `shutdown`**|**Lifespan**|
|---|---|---|
|**Синтаксис**|Декораторы `@app.on_event`.|Асинхронный контекстный менеджер (`yield`).|
|**Гибкость**|Ограничена.|Высокая (можно передавать данные).|
|**Поддержка**|Устаревший подход (но все еще работает).|Рекомендуемый подход (начиная с FastAPI 0.95.0).|
|**Использование**|Простые сценарии.|Сложные сценарии (например, несколько ресурсов).|

---

### **Когда что использовать?**

## 31. Как обрабатывать ресурсы и их освобождение в рамках lifespan в FastAPI?
Обработка ресурсов и их освобождение в рамках **lifespan** в FastAPI — это важная часть управления жизненным циклом приложения. Lifespan позволяет инициализировать ресурсы (например, подключения к базам данных, кэши, внешние сервисы) при запуске приложения и корректно освобождать их при завершении работы.

---

### **Как это работает?**

1. **Инициализация ресурсов**:
   - Код до `yield` в lifespan выполняется при **запуске** приложения.
   - Здесь можно инициализировать ресурсы, такие как подключения к базам данных, кэши, пулы потоков и т.д.

2. **Освобождение ресурсов**:
   - Код после `yield` выполняется при **завершении** работы приложения.
   - Здесь можно закрыть подключения, остановить фоновые задачи, освободить память и т.д.

---

### **Примеры обработки ресурсов**

#### 1. **Подключение к базе данных**
Инициализация и закрытие подключения к базе данных с использованием SQLAlchemy.

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Создаем подключение к базе данных
DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Lifespan менеджер
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Инициализация ресурсов при запуске
    print("Starting up...")
    app.state.db_engine = engine
    app.state.db_session = SessionLocal
    yield

    print("Shutting down...")
    app.state.db_engine.dispose()

app = FastAPI(lifespan=lifespan)

@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```

- **Что происходит**:
  - При запуске приложения создается подключение к базе данных и сохраняется в `app.state`.
  - При завершении работы подключение закрывается с помощью `engine.dispose()`.

---

#### 2. **Кэш в памяти**
Инициализация и очистка кэша в памяти.

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

# Lifespan менеджер
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Инициализация кэша при запуске
    print("Starting up...")
    app.state.cache = {}
    yield
    # Очистка кэша при завершении
    print("Shutting down...")
    app.state.cache.clear()

app = FastAPI(lifespan=lifespan)

@app.get("/cache/")
async def get_cache():
    return {"cache": app.state.cache}
```

- **Что происходит**:
  - При запуске приложения создается кэш в памяти.
  - При завершении работы кэш очищается.

---

#### 3. **Фоновые задачи**
Запуск и остановка фоновых задач.

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
import asyncio

# Lifespan менеджер
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Запуск фоновой задачи при запуске
    print("Starting up...")
    task = asyncio.create_task(background_task())
    yield
    # Остановка фоновой задачи при завершении
    print("Shutting down...")
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("Background task cancelled")

async def background_task():
    while True:
        print("Background task is running...")
        await asyncio.sleep(5)

app = FastAPI(lifespan=lifespan)

@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```

- **Что происходит**:
  - При запуске приложения создается фоновая задача, которая выполняется каждые 5 секунд.
  - При завершении работы задача отменяется.

---

#### 4. **Подключение к внешнему сервису (например, Redis)**
Инициализация и закрытие подключения к Redis.

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
import redis

# Lifespan менеджер
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Инициализация подключения к Redis при запуске
    print("Starting up...")
    app.state.redis = redis.Redis(host="localhost", port=6379, db=0)
    yield
    # Закрытие подключения к Redis при завершении
    print("Shutting down...")
    app.state.redis.close()

app = FastAPI(lifespan=lifespan)

@app.get("/redis/")
async def get_redis(key: str):
    value = app.state.redis.get(key)
    return {"key": key, "value": value}
```

- **Что происходит**:
  - При запуске приложения создается подключение к Redis.
  - При завершении работы подключение закрывается.

---

### **Преимущества использования lifespan для управления ресурсами**
1. **Централизованное управление**:
   - Весь код, связанный с инициализацией и освобождением ресурсов, находится в одном месте.
2. **Гибкость**:
   - Можно передавать данные между кодом запуска и завершения.
3. **Надежность**:
   - Ресурсы корректно освобождаются при завершении работы приложения.

---

### **Итог**
- **Lifespan** — это мощный инструмент для управления ресурсами в FastAPI.
- Используй код до `yield` для инициализации ресурсов.
- Используй код после `yield` для освобождения ресурсов.

Пример с подключением к базе данных:

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@asynccontextmanager
async def lifespan(app: FastAPI):
    print("Starting up...")
    app.state.db_engine = engine
    app.state.db_session = SessionLocal
    yield
    print("Shutting down...")
    app.state.db_engine.dispose()

app = FastAPI(lifespan=lifespan)

@app.get("/")
async def root():
    return {"message": "Hello, World!"}
```

