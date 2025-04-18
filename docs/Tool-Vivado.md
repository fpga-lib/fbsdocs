# Tool vivado

## Environment variables

Переменные сборочного и исполнительного окружения удобно задать в сборочном скрипте:

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

envx['ROOT_PATH']                 = dirs.ROOT
envx['BUILD_PATH']                = dirs.BUILD

```

где `env` и `dirs` — объекты с параметрами, получаемые:

```python
env  = import_config('env.yml')
dirs = import_config('dirpath.yml')
```

а конфигурационные файлы `env.yml` и `dirpath` представляют собой нечто подобное:

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

```yaml
#
#   dirpath.yml
#
import : main

options:
    suffix    : _DIR

parameters:
    ROOT           : = os.path.abspath(str(Dir('#')))
    LIB            : = os.path.join(ROOT,   'lib')
    SRC            : = os.path.join(ROOT,   'src')
    COMMON         : = os.path.join(SRC,    'cfg', 'common')
    CFG            : = os.path.join(SRC,    'cfg', main.VARIANT_NAME)
    SCRIPT         : = os.path.join(CFG,    'script')
    SRC_SYN        : = os.path.join(SRC,    'pl', 'syn')
    SRC_SIM        : = os.path.join(SRC,    'pl', 'sim')
    CFG_LOCAL      : = os.path.join(CFG,    'env')
    CFG_COMMON     : = os.path.join(COMMON, 'env')
    SCRIPT_COMMON  : = os.path.join(COMMON, 'script')
    BUILD          : = os.path.join(ROOT,   'build', main.VARIANT_NAME)

