# Вспомогательные и служебные средства

## Модуль utils
***
#### `pexec(cmd, wdir = os.curdir)`

Process execute. Функция осуществляет запуск внешнего процесса, производит вычитывание его выходного буфера и печатает полученное в консоль. По окончании работы процесса функция возвращает код завершения, полученный от процесса.

***
#### `print_info(text)`, `print_action(text)`, `print_error(text)`, `print_success(text)`

Группа функций, предназначенная для печати информационных сообщений. Выводят на экран сообщения в цветном виде, каждая своим цветом.

Name | Color
-----|------
info | `LIGHTCYAN`
action | `LIGHTGREEN`
error | `LIGHTRED`
success | `GREEN`

***
#### `colorize(text, color)`

Раскрашивает строку в указанный цвет. Поддерживаются цвета: `red`, `green`, `yelllow`.

***
#### `clog2(n: int) -> int`

Функция возвращает результат `ceil(log2(int)`. Функциональный аналог системной функции `$clog2()` языка `SystemVerilog`.

***

#### `max_str_len(x)`

Функция получает список строк и возвращает длину самой длинной. Используется как правило при формировании скриптов для одинакового выравнивания правых частей выражений.

***
#### `search_file(fn, search_root='')`

Осуществляет рекурсивный поиск файла по имени, начиная с пути `search_root` (если не указан, то с текущего). Возвращает полный (абсолютный) путь файла. Используется главным образом для поиска конфигурационных файлов внутри сборочных вариантов.

***
#### `class ConfigDict(object)`

Принимает словарь, конструирует объект класса с членами, имена которых соответствуют ключам словаря, а значения — соответствующим значениям ключей. Позволяет получать объекты с нотацией доступа `<name.member>` на уровне скрипта сценариев сборки. Например:

```python
...
cfg  = import_config('main.yml')  # import_config  возвращает класс типа ConfigDict
...
envx['TOP_NAME']       = cfg.TOP_NAME
envx['TESTBENCH_NAME'] = cfg.TESTBENCH_NAME
...
```

***
#### `read_config(fn: str, param_sect='parameters', search_root='')`

Читает секцию конфигурационного файла (по умолчанию секцию `parameters`), возвращает словарь, содержащий имена и значения параметров, описанных в указанной секции конфигурационного файла.

***
#### `import_config(fn: str)`

Аналог функции `read_config()`, возвращающая объект класса `Dict2Class`.

***
#### `read_ip_config(fn, param_sect, search_root='')`

Функция читает конфигурационный файл, описывающий свойства IP ядра, возвращает словарь, содержащий тип IP ядра и его конфигурационные параметры.

***
#### `read_src_list(fn: str, search_root='')`

Рекурсивно ищет конфигурационный файл со списком файлов (как правило это исходные файлы разных типов — HDL, констрейны и т.д.), читает его, возвращает список файлов, описанный в секции `sources`. Является вспомогательной.

***
#### `read_sources(*args)`

Функция использует `read_src_list()` для получения списка файлов, преобразует пути файлов в абсолютные, возвращает полученный список. Поддерживаются два контекста вызова:

1. с одним аргументом;
1. с двумя аргументами.

При вызове с одним аргументом функции передаётся имя конфигурационного файла (со списком файлов). Функция в этом случае возвращает список абсолютных путей файлов, перечисленных в конфигурационном файле.

При вызове с двумя аргументами первым аргументом передаётся имя конфигурационного файла (как и в предыдущем варианте), а вторым ­­— путь, от которого осуществляется поиск конфигурационного файла. Функция возвращает два объекта:

1. список абсолютных путей файлов (как и в предыдущем варианте);
1. значение опции `usedin`, которое извлекается из конфигурационного файла, если определено в нём, а если не определено, то по умолчанию равно `syn`. Эта опция служит для обозначения списка файлов, которые предназначены для специального использования в проекте **Vivado** — например, для симулятора.

Вариант с двумя аргументами предназначен для использования на этапе запуска билдеров. Все действия на этом этапе выполняются от корневой директории проекта, поэтому для локализации поиска конфигурационных файлов списков необходимо указывать путь, от которого необходимо осуществлять поиск — как правило, это корневая директория текущего сборочного варианта. Именно такой вызов используется в билдере создания проекта **Vivado**.

***
#### `get_dirs(flist)`

Возвращает список директорий из списка файлов. Такой список директорий нужен, как правило, для организации поиска включаемых файлов, которые могут находиться в тех же директориях, что и сами исходные файлы. По сути это простой способ организовать поиск включаемых файлов во всех директориях с исходными файлами.

***
#### `get_name(path)`

Принимает абсолютный или относительный путь, возвращает имя файла (без расширения).

***
#### `create_dirs(dirs)`

Принимает список путей директорий, создаёт указанные директории.

***