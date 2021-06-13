[![Build Status](https://travis-ci.com/Rogopl/lab09_tutorial.svg?branch=master)](https://travis-ci.com/Rogopl/lab09_tutorial)
## Laboratory work IX

Данная лабораторная работа посвещена изучению процесса создания артефактов на примере **Github Release**

```sh
$ open https://help.github.com/articles/creating-releases/
```

## Tasks

- [ ] 1. Создать публичный репозиторий с названием **lab09** на сервисе **GitHub**
- [ ] 2. Ознакомиться со ссылками учебного материала
- [ ] 3. Получить токен для доступа к репозиториям сервиса **GitHub**
- [ ] 4. Выполнить инструкцию учебного материала
- [ ] 5. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
$ export GITHUB_TOKEN=<полученный_токен>
$ export GITHUB_USERNAME=Rogopl
$ export PACKAGE_MANAGER=apt
$ export GPG_PACKAGE_NAME=gpg
```

```sh
$ $PACKAGE_MANAGER install xclip #Устанавливаем утилиту xclip, предоставляющую доступ к буферу обмена Х из коммандной строки
$ alias gsed=sed
$ alias pbcopy='xclip -selection clipboard'
$ alias pbpaste='xclip -selection clipboard -o'
```
Скачивание и установка пакета Go, для работы с релизами Github
```sh
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
$ go get github.com/aktau/github-release
```
Клонируем репозиторий
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab08_tutorial projects/lab09_tutorial
$ cd projects/lab09_tutorial
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab09_tutorial
```
Заменили все упоминания lab08_tutorial на lab09_tutorial
```sh
$ gsed -i 's/lab08/lab09/g' README.md
```
В этом блоке мы устанавливаем GPG для шифровании информации(текста например) и для электронных цифровых подписей. Затем проверяем если ли у нас секретные ключи и потом сами генерируем это ключи под параметры которые нам предложит GPG
```sh
$ $PACKAGE_MANAGER install ${GPG_PACKAGE_NAME}
$ gpg --list-secret-keys --keyid-format LONG
$ gpg --full-generate-key
#Вводим секретный ключ
$ gpg --list-secret-keys --keyid-format LONG
$ gpg -K ${GITHUB_USERNAME}
# Сохраняем перменную с публичным ключом
$ GPG_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep ssb | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}')
# Сохраняем переменную с секретным ключом
$ GPG_SEC_KEY_ID=$(gpg --list-secret-keys --keyid-format LONG | grep sec | tail -1 | awk '{print $2}' | awk -F'/' '{print $2}')
# Выводим ключ в ASCII и копируем его в буфер обмена
$ gpg --armor --export ${GPG_KEY_ID} | pbcopy
$ pbpaste
# Зесь мы должны открыть гитхаб сеттинг, и устанавливаем ключ, но у меня open не работает и поэтому я открыл не через консоль
$ open https://github.com/settings/keys
#Добавляем к гитхаб ключ который мы создали
$ git config user.signingkey ${GPG_SEC_KEY_ID}
$ git config gpg.program gpg
```
Настраиваем скрипт для добавления сообщения к тегу
```sh
$ test -r ~/.bash_profile && echo 'export GPG_TTY=$(tty)' >> ~/.bash_profile
$ echo 'export GPG_TTY=$(tty)' >> ~/.profile
```

```sh
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
$ cmake --build _build --target package
```
Логинимся в трэвис
```sh
$ travis login --auto
$ travis enable
```

```sh
# Создание тега с сообщением с информацией а затем вервефеируем этот тег
$ git tag -s v0.1.0.0
$ git tag -v v0.1.0.0
# Смотрим на изменения
$ git show v0.1.0.0
# Пушим
$ git push origin master --tags
```
Затем здесь делаем релиз. Но у меня не работала первая команда. Проблема решилась установкой go вручную и изменением некоторых параметров в файле .profile
```sh
$ github-release --version
$ github-release info -u ${GITHUB_USERNAME} -r lab09_tutorial
$ github-release release \
    --user ${GITHUB_USERNAME} \
    --repo lab09_tutorial \
    --tag v0.1.0.0 \
    --name "libprint" \
    --description "my first release"
```

```sh
# Добавление артефакта с указанием ОС и архитектуры, на которых происходила компиляция библиотек
$ export PACKAGE_OS=`uname -s` PACKAGE_ARCH=`uname -m` 
$ export PACKAGE_FILENAME=print-${PACKAGE_OS}-${PACKAGE_ARCH}.tar.gz
$ github-release upload \
    --user ${GITHUB_USERNAME} \
    --repo lab09_tutorial \
    --tag v0.1.0.0 \
    --name "${PACKAGE_FILENAME}" \
    --file _build/*.tar.gz
```

```sh
$ github-release info -u ${GITHUB_USERNAME} -r lab09_tutorial

# Скачивание артефакта из раздела релизов для проверки
$ wget https://github.com/${GITHUB_USERNAME}/lab09_tutorial/releases/download/v0.1.0.0/${PACKAGE_FILENAME}
#Проверяем что за архив мы скачали
$ tar -ztf ${PACKAGE_FILENAME}
```

