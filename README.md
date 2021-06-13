[![Build Status](https://travis-ci.com/Rogopl/lab07_tutorial.svg?branch=master)](https://travis-ci.com/Rogopl/lab07_tutorial)
```
$ export GITHUB_USERNAME=Rogopl
```
Переходим в рабочую папку
```
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```
Клонируем репозиторий
```
$ git clone https://github.com/${GITHUB_USERNAME}/lab07_tutorial lab08_tutorial
$ cd lab08_tutorial
$ git submodule update --init
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab08_tutorial
```
Создаём докер и указываем базовый образ
```
$ cat > Dockerfile <<EOF
FROM ubuntu:18.04
EOF
```
Говорим докеру, что нужно будет установить
```
$ cat >> Dockerfile <<EOF

RUN apt update
RUN apt install -yy gcc g++ cmake
EOF
```
Копируем файлы нашего каталога и задаём рабочую папку для докера
```
$ cat >> Dockerfile <<EOF

COPY . print/
WORKDIR print
EOF
```
В докере делаем cmake
```
$ cat >> Dockerfile <<EOF

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
```
Устанавливаем значение LOG_PATH
```
$ cat >> Dockerfile <<EOF

ENV LOG_PATH /home/logs/log.txt
EOF
```
Говорим докеру где будут храниться файлы, который останутся после работы с контейнером
```
$ cat >> Dockerfile <<EOF

VOLUME /home/logs
EOF
```
Переходим к папке
```
$ cat >> Dockerfile <<EOF

WORKDIR _install/bin
EOF
```
Создаём точку входа в приложение
```
$ cat >> Dockerfile <<EOF

ENTRYPOINT ./demo
EOF
```
При попытке написать docker build -t logger . в терминале появлялось сообщение об ошибке, где я увидел слова permission и denied. Поэтому в моей голове возникла идея прописать sudo перед командой, в результате чего всё заработало.
```
$ sudo docker build -t logger .
```
Просим докер показать нам, какие существующие образы у нас есть
```
$ sudo docker images
```
создаём директорию logs, создаём интеррактивный процесс logger и записываем туда текст, параллельно записывая кучу ненужных символов, пытаясь понять, как вернуться к терминалу
```
$ mkdir logs
$ sudo docker run -it -v "$(pwd)/logs/:/home/logs/" logger
text1
text2
text3
<C-D>
```
Просим докер вывести подробную информацию о контейнере
```
$ sudo docker inspect logger
```
Проверяем сохранены ли файлы
```
$ cat logs/log.txt
```
Используем gsed
```
$ gsed -i 's/lab07/lab08/g' README.md
```
Редактируем трэвис ямл
```
$ vim .travis.yml
/lang<CR>o
services:
- docker<ESC>
jVGdo
script:
- docker build -t logger .<ESC>
:wq
```
Добавляем всё на гитхаб
```
$ git add Dockerfile
$ git add .travis.yml
$ git commit -m"adding Dockerfile"
$ git push origin master
```
Логинимся в трэвис и запускаем его
```
$ travis login --token
$ travis enable
```
