# Коллекции стандартной библиотеки

Стандартная библиотека содержит полезные структуры данных, которые называются
*коллекциями*. Другие типы данных представляют собой хранение какого-то одного типа
данных. Особенностью коллекций является хранение множества однотипных данных.
В отличии от массива и кортежа, данных хранятся в куче, а это значит, что размер
коллекций может быть неизвестен в момент компиляции программы. Она может изменяться
(увеличиваться, уменьшаться) во время работы программы. Каждый вид коллекции имеет
свои особенности и ограничения производительности. Выбор конкретной коллекции зависит
от целей, которые необходимо решить. В этой главе будет рассмотрено несколько
коллекций:

* *Вектор* позволяет нам сохранять данные последовательно.
* *Строка* - последовательность символов. В этой главе мы поговорим о этом типе данных подробнее.
* *Хеш таблица* позволяет сопоставлять значение ключу. Это реализация структуры *map*.

Для того, чтобы узнать о других видах коллекций, пожалуйста, перейдите на страницу
документации [the documentation][collections].

[collections]: https://doc.rust-lang.org/std/collections/index.html

Мы начинаем свой рассказ с того, как создать и обновить вектора, строки и хэш таблицы.