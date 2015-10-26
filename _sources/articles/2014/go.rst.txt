.. post:: 06 Apr, 2014 0:58
   :tags: Go, vim
   :category: Go
   :author: Uralbash
   :language: ru

Пишем на Go (Golang)
====================

.. image:: /_static/2014/go.png
   :align: left

Go замечательный язык программирования, который можно компилировать,
компилировать под разные платформы (``ARM``, ``x86``), распараллеливать. Он
проще ``C/C++`` и уже сформировавшийся язык в отличии от ``Rust`` который
ломает программы с каждым обновлением. Область применения самая разная начиная
от консольных утилит, всяких парсеров, системного, сетевого ПО, связи с
физическими устройствами и заканчивая веб приложениями, :strike:`разве что пока нету
реализаций под смартфоны` (`android
<https://docs.google.com/document/d/1N3XyVkAP8nmWjASz8L_OjjnjVKxgeVBjIsTr5qIUcA4/edit>`_).

.. raw:: html

   <br clear="both"/>

На Go уже написаны:

* :l:`Docker` - система легковесных контейнеров (переписан с :l:`python`)
*  http://drone.io - система непрерывного тестирования, поддерживающая
   :l:`Docker` (о ней я напишу отдельно скорее всего) и огромное количество
   других проектов, что удивительно ведь :l:`go` очень молодой проект, а ещё гугл

Здесь я опишу процесс одной из возможных реализаций установки :l:`Go` под
:l:`Linux`.

УСТАНОВКА
---------

.. seealso::

   Более простой способ :github:`moovweb/gvm`

| описано подробно в оф. документации http://golang.org/doc/install
| `Скачать архив <http://code.google.com/p/go/downloads/list?q=OpSys-FreeBSD+OR+OpSys-Linux+OR+OpSys-OSX+Type-Archive>`_ и распаковать в ``~/golang`` например:

.. raw:: html

   <br>

.. code-block:: bash

   $ tar -C ~/golang -xzf go1.2.1.linux-amd64.tar.gz

Добавляем в ``~/.bashrc``:

.. code-block:: bash
   :caption: ~/.bashrc

   # Go lang
   export GOROOT=$HOME/golang
   export PATH=$PATH:$GOROOT/bin

создаем файл ``hello.go``:

.. code-block:: go
   :caption: hello.go

   package main

   import "fmt"

   func main() {
       fmt.Printf("hello, world\n")
   }

и выполняем так:

.. code-block:: bash

   $ go run hello.go
   hello, world

или компилируем:

.. code-block:: bash

   $ go build hello.go
   $ ./hello
   hello, world

теперь у нас установлен :l:`go` в системе. Если вам нужно свои пакеты хранить в
другой директории (не ``$HOME/golang``), но при этом что бы они находились в
общем окружении, то можно задать переменную окружения ``GOPATH``. У меня всё
вместе выглядит так:

.. code-block:: bash

   # Go lang
   export GOPATH=$HOME/Projects/go
   export GOROOT=$HOME/golang
   export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

VIM
---

есть много дополнений для go но самая крутое из них это https://github.com/fatih/vim-go
пример ``.vimrc``:

.. code-block:: vim

   au BufNewFile,BufRead *.go set ft=go nu
   au FileType go map <leader>r :!go run %<CR>
   Bundle "fatih/vim-go"

``vim-go`` умеет:

* подсветка синтаксиса
* автодополнение кода gocode
* автоформатирование кода через gofmt
* автоимпорт недостающих библиотек goimports
* проверять код на наличие ошибок с помощью golint
* поддерживает снипеты ultisnips or neosnippet
* и многое другое

.. raw:: html

   <iframe width="420" height="315"
   src="https://www.youtube.com/embed/rD11pEx5h8c" frameborder="0"
   allowfullscreen></iframe>

КНИГИ
-----

книг много, но на великом только одна от издательства ДМК пресс, к счастью есть
эл.вариант книги хоть и в pdf.

.. image:: /_static/2014/go_book.jpg
   :align: center
   :target: http://dmkpress.com/catalog/computer/programming/978-5-94074-854-0/

Ресурсы
-------

| http://4gophers.com/ (ru)
| #go-nuts in the irc.freenode.org network.
| http://blog.golang.org/
| google group
| reddit

AnyKey
------

| Использование Си в Гоу
| http://zacg.github.io/blog/2013/06/06/calling-c-plus-plus-code-from-go-with-swig/
|
| ORM
| https://github.com/eaigner/hood
| https://github.com/jinzhu/gorm
|
| Web
| Revel
| Gorilla
| net/http
|
| Список пакетов
| https://code.google.com/p/go-wiki/wiki/Projects
|
| Go для программистов C++
| http://netsago.org/ru/docs/1/16/
| http://eao197.narod.ru/desc/short_effective_go.html
|
|
| Calling Go from Python via JSON-RPC
| www.artima.com/weblogs/viewpost.jsp?thread=333589
|
| Go для Python программистов
| http://www.slideshare.net/kcherkasoff/go-for-pythonistas#
|
| Тесты
| http://smartystreets.github.io/goconvey/
|
| Примеры
| https://gobyexample.com/
|
| CI
| https://github.com/drone/drone
|
| Хайлод++
| http://www.youtube.com/watch?v=bqtN6XViejE
|
| Туториал
| http://tour.golang.org/
|

.. image:: /_static/2014/gopher.png
   :align: center
