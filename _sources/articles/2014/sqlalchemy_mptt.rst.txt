.. post:: 12 May, 2014 18:30
   :tags: Python, SQLAlchemy, MPTT
   :category: Python
   :author: Uralbash
   :language: ru

MPTT для SQLAlchemy
===================

Запилил я тут для своих нужд небольшое приложение (:pypi:`sqlalchemy_mptt`)
которое добавляет в модель поля и функционал необходимый для ``Nested sets``.
По аналогии с :pypi:`django-mptt`. Грубо говоря в модель добавляются поля
``left`` и ``right`` которые при помощи системы эвентов самостоятельно
пересчитываются при изменении дерева. Ниже пример обхода дерева:

.. image:: /_static/2014/2_sqlalchemy_mptt_traversal.png
   :align: center
   :width: 600px

Простой пример:

.. code-block:: python

   from sqlalchemy import Column, Integer, Boolean
   from sqlalchemy.ext.declarative import declarative_base

   from sqlalchemy_mptt.mixins import BaseNestedSets

   Base = declarative_base()


   class Tree(Base, BaseNestedSets):
       __tablename__ = "tree"

       id = Column(Integer, primary_key=True)
       visible = Column(Boolean)  # Наше кастомное поле

       def __repr__(self):
           return "<node s="">" % self.id

   Tree.register_tree()  # Регистрирует event'ы

Добавление
----------

.. code-block:: python

   node = Tree(parent_id=6)
   session.add(node)

получим:

.. code-block:: bash

        level           Nested sets example
        1                    1(1)22
                _______________|___________________
               |               |                   |
        2    2(2)5           6(4)11             12(7)21
               |               ^                   ^
        3    3(3)4       7(5)8   9(6)10    13(8)16   17(10)20
                                              |          |
        4                                  14(9)15   18(11)19

        level     Insert node with parent_id == 6
        1                    1(1)24
                _______________|_________________
               |               |                 |
        2    2(2)5           6(4)13           14(7)23
               |           ____|____          ___|____
               |          |         |        |        |
        3    3(3)4      7(5)8    9(6)12  15(8)18   19(10)22
                                   |        |         |
        4                      10(23)11  16(9)17  20(11)21

Удаление
--------

.. code-block:: python

   node = session.query(Tree).filter(Tree.id == 4).one()
   session.delete(node)

получим:

.. code-block:: bash

        level           Nested sets example
        1                    1(1)22
                _______________|___________________
               |               |                   |
        2    2(2)5           6(4)11             12(7)21
               |               ^                   ^
        3    3(3)4       7(5)8   9(6)10    13(8)16   17(10)20
                                              |          |
        4                                  14(9)15   18(11)19

        level         Delete node == 4
        1                    1(1)16
                _______________|_____
               |                     |
        2    2(2)5                 6(7)15
               |                     ^
        3    3(3)4            7(8)10   11(10)14
                                |          |
        4                     8(9)9    12(11)13

Обновление
----------

.. code-block:: python

   node = session.query(Tree).filter(Tree.id == 8).one()
   node.parent_id = 5
   session.add(node)

получим

.. code-block:: bash

        level           Nested sets example
            1                    1(1)22
                    _______________|___________________
                   |               |                   |
            2    2(2)5           6(4)11             12(7)21
                   |               ^                   ^
            3    3(3)4       7(5)8   9(6)10    13(8)16   17(10)20
                                                  |          |
            4                                  14(9)15   18(11)19

        level               Move 8 - > 5
            1                     1(1)22
                     _______________|__________________
                    |               |                  |
            2     2(2)5           6(4)15            16(7)21
                    |               ^                  |
            3     3(3)4      7(5)12   13(6)14      17(10)20
                               |                        |
            4                8(8)11                18(11)19
                               |
            5                9(9)10

Перенос ноды по дереву или между деревьев
-----------------------------------------


.. image:: /_static/2014/3_sqlalchemy_mptt_multitree.png
   :align: center
   :width: 600px

Move inside (перемещает ноду на первое место от родителя)
---------------------------------------------------------

.. code-block:: python

   node = session.query(Tree).filter(Tree.id == 8).one()
   node = session.query(Tree).filter(Tree.id == 4).one()
   node.move_inside("15")

получим:

.. code-block:: bash

                 4 -> 15
        level           Nested sets tree1
        1                    1(1)16
                _______________|_____________________
               |                                     |
        2    2(2)5                                 6(7)15
               |                                     ^
        3    3(3)4                            7(8)10   11(10)14
                                                |          |
        4                                     8(9)9    12(11)13

        level           Nested sets tree2
        1                     1(12)28
                 ________________|_______________________
                |                |                       |
        2    2(13)5            6(15)17                18(18)27
               |                 ^                        ^
        3    3(14)4    7(4)12 13(16)14  15(17)16  19(19)22  23(21)26
                         ^                            |         |
        4          8(5)9  10(6)11                 20(20)21  24(22)25

Move after (перемещает ноду после текущей ноды)
-----------------------------------------------

.. code-block:: python

   node = session.query(Tree).filter(Tree.id == 8).one()
   node = session.query(Tree).filter(Tree.id == 8).one()
   node.move_after("5")

получим:

.. code-block:: bash

       level           Nested sets example
            1                    1(1)22
                    _______________|___________________
                   |               |                   |
            2    2(2)5           6(4)11             12(7)21
                   |               ^                   ^
            3    3(3)4       7(5)8   9(6)10    13(8)16   17(10)20
                                                  |          |
            4                                  14(9)15   18(11)19

        level               Move 8 after 5
            1                     1(1)22
                     _______________|__________________
                    |               |                  |
            2     2(2)5           6(4)15            16(7)21
                    |               ^                  |
            3     3(3)4    7(5)8  9(8)12  13(6)14   17(10)20
                                    |                  |
            4                    10(9)11            18(11)19

Move to top level (выделение ноды в самостоятельное дерево)
-----------------------------------------------------------

.. code-block:: python

   node = session.query(Tree).filter(Tree.id == 8).one()
   node = session.query(Tree).filter(Tree.id == 15).one()
   node.move_after("1")

получим:

.. code-block:: bash

        level           tree_id = 1
        1                    1(1)22
                _______________|___________________
               |               |                   |
        2    2(2)5           6(4)11             12(7)21
               |               ^                   ^
        3    3(3)4       7(5)8   9(6)10    13(8)16   17(10)20
                                              |          |
        4                                  14(9)15   18(11)19

        level           tree_id = 2
        1                     1(15)6
                                 ^
        2                 2(16)3   4(17)5

        level           tree_id = 3
        1                    1(12)16
                 _______________|
                |               |
        2    2(13)5          6(18)15
                |               ^
        3    3(14)4     7(19)10   11(21)14
                           |          |
        4               8(20)9    12(22)13


За основу был взят пример ``Mike Bayer``, впринципе в тестах можно посмотреть
больше примеров. Ссылка на github: https://github.com/ITCase/sqlalchemy_mptt
