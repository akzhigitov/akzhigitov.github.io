---
date: 2020-10-28
subtitle: Как описать Dockerfile для запуска Java в Kubernetes
title: Запуск Java в Kubernetes. Dockerfile
tags:
- java
- k8s
---

Для запуска любого приложения в Kubernetes требуется завернуть это приложение в контейнер.
<!--more-->
Для этого надо создать в корне репозитория Dockefile.

# Подготовка Dockerfile
Пример самого просто Dockerfile для запуска 11 версии Java:

```
FROM openjdk:11
ENV JAVA_OPTS ""

WORKDIR /srv
COPY app.jar /srv/app.jar

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

## Выбор базового образа

Первая строчка `FROM openjdk:11` в Dockerfile определяет базовый образ для нашего образа.
Выбор базового образа влияет на то, какие библиотеки и утилиты будут нам доступны.

В примере запуск происходит на сборке от openjdk c 11 версией java внутри.

### Варианты выбора базового образа

Есть относительно большой выбор базовых образов зависящих от предоставляемой JDK или JRE.
При выборе официальной сборки от Oracle есть два варианта:

1. Коммерческая версия с платной поддержкой:
[Oracle JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html) 


2. Бесплатная:
[OpenJDK Build от Oracle](http://jdk.java.net/)


Есть и другие бесплатные и открытые, которые, при желании можно рассмотреть:
* [AdoptOpenJDK](https://adoptopenjdk.net/)
* [Azul Zulu](https://www.azul.com/downloads/zulu-community)
* [GraalVM CE](https://www.graalvm.org/downloads/)


### Выбор версии openjdk

https://hub.docker.com/_/openjdk

У образов openjdk есть несколько типов версий в зависимости от того, что в нем собираемся запускать и за счет этого можно сильно сократить объем финального образа.

Посмотрим на размеры базовых образов:

	docker pull --quiet openjdk:11 
	docker pull --quiet openjdk:11-slim 
	docker pull --quiet openjdk:11-jdk  
	docker pull --quiet openjdk:11-jdk-slim 
	docker pull --quiet openjdk:11-jre 
	docker pull --quiet openjdk:11-jre-slim 
	docker images

И получаем такую картину:

	openjdk                       11                  628MB
	openjdk                       11-jdk              628MB
	openjdk                       11-slim             403MB
	openjdk                       11-jdk-slim         403MB
	openjdk                       11-jre              286MB
	openjdk                       11-jre-slim         205MB


## Передача параметров запуска Java

Вторая строчка `ENV JAVA_OPTS ""` объявляем пустую переменную окружения. 
При запуске контейнер в переменную JAVA_OPTS будет передана переменная окружения.

## Рабочая директорию

Следующая строчка `WORKDIR /srv` устанавливает рабочую директорию. Если такой директории нет, то она будет создана. 
Это не обязательный шаг при работе из образа, просто позволяет отделить другие приложения находящиеся в базовом образе от нашего.

## Копирование приложения в образ

Строчка `COPY app.jar /srv/app.jar` копирует файл с названием `app.jar` из контекста в котором был запущен билд образа.
В самом простом понимании контекст сборки - это директория для которой запущен сборка, как правило (хоть это можно и переопределить) место где лежит Dockerfile

app.jar кладется сюда внешней системой сборки, например, из GitLab CI

## Запуск Java

Последняя строчка ENTRYPOINT `["sh", "-c", "java $JAVA_OPTS -jar app.jar"]` это инструкция как запустить наше Java приложение.