## Обобщённые типы данных

Мы можем использовать обобщённые типы данных для функций или структур, которые затем можно использовать с различными конкретными типами данных. Давайте сначала посмотрим, как объявлять функции, структуры, перечисления и методы, используя обобщённые типы данных. Затем мы обсудим, как обобщённые типы данных влияют на производительность кода.

### В объявлении функций

Когда мы объявляем функцию с обобщёнными типами, мы размещаем обобщённые типы в сигнатуре функции, там где мы обычно указываем типы данных аргументов и возвращаемое значение. Такое использование делает код более гибким и предоставляет большую функциональность при вызове нашей функции, также предотвращая дублирование кода.

Рассмотрим пример с функцией `largest`. Листинг 10-4 показывает две функции, каждая из которых находит самое большое значение в срезе.

<span class="filename">Файл: src/main.rs</span>

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 100);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
#    assert_eq!(result, 'y');
}
```

<span class="caption">Listing 10-4: Две функции, отличия которых только в имени и типе обрабатываемых данных</span>

Функцию `largest_i32` мы уже извлекли в листинге 10-3, где она находит наибольшее значение типа `i32` в срезе. Функция `largest_char` находит самое большое значение типа `char` в срезе. Тело у этих функций одинаковое, так что давайте избавимся от дублируемого кода, добавив обобщённые типы данных.

Для параметризации типов данных в новой объявляемой функции, нам нужно дать имя обобщённому типу, также как мы это делаем для аргументов функций. Можно использовать любой идентификатор для имени параметра типа. Но мы будем использовать `T`, потому что, по соглашению, имена параметров в Rust должны быть короткими, часто длиной в один символ, и Rust именование типов делается в нотации CamelCase. Сокращение слова “type,” `T` является стандартным выбором большинства программистов на языке Rust.

Когда мы используем параметр в теле функции, мы должны объявить имя параметра в сигнатуре, так компилятор будет знать, что имя означает. Аналогично, когда мы используем имя параметра в сигнатуре функции, мы должны объявить имя параметра раньше, чем мы его используем. Чтобы определить обобщённую функцию `largest`, поместим объявление имён параметров в треугольные скобки, `<>`, между именем функции и списком параметров, как здесь:

```rust,ignore
fn largest<T>(list: &[T]) -> T {
```

Объявление читается так: функция `largest` является обобщённой с типом `T`. Эта функция имеет один параметр с именем `list`, который является срезом значений с типом данных `T`. Функция `largest` возвращает данные такого же типа `T`.

Листинг 10-5 показывает определение функции `largest` с использованием обобщённых типов данных в её сигнатуре. Листинг также показывает, как мы можем вызвать функцию со срезом данных типа `i32` или `char`. Данный код пока не будет компилироваться, но мы исправим это к концу раздела.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

<span class="caption">Листинг 10-5: Определение функции <code>largest</code> с использованием обобщённых типов, но он пока не компилируется</span>

Если мы скомпилируем программу сейчас, мы получим следующую ошибку:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:5:12
  |
5 |         if item > largest {
  |            ^^^^^^^^^^^^^^
  |
  = note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

В подсказке упоминается `std::cmp::PartialOrd`, который является *типажом*. Мы поговорим про типажи в следующей секции. Сейчас, ошибка в функции `largest` указывает, что функция не будет работать для всех возможных типов `T`. Так как мы хотим сравнивать значения типа `T` в теле функции, то можно использовать только те типы, данные которых можно упорядочить. Для возможности сравнения, стандартная библиотека имеет типаж `std::cmp::PartialOrd`, который вы можете реализовать для типов (смотрите дополнение С для большей информации про данных типаж). Вы узнаете, как указать, какой обобщённый тип имеет отдельный типаж в секции [“Типажи как параметры”]<comment>, но сначала давайте рассмотрим другие варианты использования обобщённых типов.</comment>

### В определении структур

Также можно определять структуры с использованием обобщённых типов в одном или нескольких полях структуры с помощью синтаксиса `<>`. Листинг 10-6 показывает как определить структуру `Point<T>`, чтобы хранить поля координат `x` и `y` любого типа данных.

<span class="filename">Файл: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

<span class="caption">Листинг 10-6: Структура <code>Point</code> содержащая поля <code>x</code> и <code>y</code> типа <code>T</code></span>

Синтаксис использования обобщённых типов в определении структуры такой же как и в определении функции. Сначала мы объявляем имена параметров внутри треугольных скобок сразу после имени структуры. Затем мы можем использовать обобщённые типы в определении структуры, где иначе мы бы указывали конкретные типы.

Так как мы используем только один обобщённый тип данных для определения структуры `Point<T>`, это определение означает, что структура `Point<T>` является обобщённой с типом `T`, и *оба* поля `x` и `y` имеют одинаковый тип, каким бы ни был тип. Если мы создадим экземпляр структуры `Point<T>` со значениями разных типов, как показано в Листинге 10-7, наш код не компилируется.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Листинг 10-7: Поля <code>x</code> и <code>y</code> должны быть одного типа, так как они имеют один и тот же обобщённый тип <code>T</code></span>

В этом примере, когда мы присваиваем целочисленное значение 5 переменной `x` , мы позволяем компилятору знать, что обобщённый тип `T` будет целым числом для этого экземпляра `Point<T>`. Затем, когда мы указываем значение 4.0 для `y`, который мы определили имеющим тот же тип, что и `x`, мы получим ошибку несоответствия типов:

```text
error[E0308]: mismatched types
 --> src/main.rs:7:38
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integer, found
floating-point number
  |
  = note: expected type `{integer}`
             found type `{float}`
```

Чтобы определить структуру `Point` где оба `x` и `y` являются обобщёнными, но могут иметь различные типы, можно использовать несколько параметров обобщённого типа. Например, в листинге 10-8 мы можем изменить определение `Point`, чтобы оно было общим для типов `T` и `U` где `x` имеет тип `T` а `y` имеет тип `U`.

<span class="filename">Файл: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Листинг 10-8. Структура <code>Point<T, U></code> обобщена для двух типов, так что <code>x</code> и <code>y</code> могут быть значениями разных типов</span>

Теперь разрешены все показанные экземпляры типа `Point`! В объявлении можно использовать столько много обобщённых параметров типа, сколько хочется, но использование более чем несколько типов делает код трудно читаемым. Когда вам нужно много обобщённых типов в коде, это может указывать на то, что ваш код нуждается в реструктуризации на более мелкие части.

### В определениях перечислений

Как и в случае со структурами, можно определить перечисления для хранения обобщённых типов в их вариантах. Давайте ещё раз посмотрим на перечисление `Option<T>` предоставленное стандартной библиотекой, которое мы использовали в главе 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Это определение теперь должно иметь больше смысла. Как видите,  перечисление `Option<T>`, которое является общим для типа `T` и имеет два варианта: `Some`, который содержит одно значение типа `T` и вариант `None`, который не содержит никакого значения. Используя перечисление `Option<T>`, можно выразить абстрактную концепцию необязательного значения и так как `Option<T>` является обобщённым, можно использовать эту абстракцию независимо от того, каким будет тип для необязательного значения.

Перечисления также могут использовать в определении несколько обобщённых типов. Определение перечисления `Result`, которое мы использовали в главе 9, является таким примером:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Перечисление `Result` имеет два обобщённых типа `T` и `E` и два варианта:  `Ok`, которое содержит тип `T` и `Err`, которое содержит тип `E`. Такое определение позволяет использовать перечисление `Result` везде, где операции могут быть выполнены успешно (возвращая значение типа данных `T`) или не успешно (возвращая значение типа данных `E`). Это то что мы делали в коде листинга 9-2, где при открытии файла заполнялись данные типа `T`, в примере тип  `std::fs::File` или `E` тип  `std::io::Error` при ошибке, при каких-либо проблемах открытия файла.

Когда вы в коде распознаете ситуации с несколькими структурами или определениями перечислений, которые отличаются только типами содержащих значений, вы можете избежать дублирования, используя обобщённые типы.

### В определении методов

Также, как и в Главе 5, можно реализовать методы структур и перечислений с помощью обобщённых типов и их объявлений. Код листинга 10-9 демонстрирует пример добавления метода с названием `x` в структуру `Point<T>`, которые мы ранее описал в листинге 10-6.

<span class="filename">Файл: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

<span class="caption">Листинг 10-9. Реализация метода с именем <code>x</code> у структуры <code>Point<T></code>, которая будет возвращать ссылку на поле <code>x</code> типа <code>T</code></span>

Здесь мы определили метод с именем `x` у `Point<T>` который возвращает ссылку на данные в поле `x`.

Обратите внимание, что нужно объявить `T` сразу после `impl`, чтобы можно было использовать его для указания, что мы реализуем методы для типа `Point<T>`. Объявляя `T` как обобщённый тип после `impl`, Rust может определить, что тип в угловых скобках у `Point` - это обобщённый, а не конкретный тип.

Мы могли бы, например, реализовать методы только для экземпляров типа `Point<f32>` вместо экземпляров `Point<T>` с любым обобщённым типом. В листинге 10-10 мы используем конкретный тип `f32`, что означает мы не объявляем никаких типов после `impl`.

```rust
# struct Point<T> {
#     x: T,
#     y: T,
# }
#
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

<span class="caption">Листинг 10-10. Блок <code>impl</code> который применяется только к структуре с конкретным типом для параметра обобщённого типа <code>T</code></span>

Этот код означает, что тип `Point<f32>` будет иметь метод с именем `distance_from_origin` и другие экземпляры `Point<T>` где `T` не имеет тип `f32` не будет определять этот метод. Метод измеряет, насколько далеко наша точка находится от точки с координатами (0,0, 0,0) и использует математические операции, доступные только для типов с плавающей запятой.

Обобщённые типы параметров в определении структур являются не всегда такими же, как используемые в сигнатурах методов. Код листинга 10-11 описывает метод `mixup` у структуры `Point<T, U>`. Метод получает другую структуру `Point` в качестве параметра, которая могла бы иметь типы отличные от `self` для `Point` у который мы вызываем метод `mixup`. Метод создаёт новый экземпляр структуры `Point`, который получает значение `x` из `self` структуры `Point` (типа `T`) и `y` из `Point` (типа `W`):

<span class="filename">Файл: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

<span class="caption">Листинг 10-11. Метод, использующий разные обобщённые типы из определения его структур</span>

В функции `main`, мы определили экземпляр переменной p1 типа `Point`, который имеет `i32` для `x` (со значением `5` ) и `f64` для `y` (со значением `10.4` ). Переменная `p2` является структурой `Point` который имеет строковый срез для `x` (со значением `"Hello"`) и `char` для `y` (со значением `'c'` ). Вызов `mixup` на `p1` с аргументом `p2` даёт нам `p3`, который будет иметь `i32` для `x`, потому что `x` пришёл из `p1`. Переменная `p3` будет иметь `char` для `y`, потому что `y` пришёл из `p2`. Вызов макроса `println! ` выведет `p3.x = 5, p3.y = c`.

Цель этого примера продемонстрировать ситуацию, в которой некоторые обобщённые параметры объявлены с помощью `impl`, а некоторые объявлены в определении метода. Здесь обобщённые параметры `T` и `U` объявляются после `impl`, потому что они идут вместе с определением структуры. Обобщённые параметры типа `V` и `W` объявляются после `fn mixup`, потому что они относятся только к методу.

### Производительность кода с использованием обобщений

Вы могли бы задаться вопросом, есть ли стоимость времени выполнения при использовании параметров обобщённого типа. Хорошей новостью является то, что Rust реализует обобщённые типы таким способом, что ваш код не работает медленнее при их использовании, чем если бы это было с конкретными типами.

Rust достигает этого, выполняя мономорфизацию кода использующего обобщения во время компиляции. *Мономорфизация* - это процесс превращения обобщённого кода в конкретный код, заполняя конкретные типы используемые при компиляции.

В этом процессе компилятор выполняет противоположные шаги, которые обычно используются для создания обобщённой функции в листинге 10-5: компилятор просматривает все места, где вызывается обобщённый код и генерирует код для конкретных типов, с которыми вызван обобщённый код.

Давайте посмотрим, как это работает, на примере, который использует перечисление `Option<T>` из стандартной библиотеки:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Когда Rust компилирует этот код, он выполняет мономорфизацию. Во время этого процесса компилятор считывает значения, которые были использованы у экземпляра `Option<T>` и определяет два вида `Option<T>`: один `i32`, а другой это `f64`. Таким образом, он расширяет общее определение `Option<T>` в `Option_i32` и `Option_f64`, тем самым заменяя обобщённое определение на конкретное.

Мономорфизированная версия кода выглядит следующим образом. Обобщённый `Option<T>` заменяется конкретными определениями, созданными компилятором:
<span class="filename">Файл: src/main.rs</span>

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

Так как Rust компилирует обобщённый код, в код указывающий тип в каждом экземпляре, то мы не платим временем выполнения за использование обобщённых типов. Когда код выполняется, он работает так же, как если бы мы дублировали каждое определение вручную. Процесс мономорфизации делает обобщённые типы Rust чрезвычайно эффективными во время выполнения.


[“Типажи как параметры”]: ch10-02-traits.html#traits-as-parameters