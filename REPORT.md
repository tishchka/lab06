## Laboratory work VI

Данная лабораторная работа посвещена изучению средств пакетирования на примере **CPack**

```sh
$ open https://cmake.org/Wiki/CMake:CPackPackageGenerators
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab06** на сервисе **GitHub**
- [x] 2. Выполнить инструкцию учебного материала
- [x] 3. Ознакомиться со ссылками учебного материала
- [x] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
# Создаем переменную с именем пользователя на GitHub
$ export GITHUB_USERNAME=tishchka
# Создаем переменную окружения названия почты пользователя
$ export GITHUB_EMAIL=katya.t0505@gmail.com
# Создаем команду редактирования с помощью тестового редактора
$ alias edit=subl
# Заменяем команду gsed на команду sed
$ alias gsed=sed 
```

```sh
# Переходим в рабрчую директорию
$ cd ${GITHUB_USERNAME}/workspace
# Запоминаем текущий каталог в виртуальном стеке каталогов
$ pushd .
~/tishchka/workspace ~/tishchka/workspace

# Исполняем команду из файла scripts/activate
$ source scripts/activate
```

```sh
# Клонируем репозиторий, созданный в 5-ой лабораторной,
# в папку с лабораторной №6
$ git clone https://github.com/${GITHUB_USERNAME}/lab05 projects/lab06
Cloning into 'projects/lab06'...
remote: Enumerating objects: 37, done.
remote: Counting objects: 100% (37/37), done.
remote: Compressing objects: 100% (21/21), done.
remote: Total 37 (delta 8), reused 37 (delta 8), pack-reused 0
Unpacking objects: 100% (37/37), 5.32 KiB | 87.00 KiB/s, done.

# Переходим в созданную директорию
$ cd projects/lab06
# Удаляем ссылку на репозиторий 5-ой лабораторной
$ git remote remove origin
# Добывляем ссылку с именем origin на созданный репозиторий 6-ой лабораторной
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab06
```

```sh
# Настройка CPack через определение CPack-переменных внутри файла CMakeLists.txt Добавление истории версий для проекта: версия 0.1.0.0:

# Полная версия пакета
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_STRING "v\${PRINT_VERSION}")
' CMakeLists.txt
# Версия исправленных ошибок
$ gsed -i '/project(print)/a\
set(PRINT_VERSION\
  \${PRINT_VERSION_MAJOR}.\${PRINT_VERSION_MINOR}.\${PRINT_VERSION_PATCH}.\${PRINT_VERSION_TWEAK})
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_TWEAK 0)
' CMakeLists.txt
# Версия пакета исправления
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_PATCH 0)
' CMakeLists.txt
# Младшая версия пакета
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_MINOR 1)
' CMakeLists.txt
# Основная версия пакета
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_MAJOR 0)
' CMakeLists.txt

# Сравниваем изменения, внесенные в файлы
$ git diff
diff --git a/CMakeLists.txt b/CMakeLists.txt
index aa7a323..71b64e3 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -7,6 +7,13 @@ option(BUILD_EXAMPLES "Build examples" OFF)
 option(BUILD_TESTS "Build tests" OFF)
 
 project(print)
+set(PRINT_VERSION_MAJOR 0)
+set(PRINT_VERSION_MINOR 1)
+set(PRINT_VERSION_PATCH 0)
+set(PRINT_VERSION_TWEAK 0)
+set(PRINT_VERSION
+  ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK})
+set(PRINT_VERSION_STRING "v${PRINT_VERSION}")
 
 add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)

```

```sh
# Создание файла DESCRIPTION и его открытие для редактирования
$ touch DESCRIPTION && edit DESCRIPTION
# Создание файла ChangeLog.md - список изменений в пакете
$ touch ChangeLog.md
# Создание переменной DATE
$ export DATE="`LANG=en_US date +'%a %b %d %Y'`"
# Записываем в файл дату, имя пользователя, его почту и версию пакета
$ cat > ChangeLog.md <<EOF
* ${DATE} ${GITHUB_USERNAME} <${GITHUB_EMAIL}> 0.1.0.0
- Initial RPM release
EOF
```

```sh
# Создаем файл CPackConfig.cmake
# Подключение необходимых системных библиотек
$ cat > CPackConfig.cmake <<EOF
include(InstallRequiredSystemLibraries)
EOF
```

```sh
# Установка значений переменных в пакете
$ cat >> CPackConfig.cmake <<EOF
set(CPACK_PACKAGE_CONTACT ${GITHUB_EMAIL})
set(CPACK_PACKAGE_VERSION_MAJOR \${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR \${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH \${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK \${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION \${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE \${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing")
EOF
```

```sh
# Добавление файлов в пакет
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_RESOURCE_FILE_LICENSE \${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README \${CMAKE_CURRENT_SOURCE_DIR}/README.md)
EOF
```

```sh
# Настройки для RPM-пакета
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_RPM_PACKAGE_NAME "print-devel")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "print")
set(CPACK_RPM_CHANGELOG_FILE \${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)
EOF
```

```sh
# Настройки для Debian-пакета
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_DEBIAN_PACKAGE_NAME "libprint-dev")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
EOF
```

```sh
# Подключение модуля CPack
$ cat >> CPackConfig.cmake <<EOF

include(CPack)
EOF
```

