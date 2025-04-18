# Простой пример

## Общие сведения

Пример содержит три сборочных варианта (СО) для трёх аппаратных платформ, являющихся отладочными платами **Xilinx**:

* 7A35T.
* 7A50T.
* AC701.

Все три СО одинаковы за исключением настроек, касающихся целевых плат&nbsp;– констрейны размещения, параметры тактирования&nbsp;– значение опорной тактовой частоты, дифференциальное подключение тактового генератора или нет, и т.д.

Исходный код примера доступен по [по ссылке](https://github.com/fpga-lib/vivado-boilerplate).

<br>
!!! tip "СОВЕТ"

    Данный пример демонстрирует возможности системы сборки непосредственным образом и может использоваться как шаблон для начала работы с рабочими проектами. Однако существует более [развитый подход к организации сборочного процесса](Advanced-Example.md), и именно он рекомендуется как отправная точка. 

## Использование

Запуск на сборку осуществляется с помощью команды `scons [options] variant=<variant-name> [target]` из корневой директории проекта (там, где расположен файл `SConstruct`) или из любого места дерева проекта при указанной опции `-D`.

### Примеры команд

##### Показать справку по целям

```
scons -s -D variant=ac701 -h
```

##### Создать проект Vivado

```
scons -s -D variant=ac701 prj
```

Будет создан проект **Vivado**, предварительно сгенерированы IP ядра с последующим добавлением в проект.

##### Синтезировать проект для варианта 7A50T

```
scons -s -D variant=7a50t prjsyn
```

##### Открыть проект для варианта 7A35T

```
scons -s -D bv=7a35t prjopen
```

Здесь попутно будут синтезированы IP ядра, т.к. после открытия проекта в **Vivad** можно сразу приступать к синтезу непосредственно проекта. Для выбора варианта тут используется аргумент `bv` вместо `variant`, они являются синонимами.

#####  Компилировать рабочую библиотеку симулятора (цель по умолчанию)

```
scons -s -D bv=7a35t
```

Цель по умолчанию `wlib`, её указание можно опустить. По умолчанию выбрана эта цель, т.к. её запуск является самым частым&nbsp;– это быстрый и удобный способ проверить исходный HDL код на синтаксическую правильность, а так же компиляция рабочей библиотеки требуется при отладке в симуляторе.
