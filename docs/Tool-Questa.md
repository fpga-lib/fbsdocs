# Tool 'questa'

## Environment variables

Пользовательский скрипт должен предоставить следующие внешние переменные сборочного окружения, необходимые для работы инструмента:

Name                 | Description                            | Default value
---------------------|----------------------------------------|---------------
`MENTOR`               | Путь до директории с устано-<br>вленными продуктами Mentor | `os.environ['MENTOR']` — <br>переменная окружения должна<br> быть определена на используе-<br>мом хосте
`MGLS_LICENSE_FILE`    | Путь до файла лицензии <br>симулятора      | `os.path.join(MENTOR, 'license.dat')`
`QUESTABASE`           | Путь до коренной директории <br>используемой версии симулятора |`os.path.join(MENTOR, QUESTA_TOOL_VERSION, 'questasim')`
`QUESTABIN`            | Путь до директории с исполня-<br>емыми файлами симулятора <br>(`vlog`, `vlib`, `vmap`) | `os.path.join(QUESTABASE, 'bin')`
`QUESTASIM`            | Команда (скрипт) для запуска <br>симулятора в графическом режи-<br>ме         | `os.path.join(QUESTABASE, 'linux_x86_64', 'vsim')`
`VENDOR_LIB_PATH`      | Путь до директории, содержащей <br>скомплированные библиотеки<br> производителя ПЛИС | `os.path.join( os.path.dirname(env['QUESTASIM']), 'vendor', 'xlib', 'func')`

Переменные `QUESTABIN`, `QUESTASIM` проверяются при запуске, и если любая из них не определена, сборочная система завершает работу с ошибкой.  Переменная `VENDOR_LIB_PATH` так же проверяется на наличие, и если не определена, то ей присваивается значение по умолчанию.

Определить переменные удобно в сборочном скрипте, например:

```python
#-------------------------------------------------------------------------------
#
#    Environment
#
envx['ENV']['DISPLAY']            = os.environ['DISPLAY']
envx['ENV']['HOME']               = os.environ['HOME']
envx['ENV']['XILINX']             = env.XILINX
envx['ENV']['MENTOR']             = env.MENTOR
envx['ENV']['MGLS_LICENSE_FILE']  = env.MGLS_LICENSE_FILE
envx['ENV']['XILINX_VIVADO']      = env.XILINX_VIVADO
envx['XILINX_VIVADO']             = env.XILINX_VIVADO
envx['XILINX_HLS']                = env.XILINX_HLS
envx['QUESTABIN']                 = env.QUESTABIN
envx['QUESTASIM']                 = env.QUESTASIM
envx['VENDOR_LIB_PATH']           = env.VENDOR_LIB_PATH
```

где `env` — объект с параметрами, получаемый:

```python
env = import_config('env.yml')
```

а конфигурационный файл `env.yml` представляет собой нечто подобное:

```yaml
#
#   env.yml
#
parameters:
    XILINX              : = os.environ['XILINX']
    MENTOR              : = os.environ['MENTOR']

    XILINX_TOOL_VERSION : '2021.2'
    QUESTA_TOOL_VERSION : '2021.1'

    XILINX_VIVADO       : = os.path.join(XILINX, 'Vivado', XILINX_TOOL_VERSION)
    XILINX_HLS          : = os.path.join(XILINX, 'Vitis_HLS', XILINX_TOOL_VERSION)

    MGLS_LICENSE_FILE   : = os.path.join(MENTOR, 'license.dat')
    QUESTABASE          : = os.path.join(MENTOR, QUESTA_TOOL_VERSION, 'questasim')
    QUESTABIN           : = os.path.join(QUESTABASE, 'bin')
    QUESTASIM           : = os.path.join(QUESTABASE, 'linux_x86_64', 'vsim')

    VENDOR_LIB_NAME     : = 'xlib-vv' + XILINX_TOOL_VERSION + '-qs' + QUESTA_TOOL_VERSION
    VENDOR_LIB_PATH     : = os.path.join(MENTOR, 'vendor', VENDOR_LIB_NAME, 'func')
```