```sh
# Добавление CPackConfig.cmake в основной CMakeLists.txt
$ cat >> CMakeLists.txt <<EOF

include(CPackConfig.cmake)
EOF
```

```sh
# Изменяем значение для иконки travis для лабораторной 6
$ gsed -i 's/lab05/lab06/g' README.md
```

```sh
# Добавляем файлы для отслеживания
$ git add .
# Создаем коммит
$ git commit -m"added cpack config"
[main b0df96d] added cpack config
 5 files changed, 36 insertions(+), 1 deletion(-)
 create mode 100644 CPackConfig.cmake
 create mode 100644 ChangeLog.md
 create mode 100644 DESCRIPTION

# Создаем тэг с версией пакета
$ git tag v0.1.0.0
# Загружаем изменения на сервер
$ git push origin main --tags
Username for 'https://github.com': tishchka
Password for 'https://katya.t0505@github.com': 
Enumerating objects: 43, done.
Counting objects: 100% (43/43), done.
Delta compression using up to 2 threads
Compressing objects: 100% (35/35), done.
Writing objects: 100% (43/43), 6.25 KiB | 305.00 KiB/s, done.
Total 43 (delta 11), reused 0 (delta 0)
remote: Resolving deltas: 100% (11/11), done.
To https://github.com/tishchka/lab06
 * [new branch]      main -> main
 * [new tag]         v0.1.0.0 -> v0.1.0.0

```

```sh
# Авторизируемся
$ travis login --auto
Successfully logged in as tishchka!

# Проверяем на доступность 
$ travis enable
Detected repository as tishchka/lab06, is this correct? |yes| yes
tishchka/lab06: enabled :)

```

```sh
# Генерируем файлы сборки в директорию build и настраиваем их
$ cmake -H. -B_build
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/tishchka/tishchka/workspace/projects/lab06/_build

# Запускаем сборку проекта
$ cmake --build _build
Scanning dependencies of target print
[ 50%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[100%] Linking CXX static library libprint.a
[100%] Built target print

# Переходим в директорию _build
$ cd _build
# Архивируем пакет
$ cpack -G "TGZ"
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: print
CPack: - Install project: print []
CPack: Create package
CPack: - package: /home/tishchka/tishchka/workspace/projects/lab06/_build/print-0.1.0.0-Linux.tar.gz generated.

# Поднимаемся на директорию выше
$ cd ..
```

```sh
# Генерируем и создаем файлы для сборки проекта через генератор сборки "TGZ"
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
-- Configuring done
-- Generating done
-- Build files have been written to: /home/tishchka/tishchka/workspace/projects/lab06/_build

$ cmake --build _build --target package
[100%] Built target print
Run CPack packaging tool...
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: print
CPack: - Install project: print []
CPack: Create package
CPack: - package: /home/tishchka/tishchka/workspace/projects/lab06/_build/print-0.1.0.0-Linux.tar.gz generated.

```

```sh
$ mkdir artifacts # Создаем директорию artifacts
$ mv _build/*.tar.gz artifacts # Перемещаем в нее заархивированный пакет
$ tree artifacts # Демонстрация дерева директории artifacts
artifacts
└── print-0.1.0.0-Linux.tar.gz

0 directories, 1 file
```

## Report

```sh
$ popd
$ export LAB_NUMBER=06
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Homework

После того, как вы настроили взаимодействие с системой непрерывной интеграции,</br>
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься</br>
о создание пакетов для измениний, которые помечаются тэгами (см. вкладку [releases](https://github.com/tp-labs/lab06/releases)).</br>
Пакет должен содержать приложение _solver_ из [предыдущего задания](https://github.com/tp-labs/lab03#задание-1)
Таким образом, каждый новый релиз будет состоять из следующих компонентов:
- архивы с файлами исходного кода (`.tar.gz`, `.zip`)
- пакеты с бинарным файлом _solver_ (`.deb`, `.rpm`, `.msi`, `.dmg`)

В качестве подсказки:
```sh
$ cat .travis.yml
os: osx
script:
...
- cpack -G DragNDrop # dmg

$ cat .travis.yml
os: linux
script:
...
- cpack -G DEB # deb

$ cat .travis.yml
os: linux
addons:
  apt:
    packages:
    - rpm
script:
...
- cpack -G RPM # rpm

$ cat appveyor.yml
platform:
- x86
- x64
build_script:
...
- cpack -G WIX # msi
```

Для этого нужно добавить ветвление в конфигурационные файлы для **CI** со следующей логикой:</br>
если **commit** помечен тэгом, то необходимо собрать пакеты (`DEB, RPM, WIX, DragNDrop, ...`) </br>
и разместить их на сервисе **GitHub**. (см. пример для [Travi CI](https://docs.travis-ci.com/user/deployment/releases))</br>

## Links

- [DMG](https://cmake.org/cmake/help/latest/module/CPackDMG.html)
- [DEB](https://cmake.org/cmake/help/latest/module/CPackDeb.html)
- [RPM](https://cmake.org/cmake/help/latest/module/CPackRPM.html)
- [NSIS](https://cmake.org/cmake/help/latest/module/CPackNSIS.html)

```
Copyright (c) 2015-2021 The ISC Authors
```