```

Из описанных переменных окружения далее формируются пути к исполняемым файлам внешних инструментов и строится структура файлов и директорий для синтезатора и симулятора.

```python
#-------------------------------------------------------------------------------
#
#    Tool 'vivado'
#
...
VIVADO = os.path.join(env['XILINX_VIVADO'], 'bin', 'vivado')
HLS    = os.path.join(env['XILINX_HLS'], 'bin', 'vitis_hls')
...
env['BUILD_SYN_PATH'] = os.path.join(env['BUILD_PATH'], 'syn')
...
```
```python
#-------------------------------------------------------------------------------
#
#    Tool 'questa'
#
...
env['VLOGCOM']        = os.path.join(env['QUESTABIN'], 'vlog')
env['VCOMCOM']        = os.path.join(env['QUESTABIN'], 'vcom')
env['VLIBCOM']        = os.path.join(env['QUESTABIN'], 'vlib')
env['VMAPCOM']        = os.path.join(env['QUESTABIN'], 'vmap')
env['VSIMCOM']        = os.path.join(env['QUESTABIN'], 'vsim')
...
env['BUILD_SIM_PATH'] = os.path.join(env['BUILD_PATH'], 'sim')
env['SIM_CMD_SCRIPT'] = os.path.join(env['ROOT_PATH'], 'site_scons', 'site_tools', 'questa.tcl' )
...
```

!!! info "**ЗАМЕЧАНИЕ**"

    Переменная `envx['BUILD_PATH']` задаёт базовый путь для текущей сборки&nbsp;– место, где создаются все продукты генерации, начиная от генерируемых исходных и заголовочных файлов и заканчивая проектом синтезатора, скриптами симулятора и т.п. 

    Как правило, значение этой переменной связано с именем сборочного варианта&nbsp;– так проще избежать путаницы: все продукты генерации, компиляции, симуляции, синтеза и т.д. воздаются в директории с тем же именем, что и имя сборочного варианта, включая относительный путь. 

    Однако, существует возможность указать любой произвольный путь для целевой сборки, определив переменную сборочного окружения `envx['BUILD_PATH']` соответствующим образом. Это позволяет, в частности, генерировать разные целевые сборки из одного и того же сборочного варианта, указав, например, разные параметры сборки (через аргументы командной строки), и, что крайне важно, такой подход **позволяет запускать эти сборки параллельно**. 

    Возможность запускать сборки параллельно актуальна при прогоне тестов на симуляторе: можно запустить несколько длительных тестов из одного и того же сборочного варианта, указав параметром разные пути к исходным файлам тестов. За счёт многоядерности современных процессоров достигается существенная экономия времени тестирования.

### Переменные общего назначения

|Name   | Description | Default Value
----------------|----------------------------------------|-----
`VIVADO_VERNUM` | Номер используемой<br> версии САПР **Vivado** | Извлекается из <br>`env['XILINX_VIVADO']`
`VIVADO_PROJECT_NAME` | Название проекта **САПР Vivado** | `'vivado_project'`
`TOP_NAME` | Имя модуля верхнего уровня | `'top'`
`DEVICE` | Наименование ПЛИС | `'xc7a200tfbg676-2'`
`VIVADO_PROJECT_MODE` | Режим работы с САПР.<br> В настоящее время поддерживается <br>только Project Mode | `True`
`SYNCOM` | Команда запуска САПР в пакетном<br> режиме | `VIVADO + ' -mode batch'`
`SYNSHELL` | Команда запуска САПР в <br>интерактивном режиме | `VIVADO + ' -mode tcl'`
`SYNGUI` | Команда запуска САПР в <br>графическом режиме | `VIVADO + ' -mode gui'`
`HLSCOM` | Команда запуска HLS компилятора | `HLS`
`SYN_TRACE` | Опция управления трассировкой<br> команд |  `' -notrace'`
`SYN_JOURNAL` | Опция ведения журнала | `' -nojournal'`
`PROJECT_CREATE_FLAGS` | Опции, передаваемые Tcl <br>команде `create_project`. <br>Типовое значение `-f` (force),<br> что вынуждает САПР FPGA <br>создавать проект даже если он<br> открыт |  `''`
`HLSFLAGS` | Флаги компилятора HLS | `''`
`VERBOSE` | Управляет печатью полной<br> командной строки выполняемого<br> действия | `True`

### Директории и пути

Name            | Description                            | Default Value
----------------|----------------------------------------|-----
`CFG_PATH` | Директория текущего сборочного<br> варианта | `os.path.abspath(os.curdir)`
`CONFIG_SEARCH_PATH` | Список путей, по которым<br> осуществляется поиск конфи-<br>гурационных файлов во время <br>работы сканера (обработка <br>секции `import` в конфигу-<br>рационных файлах) |`env['CFG_PATH']`
`BUILD_SRC_PATH` | Директория, куда помещаются <br>сгенерированные исходные файлы <br>(\*.svh, \*.tcl) | `os.path.join(env['BUILD_PATH'], 'src')`
`BUILD_SYN_PATH` | Директория, в которой создаётся<br> исполнительное окружение <br>для синтеза (проект, IP ядра и т.п.) | `os.path.join(env['BUILD_PATH'], 'syn')`
`IP_OOC_PATH` | Директория для IP ядер, скриптов, <br>библиотеки симуляционных <br>моделей | `os.path.join(env['BUILD_SYN_PATH'], 'ip_ooc')`
`BD_OOC_PATH` | Директория для блочных дизайнов, <br>создаваемых out-of-context | `os.path.join(env['BUILD_PATH'], 'bd')`
`BUILD_HLS_PATH` | Директория для создания проектов<br> HLS, скриптов их создания и компи-<br>ляции и репозитория целевых IP <br>ядер | `os.path.join(env['BUILD_SYN_PATH'], 'hls')`
`INC_PATH` | Список путей, в которых произво-<br>дится поиск включаемых файлов. <br>Поиск автоматически выполняется<br> в директориях, где расположены<br> исходные файлы, и `INC_PATH` <br>дополняет этот список элементами,<br> в которых исходных файлов нет,<br> но есть включаемые —<br> например, `BUILD_SRC_PATH` | `''`
`IP_SCRIPT_DIRNAME`     | Имя директории, в которую поме-<br>щаются скрипты для обслужива-<br>ния IP ядер. Сама директория <br>располагается в `IP_OOC_PATH`  | `'_script'`
`BD_SCRIPT_DIRNAME`     | Имя директории, в которую <br>помещаются **Tcl** скрипты <br>создания OOC блочных<br> дизайнов. Сама директория<br> располагается в `BD_OOC_PATH`  | `'_script'`
`SIM_SCRIPT_DIRNAME` | Имя директории, где поме-<br>щаются скрипты для создания <br>симуляционных библиотек IP ядер,<br> блочных дизайнов, HLS IP ядер и т.п. | `'sim_script'`
`SIM_SCRIPT_PATH` | Директория, содержащая скрипты<br> для создания симуляционных <br>библиотек IP ядер, блочных<br> дизайнов, HLS IP ядер и т.п. | `os.path.join(env['BUILD_SYN_PATH'], env['SIM_SCRIPT_DIRNAME'])`
`HLS_SCRIPT_DIRNAME` | Имя директории, в которую<br> помещаются скрипты для создания<br> и компиляции HLS IP | `'_script'`
`HLS_IP_NAME_SUFFIX` | Суффикс для имён HLS IP | `'_hlsip'`

### Расширения файлов

Name            | Description                            | Default Value
----------------|----------------------------------------|-----
`CONFIG_SUFFIX`         | конфигурационные файлы  | `'yml'`
`TOOL_SCRIPT_SUFFIX`    | скрипты  | `'tcl'`
`IP_CORE_SUFFIX`        | IP ядра  | `'xci'`
`BD_SUFFIX`             | Блочные дизайны  | `'bd'`
`DCP_SUFFIX`            | Design Checkpoint. Архив, содержащий<br> результат работы САПР: синтезированный <br>проект, разведённый, синтезированное <br>IP ядро и т.п.  | `'dcp'`
`BITSTREAM_SUFFIX`      | выходной файл  | `'bit'`
`CONSTRAINTS_SUFFIX`    | констрейны  | `'xdc'`
`VIVADO_PROJECT_SUFFIX` | проект **Vivado**  | `'xpr'`
`V_SUFFIX`              | Verilog  | `'v'`
`SV_SUFFIX`             | SystemVerilog  | `'sv'`
`V_HEADER_SUFFIX`       | Verilog header  | `'vh'`
`SV_HEADER_SUFFIX`      | SystemVerilog header  | `'svh'`
`HLS_TARGET_SUFFIX`     | HLS IP repository archived item | `'zip'`
`USER_DEFINED_PARAMS`   | Пользовательские параметры, передаваемые<br> на этап создания проекта САПР путём создания <br>соответствующих переменных в скрипте, <br>который создаёт проект **Vivado**  | `{}`

## Builders

Перечислены [билдеры](Build-Scripts.md#builders) и [псевдобилдеры](Build-Scripts.md#pseudo-builders). Билдеры как правило не используются напрямую, т.к. они имеют
вполне определённый интерфейс запуска, который далеко не всегда удобен, поэтому в скрипте сборочных
сценариев обычно используются псевдобилдеры, которые по сути являются "обёртками" вокруг самих
билдеров.

----

### IpCreateScript
<u>True builder</u>

Генерирует **Tcl** скрипт, с помощью которого создаётся IP ядро. 

----

### IpCreateScripts
<u>Pseudo-builder</u>

Генерирует **Tcl** скрипты для создания IP ядер из списка конфигурационных файлов. Создаёт скрипты по схеме `<name>-create.tcl`. Использует билдер `IpCreateScript`.

Пример использования:

```python
ip = read_sources('ip.yml')
...
IP_Create_Scripts = envx.IpCreateScripts(ip)
```

----

### IpSynScript
<u>True builder</u>

Генерирует скрипт для out-of-context синтеза IP ядра.

----

### IpSynScripts
<u>Pseudo-builder</u> 

Генерирует **Tcl** скрипты для синтеза IP ядер из списка конфигурационных файлов. Создаёт скрипты по схеме `<name>-syn.tcl`. Использует билдер `IpSynScript`.

Пример использования:

```python
ip = read_sources('ip.yml')
...
IP_Syn_Scripts = envx.IpSynScripts(ip)
```

----

### IpCreate
<u>True builder</u>

Создаёт IP ядро.

----

### CreateIps
<u>Pseudo-builder</u> 

Создаёт IP ядра по списку скриптов, сгенерированы с помощью `IpCreateScripts`. Использует билдер `IpCreate`.

Пример использования:

```python
IP_Cores = envx.CreateIps(IP_Create_Scripts)
```

----

### IpSyn
<u>True builder</u>

Синтезирует IP ядро в режиме out-of-context.

----

### SynIps
<u>Pseudo-builder</u> 

Синтезирует IP ядра, используя в качестве зависимостей скрипты, созданные `IpSynScrips` и файлы самих IP ядер. Использует билдер `IpSyn`.

Пример использования:

```python
IP_OOC_Syn = envx.SynIps(IP_Syn_Scripts, IP_Cores)
```

----

### BdCreate
<u>True builder</u>

Создаёт блочный дизайн на основе **Tcl** скрипта, описывающего используемые элементы (cells), их межсоединения и порты. Блочный дизайн создаётся в виде отдельно стоящего **Vivado** проекта (out-of-context). В дальнейшем блочный дизайн из такого проекта может быть подключен в рабочий проект. Методика подготовки **Tcl** скрипта описана [по ссылке](Advanced-Topics.md#bd-create-example).

---- 

### CreateOocBd
<u>Pseudo-builder</u> 

Создаёт проекты с блочными дизайнами на основе списка конфигурационных файлов. Использует `BdCreate` билдер.

```python
bd = read_sources('bd.yml')
...
bd_ooc = envx.CreateOocBd(bd)
```

----

### HlsCSynthScript
<u>True builder</u>

Создаёт **Tcl** скрипт для создания HLS проекта и его компиляции. Зависимостью является `yml` [файл параметров HLS](Build-Variants.md#hls-config). В результате создаётся **Tcl** скрипт с командами создания HLS проекта и его компиляции до элемента IP репозитория. Пример целевого скрипта:

```tcl
--------------------------------------------------------------------------------
#
#   This file is automatically generated. Do not edit the file!
#
#--------------------------------------------------------------------------------