Директория, на которую указывает `VENDOR_LIB_PATH`, должна содержать следующий перечень библиотек:

```
.
├── secureip
├── simprims_ver
├── unifast
├── unifast_ver
├── unimacro
├── unimacro_ver
├── unisim
├── unisims_ver
├── xilinx_vip
└── xpmlib
```
Указанный перечень библиотек, кроме `xpmlib` может быть создан путём запуска в консоли **Vivado** команды:

```tcl
compile_simlib -force -language verilog -language vhdl -dir <path-to-target-dir> -simulator questa -simulator_exec_path <path-to-simulator-bin-dir> -library all -family all -no_ip_compile
```

`xpmlib` можно создать с помощью shell скрипта:

```bash
#!/bin/sh

MENTOR_QUESTA=<path-to-questa-bin-dir>
XILINX_VIVADO=<path-to-xilinx-home>/Vivado/<version-number>

$MENTOR_QUESTA/vlog -work xpmlib -64 -sv -O5 -mfcu \
$XILINX_VIVADO/data/ip/xpm/xpm_cdc/hdl/xpm_cdc.sv \
$XILINX_VIVADO/data/ip/xpm/xpm_memory/hdl/xpm_memory.sv \
$XILINX_VIVADO/data/ip/xpm/xpm_fifo/hdl/xpm_fifo.sv
```

Из описанных переменных окружения далее строятся все пути к исполняемым файлам внешних инструментов.

### Вспомогательный командный скрипт

Симулятор используется в non-project режиме и управляется из консоли (transcript window). Доступные действия:

* <u>Compile Work Library</u>, команда консоли `'c'`;
* <u>Start simulation</u>, команда консоли `'s [cfg]'`, где `cfg` — имя конфигурации, которая есть
  простой `do` скрипт (**Tcl**) с перечислением команд, которые требуется запустить после остановки
  прогона симулятора;
* <u>Restart simulation</u>, команда консоли `'r'` (`'rr'` со сбросом текущего лога transcript);
* <u>Show Simulation Results</u>, команда консоли `'show_res <cfg>'`, где `cfg`&nbsp;— имя
  конфигурации(см. выше).

Для поддержки этой функциональности система сборки содержит соответствующий файл `questa.tcl`. Этот же скрипт используется и для пакетного запуска, при этом симулятору передаётся соответствующая команда (`'c'`, `'s'`).

### Переменные общего назначения

Name                 | Description                            | Default value
---------------------|----------------------------------------|-----
`TESTBENCH_NAME`       | Имя модуля тестбенча |`'top_tb'`
`SIMLIB_NAME`          | Имя директории для <br>библиотек симуляционных мо-<br>делей IP ядер, блочных дизайнов<br> и т.п. | `'sim_lib'`
`SIMLIB_PATH`          | Путь до директории для библи-<br>отек симуляционных моделей IP <br>ядер, блочных дизайнов и т.п.| `os.path.join(env['BUILD_SYN_PATH'], env['SIMLIB_NAME'])`
`SIM_WORKLIB_NAME`     | Имя рабочей библиотеки | `'wlib'`
`SIM_INC_PATH`         | Список путей поиска заголовоч-<br>ных файлов при запуске симуля-<br>тора | `''`
`BUILD_SIM_PATH`       | Путь, по которому формируется<br> исполнительное окружение для<br> выполнения задач моделирова-<br>ния | `os.path.join(root_dir, 'build', os.path.basename(cfg_name), 'sim')`


### Внешние инструменты

