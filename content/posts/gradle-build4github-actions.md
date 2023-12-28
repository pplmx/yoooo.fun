---
categories:
    - general
date: 2020-04-26T15:21:04Z
description: Configure gradle build for github workflow
keywords: gradle, github actions, java, build
lastmod: 2020-04-26T15:30:42Z
tags:
    - gradle
title: How to configure gradle build on Github Actions
---



```yaml
# This workflow will build a Java project with Gradle
# For more information see: https://help.github.com/actions/language-and-framework-guides/building-and-testing-java-with-gradle

name: Java CI with Gradle

on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]

jobs:
    build:
        name: Gradle Automation Build
        runs-on: ubuntu-latest
        strategy:
            matrix:
                java: [11, 13]
        steps:
            -
                uses: actions/checkout@v2
            -
                uses: actions/setup-java@v1
                with:
                    java-version: ${{ matrix.java }}

            # add cache to improve workflow execution time
            -
                name: Cache .gradle/caches
                uses: actions/cache@v1
                with:
                    path: ~/.gradle/caches
                    key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*') }}
                    restore-keys: ${{ runner.os }}-gradle-
            -
                name: Cache .gradle/wrapper
                uses: actions/cache@v1
                with:
                    path: ~/.gradle/wrapper
                    key: ${{ runner.os }}-gradle-wrapper-${{ hashFiles('**/*.gradle') }}
                    restore-keys: ${{ runner.os }}-gradle-wrapper-
            -
                name: Grant execute permission for gradlew
                run: chmod +x gradlew
            -
                name: Build with Gradle
                run: ./gradlew clean build -s
```
