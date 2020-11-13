---
layout: post
title: Helm v2로 k8s에 MySQL 띄우기
tags: [Kubernetes, Helm]
comments: true
---

## 0. 준비물

- Kubernetes 클러스터
- 해당 클러스터에 접속할 수 있는 kubectl (~/.kube/config 파일이 세팅되어 있어야 함)

## 1. Helm v2 설치

`brew install helm@2`

- Helm 설치는 자동으로 완료되나, 환경변수에 자동 등록되지 않음. (v3 최신버전 helm과 중복되지 않도록 하기 위함인 듯)
- PATH에 helm 경로를 추가하여 환경변수에 이를 등록한다. `echo 'export PATH="/usr/local/opt/helm@2/bin:$PATH"' >> ~/.{{YOUR SHELL RC}}`

## 2. 초기설정

**Tiller 세팅**

- Helm은 클라이언트의 요청을 처리하는 **Tiller** 라는 Helm Server가 필요하고, 이는 클러스터 내에서 Pod로 동작한다.
    - v3 에서는 보안 이슈로 인해 Tiller가 사라졌다.
- 이 Tiller가 사용할 cluster-admin 권한의 ServiceAccount가 필요하다.
    - 아래 내용의 yaml 파일을 생성한다. (rbac-config.yaml)

        ```yaml
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: tiller
          namespace: kube-system
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
          name: tiller
        roleRef:
          apiGroup: rbac.authorization.k8s.io
          kind: ClusterRole
          name: cluster-admin
        subjects:
          - kind: ServiceAccount
            name: tiller
            namespace: kube-system
        ```

    - 해당 파일을 적용시킨다.
    `kubectl apply -f rbac-config.yaml`

**Helm 초기화**

`helm init --service-account tiller --history-max 200`

위 명령어로 Heml 초기화를 진행한다.

- --service-account 옵션으로 로 위에서 생성한 ServiceAccount의 이름을 지정해준다.
- --history-max 옵션으로 history가 무한으로 생성되는 것을 제한한다.

## 3. 차트 설치하기

**차트 리포지토리 준비**

`helm repo add stable [https://kubernetes-charts.storage.googleapis.com/](https://kubernetes-charts.storage.googleapis.com/)`

- 공식 Stable 차트를 Helm 레포지토리에 추가한다.
- 해당 레포지토리의 차트들을 `helm search` 또는 `helm search {{KEYWORD}}` 명령어로 찾을 수 있다.

**차트 설치**

`helm repo update` 

- 최신 버전의 차트로 업데이트 하고

`helm install stable/mysql` 

- MySQL 차트를 설치한다.