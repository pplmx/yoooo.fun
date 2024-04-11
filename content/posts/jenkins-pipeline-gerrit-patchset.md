---
categories:
    - ci
date: 2024-04-11T14:25:42Z
description: Get Patchset Number from a Gerrit Event in a Jenkins Pipeline
keywords:
    - ci
    - jenkins
    - gerrit
    - pipeline
    - groovy
lastmod: 2024-04-11T14:25:42Z
tags:
    - ci
    - jenkins
    - gerrit
    - pipeline
    - groovy
title: How to get the Patchset Number from a Gerrit Event in a Jenkins Pipeline
---




# Retrieving the Patchset Number from a Gerrit Event

## clone by http

```groovy
pipeline {
    agent any
    options { skipDefaultCheckout() }
    stages {
        stage('Checkout PatchSet') {
            steps {
                checkout scmGit(
                    branches: [[name: 'FETCH_HEAD']],
                    extensions: [
                        cloneOption(
                            depth: 1,
                            shallow: true,
                            noTags: true,
                        )
                    ],
                    userRemoteConfigs: [[
                        refspec: '${GERRIT_REFSPEC}',
                        url: 'https://${GERRIT_HOST}/${GERRIT_PROJECT}'
                    ]]
                )
            }
        }
    }
}

```

## clone by ssh

```groovy
pipeline {
    agent any
    options { skipDefaultCheckout() }
    stages {
        stage('Checkout PatchSet') {
            steps {
                checkout scmGit(
                    branches: [[name: 'FETCH_HEAD']],
                    extensions: [
                        cloneOption(
                            depth: 1,
                            shallow: true,
                            noTags: true,
                        ),
                        [$class: 'UserIdentity', email: 'jenkins@x.internal', name: 'Jenkins']
                    ],
                    userRemoteConfigs: [[
                        credentialsId: '<CRED_ID>',
                        refspec: '${GERRIT_REFSPEC}',
                        url: "ssh://jenkins@${GERRIT_HOST}:${GERRIT_PORT}/${GERRIT_PROJECT}"
                    ]]
                )
            }
        }
    }
}

```