Name                 | Description                             | Default value
---------------------|-----------------------------------------|-----
`VLOGCOM`              | компилятор языка <br>`Verilog/SystemVerilog`| `os.path.join(env['QUESTABIN'], 'vlog')`
`VCOMCOM`              | компилятор языка `VHDL`                 | `os.path.join(env['QUESTABIN'], 'vcom')`
`VLIBCOM`              | утилита создания библиотеки | `os.path.join(env['QUESTABIN'], 'vlib')`
`VMAPCOM`              | утилита, выполняющая отобра-<br>жение логического имени библи-<br>отеки на физический файл | `os.path.join(env['QUESTABIN'], 'vmap')`
`VSIMCOM`              | симулятор | `os.path.join(env['QUESTABIN'], 'vsim')`
`VERBOSE`              | управляет печатью команд при<br> запуске целей на сборку | `True`
`VLOG_FLAGS`           | опции компилятора языков<br> `Verilog/SystemVerilog` | `' -incr -sv -mfcu'`
`VCOM_FLAGS`           | опции компилятора языка `VHDL` | `' -64 -93'`
`VLOG_OPTIMIZATION`    | опции оптимизации компилятора <br>языков `Verilog/SystemVerilog` | `' -O5'`
`VOPT_FLAGS`   | опции `vopt` | `''`

В случае использования синтеза от **Xilinx** на этапе elaboration необходимо загружать файл глобальных сигналов:

```python
if 'vivado' in env['TOOLS']:
    env['VOPT_FLAGS'] = ' glbl'
```

## Builders

Перечислены [билдеры](/Build-Scripts#builders) и [псевдобилдеры](/Build-Scripts#pseudo-builders). Билдеры как правило не используются напрямую, т.к. они имеют вполне определённый интерфейс запуска, который далеко не всегда удобен, поэтому в скрипте сборочных сценариев используются как правило псевдобилдеры, которые по сути являются "обёртками" вокруг самих билдеров.

----

### <pre>SimLib</pre>
<u>True builder</u> 

Осуществляет создание симуляционной библиотеки, отображение логического имени на физический файл, компиляцию симуляционных моделей IP ядер, блочных дизайнов и т.п. из скриптов компиляции, созданных с помощью команды `export_simulation` САПР **Vivado**.

----

### <pre>CompileSimLib</pre>
<u>Pseudo-builder</u> 

Формирует имя целевой директории симуляционной библиотеки библиотеки. Запускает билдер `IpSimLib`, который выполняет основную работу.

Пример использования:

```python
IP_Cores   = envx.CreateIps(IP_Create_Scripts)
...
IP_SimLib  = envx.CompileSimLib(IP_Cores)

```

----

### <pre>WorkLib</pre>
<u>True builder</u> 

Создаёт рабочую библиотеку симулятора (и производит все связанные с этим действия), генерирует файл с параметрами запуска симулятора `handoff.do` (опции запуска, списки исходных файлов и директорий поиска включаемых файлов, переменные и т.д.) и производит собственно компиляцию библиотеки.

----

### <pre>CompileWorkLib</pre>
<u>Pseudo-builder</u> 

Формирует имя целевой директории рабочей библиотеки симулятора, создаёт эту директорию и запускает билдер `WorkLib`.

Пример использования:

```python
src_syn = read_sources('src_syn.yml')
src_sim = read_sources('src_sim.yml')
...
WLib    = envx.CompileWorkLib(src_syn + src_sim)
```

----

### <pre>QuestaGui</pre>
<u>True builder</u> 

Выполняет переход к исполнительному окружению симулятора (env['BUILD_SIM_PATH']) и запускает симулятор в графическом режиме.

----

### <pre>LaunchQuestaGui</pre>
<u>Pseudo-builder</u> 

Вызывает билдер `QuestaGui`, передавая фиктивную цель, что вынуждает всегда запускать действие билдера.

Пример использования:

```python
LaunchQuestaGui = envx.LaunchQuestaGui()
```

----

### <pre>QuestaRun</pre>
<u>True builder</u> 

Производит прогон симуляционной сессии в пакетном режиме (в консоли). Для этого сначала делается переход в директорию с исполнительным окружением симулятора, после чего непосредственно запускается симулятор в пакетном режиме.

----

### <pre>LaunchQuestaRun</pre>
<u>Pseudo-builder</u> 

Вызывает билдер `QuestaRun`, передавая фиктивную цель, что вынуждает всегда запускать действие билдера.

Пример использования:

```python
LaunchQuestaRun    = envx.LaunchQuestaRun()
```

----