set PROJECT_NAME  adder
set TOP_NAME      adder
set DEVICE        xc7a50tftg256-1
set SOLUTION_NAME sol_1

# Project structure
open_project -reset ${PROJECT_NAME}

# Add syn sources
add_files -cflags "-g -DDATA_WIDTH=4" /opt/slon/xilinx/build-system-examples/src/hls/adder/src/adder.cpp


# Add sim sources
add_files -tb -csimflags "-g -I/opt/cad/mentor/2021.1/questasim/include -I/opt/cad/xilinx/Vitis_HLS/2021.2/include" /opt/slon/xilinx/build-system-examples/src/hls/adder/tb/main.cpp


set_top ${TOP_NAME}
# Add solution
open_solution -reset -flow_target vivado ${SOLUTION_NAME}
set_part ${DEVICE}
create_clock -period 8.0ns -name adder_clk
set_clock_uncertainty 25%

# Add hooks
source /opt/slon/xilinx/build-system-examples/src/cfg/7a50t/hls/adder/directives.tcl


csynth_design

export_design -rtl verilog -format ip_catalog -ipname adder -version 1.0 -vendor slon -library hls -output /opt/slon/xilinx/build-system-examples/build/7a50t/syn/hls/ip/adder.zip

exit
#--------------------------------------------------------------------------------
```

----

### CreateHlsCSynthScript
<u>Pseudo-builder</u>

Псевдобилдер, получающий аргументом список конфигурационных HLS `yml` файлов и осуществляющий запуск для каждого из них билдера `HlsCSynthScript`. Пример использования:

```python
HlsCSynScripts = envx.CreateHlsCSynthScript(hls)
```

----

### HlsCSynth
<u>True builder</u>

Билдер выполняет создание HLS проекта и его компиляцию, в результате которой создаётся элемент IP репозитория, пригодный для использования в качестве IP ядра в целевом проекте. Напрямую не используется, вызывается из псевдобилдера `LaunchHlsCSynth` (см. ниже)

----

### LaunchHlsCSynth
<u>Pseudo-builder</u>

Служит для запуска создания и компиляции HLS проектов. Пример использования:

```python
HlsCsyn = envx.LaunchHlsCSynth(HlsCSynScripts, hls)
```
где, `HlsCSynScripts` — список скриптов, полученных в результате работы `CreateHlsCSynthScript`, `hls` — список конфигурационных `yml` файлов HLS.

----

### HlsIpSynScripts
<u>Pseudo-builder</u>

Псевдобилдер осуществляет генерирование скриптов создания IP ядер из HLS IP репозитория, созданного инструментами, описанными выше. Полученные скрипты далее используются точно так же, как и скрипты для синтеза IP ядер, получаемые из конфигурационных `yml` файлов IP ядер.

```python
HLS_IP_Syn_Scripts = envx.HlsIpSynScripts(HlsCsyn)
```

Использование результата:

```python
#   IP scripts
IP_Create_Scripts  = envx.IpCreateScripts(ip)                 # скрипты создания IP из yml описания
IP_Syn_Scripts     = envx.IpSynScripts(ip)                    # скрипты синтеза IP
HLS_IP_Syn_Scripts = envx.HlsIpSynScripts(HlsCsyn)            # скрипты синтеза HLS IP

