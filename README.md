
Задаём имя
```sh
$ export GITHUB_USERNAME=Rogopl
$ alias gsed=sed
```
Переходим в папочку
```sh
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```
Клонируем репозиторий, в папку lab07_tutorial, переходим туда, добавляем репо и тд
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab06_tutorial projects/lab07_tutorial
$ cd projects/lab07_tutorial
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab07_tutorial
```
Создаём cmake и потом получаем архив по ссылке из туториала и сохраняем в файл HunterGate.cmake
```sh
$ mkdir -p cmake
$ wget https://raw.githubusercontent.com/cpp-pm/gate/master/cmake/HunterGate.cmake -O cmake/HunterGate.cmake
$ gsed -i '/cmake_minimum_required(VERSION 3.4)/a\

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/cpp-pm/hunter/archive/v0.23.251.tar.gz"
    SHA1 "5659b15dc0884d4b03dbd95710e6a1fa0fc3258d"
)
' CMakeLists.txt
```
Удаляем директорию third-party/gtest
```sh
$ git rm -rf third-party/gtest
```
Добавляем через hunter пакет gtest и ищем его
```sh
$ gsed -i '/set(PRINT_VERSION_STRING "v\${PRINT_VERSION}")/a\

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)
' CMakeLists.txt
```
Удаляем строку с добавлением поддиректории gtest
```sh
$ gsed -i 's/add_subdirectory(third-party/gtest)//' CMakeLists.txt
```
Заменяем обращение к gtest на gtest_main -> GTest::main
```sh
$ gsed -i 's/gtest_main/GTest::main/' CMakeLists.txt
```
Запусаем сбору при помощи хантера
```sh
$ cmake -H. -B_builds -DBUILD_TESTS=ON
$ cmake --build _builds
$ cmake --build _builds --target test
```
Выводим содержимое директории
```sh
$ ls -la $HOME/.hunter
```

```sh
## устанавливаем хантер в систему
$ git clone https://github.com/cpp-pm/hunter $HOME/projects/hunter
## устанавливаем значение переменной
$ export HUNTER_ROOT=$HOME/projects/hunter
## удаляем директории билдс
$ rm -rf _builds
## запускаем сборку
$ cmake -H. -B_builds -DBUILD_TESTS=ON
$ cmake --build _builds
$ cmake --build _builds --target test
```

```sh
## просмотр версий джитестподдежирваемеых хантером
$ cat $HUNTER_ROOT/cmake/projects/GTest/hunter.cmake
## создаём каталог
$ mkdir cmake/Hunter
## устанавливаем нужную версию gtest
$ cat > cmake/Hunter/config.cmake <<EOF
hunter_config(GTest VERSION 1.7.0-hunter-9)
EOF
# add LOCAL in HunterGate function
```

```sh
$ mkdir demo
$ cat > demo/main.cpp <<EOF
#include <print.hpp>

#include <cstdlib>

int main(int argc, char* argv[])
{
  const char* log_path = std::getenv("LOG_PATH");
  if (log_path == nullptr)
  {
    std::cerr << "undefined environment variable: LOG_PATH" << std::endl;
    return 1;
  }
  std::string text;
  while (std::cin >> text)
  {
    std::ofstream out{log_path, std::ios_base::app};
    print(text, out);
    out << std::endl;
  }
}
EOF
### Добавляем файл в CMakeLists.txt
$ gsed -i '/endif()/a\

add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print)
install(TARGETS demo RUNTIME DESTINATION bin)
' CMakeLists.txt
```

```sh
## Добавление подмодуля polly, который содержит инструкции для сборки проектов с установленным hunter.
$ mkdir tools
$ git submodule add https://github.com/ruslo/polly tools/polly
$ tools/polly/bin/polly.py --test
$ tools/polly/bin/polly.py --install
$ tools/polly/bin/polly.py --toolchain clang-cxx14
```