#   IP cores
IP_Cores           = envx.CreateIps(IP_Create_Scripts)        # создание IP ядер
All_IP             = IP_Cores + HlsCsyn                       # полный список IP ядер: пользовательских и HLS

All_IP_Syn_Scripts = IP_Syn_Scripts + HLS_IP_Syn_Scripts      # скрипты для синтеза всех IP ядер
IP_OOC_Syn         = envx.SynIps(All_IP_Syn_Scripts, All_IP)  # синтез всех IP ядер
```

----

### CfgParamsHeader
<u>True builder</u>

Создаёт заголовочные файлы HDL с параметрами, взятыми из указанных конфигурационных файлов. Генерируется пара файлов:

* `<hdl-param-filename>.svh`;
* `<hdl-param-filename>_pkg.svh`,

где `<hdl-param-filename>`&nbsp;– имя целевого файла, задаваемое в скрипте сборочного сценария при вызове билдера.

Если в конфигурационном файле, являющимся зависимостью, определён раздел `options`, в котором, в свою очередь, определены параметры `prefix`, `suffix`, то значения этих параметров конкатенируются с именами рабочих параметров, соответственно до и после:

```yaml
#
#  params.yml
#
...
options:
    prefix : PARAMS_

parameters:
    DATA  : 10
    WIDTH : 8
```

Результат:


`<hdl-param-filename>.svh`:

```


`define PARAMS_DATA  10
`define PARAMS_WIDTH 8  


```


`<hdl-param-filename>_pkg.svh`:

```
...
package <hdl-param-filaname>;
    ...
    localparam int PARAMS_DATA  = 10;
    localparam int PARAMS_WIDTH = 8;  
    ...
endpackage    
...
```

Для параметров поддерживается три типа:

1. целый знаковый `int`;
2. с плавающей точкой `real`;
3. строковый `string`.

Типы параметров используются в **package** и выводятся из типов параметров конфигурационных (yaml) файлов.

----

### CreateCfgParamsHeader
<u>Pseudo-builder</u> 

Может принимать в качестве зависимостей строку с именами файлов, разделёнными пробелом, или список. Создаёт полный (абсолютный) путь для каждого файла и вызывает билдер `CfgParamsHeader`.

Пример использования:

```python
hdl_param_deps = 'main.yml clk.yml'
...
CfgParamsHeader = envx.CreateCfgParamsHeader(os.path.join(envx['BUILD_SRC_PATH'], 'cfg_params.svh'), hdl_param_deps)
```

----

### CfgParamsTcl
<u>True builder</u>

Создаёт **Tcl** файл с параметрами, взятыми из указанных конфигурационных файлов. Как и билдер по генерированию включаемых HDL файлов, данный билдер распознаёт раздел `options` с параметрами `prefix` и `suffix`, используя их сходным образом:

```yaml
#
#   dirpath.yml
#
options: 
    suffix : _DIR

parameters:
    SRC_SYN  : = '{' + os.path.join('src', 'syn') + '}'
    SRC_SIM  : = '{' + os.path.join('src', 'sim') + '}'
```

Результат:

```tcl
set SRC_SYN_DIR {src/syn}
set SRC_SIM_DIR {src/sim}
```

----

### CreateCfgParamsTcl
<u>Pseudo-builder</u> 

Может принимать в качестве зависимостей строку с именами файлов, разделёнными пробелом, или список. Создаёт полный (абсолютный) путь для каждого файла и вызывает билдер `CfgParamsTcl`.

Пример использования:

```python
CfgParamsTcl = envx.CreateCfgParamsTcl(os.path.join(envx['BUILD_SRC_PATH'], 'cfg_params.tcl'), 'params.yml')
```

----

### VivadoProject
<u>True builder</u>

Создаёт проект САПР **Vivado**.

----

### CreateVivadoProject
<u>Pseudo-builder</u> 

Принимает в качестве зависимостей исходные файлы HDL, файлы констрейнов и IP ядер. Генерирует имя целевого файла, обрабатывает список зависимостей, формируя полные (абсолютные) пути для каждого файла HDL и констрейнов, подготавливает результирующий список и вызывает билдер `VivadoProject`. Целевым файлом является `<project name>.prj`, который является копией `project name>.xpr`. Такое решение используется по той причине, что САПР **Vivado** в процессе работы постоянно изменяет `xpr` файл, что делает его непригодным в качестве цели для системы сборки. С другой стороны, целевой файл должен быть релевантным к изменениям настроек проекта, т.к. система сборки проверяет актуальность файла по MD хэшу. Копия файла проекта достаточно хорошо отвечает вышеперечисленным требованиям.

Пример использования:

```python
xpr_hook       = read_sources('xpr_hook.yml')
...
xpr_deps = 'src_syn.yml xdc.yml'.split() + xpr_hook
...
VivadoProject = envx.CreateVivadoProject(xpr_deps , IP_Cores)
```

----

### SynthVivadoProject
<u>True builder</u>

Выполняет синтез проекта. Целевым файлом является `<top name>`.dcp, расположенный в `<project name>.runs/synth_1`.

----

### LaunchSynthVivadoProject
<u>Pseudo-builder</u> 

Осуществляет подготовку и запуск проекта на синтез. Принимает в качестве зависимостей целевой файл проекта и перечень исходных файлов, который может быть представлен в виде строки с именами файлов, разделёнными пробелом, либо в виде списка. Запускает билдер `SynthVivadoProject`.

Пример использования:

```python
src_syn  = read_sources('src_syn.yml')
xdc      = read_sources('xdc.yml')
...
syn_deps = src_syn + xdc
...
VivadoProject      = envx.CreateVivadoProject(xpr_deps , IP_Cores)
...
SynthVivadoProject = envx.LaunchSynthVivadoProject(VivadoProject, syn_deps)

```

----

### ImplVivadoProject
<u>True builder</u>

Производит этапы размещения и разводки проекта (place and route). Целевым файлом является битстрим `<project name>.runs/impl_1/<top level>.bit`

----

### LaunchImplVivadoProject
<u>Pseudo-builder</u> 

Выполняет формирование целевого пути и запускает билдер `ImplVivadoProject`.

Пример использования:

```python
SynthVivadoProject = envx.LaunchSynthVivadoProject(VivadoProject, syn_deps)
ImplVivadoProject  = envx.LaunchImplVivadoProject(SynthVivadoProject)
```

----

### OpenVivadoProject
<u>True builder</u>

Запускает САПР **Vivado** в графическом режиме с переходом в директорию проекта и загрузкой проекта. При этом проверяются зависимости и при необходимости выполняется сборка по всей цепочке. Типовой сценарий построен так, что при запуске проекта в графическом режиме проверяется актуальность целевого файла проекта и целевых файлов синтезированных IP ядер. Смысл этого в том, чтобы когда проект запущен в графическом режиме, все "пререквизиты" (проект и IP ядра)  были в актуальном состоянии, и пользователь сразу мог приступить к синтезу проекта, не ожидая завершение посторонних заданий (которые в том числе "засоряют" лог сообщений), и, самое главное, чтобы всё требуемое было в актуальном состоянии, чтобы пользователю не нужно было следить за этим, поменяв тот или иной параметр [сборочного варианта](Build-Variants.md).

----

### LaunchOpenVivadoProject
<u>Pseudo-builder</u> 

Запускает билдер `OpenVivadoProject` с фиктивной целью, что вынуждает билдер всегда включать цель в граф зависимостей как находящуюся в неактуальном состоянии.

Пример использования:

```python
VivadoProject      = envx.CreateVivadoProject(xpr_deps , IP_Cores)
...
OpenVivadoProject  = envx.LaunchOpenVivadoProject(VivadoProject)
```

----
