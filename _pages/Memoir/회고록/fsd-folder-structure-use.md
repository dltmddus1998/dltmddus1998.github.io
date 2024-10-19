---
title: "FSD 폴더구조 사용 후기 (1)"
tags:
  - FSD
  - Feature-Sliced Design
  - 방법론
date: "2024-10-19"
thumbnail: "/assets/img/thumbnail/fsd.png"
bookmark: true
---

# FSD 폴더구조

## Overview

### ✍️

> 실제 프로젝트에서 실제로 FSD 폴더구조를 사용했다
> 이 글은 그에 대한 회고가 담긴 글이다.

## FSD 폴더구조란?

<aside>

FSD (Feature-Sliced Design)는 프론트엔드 애플리케이션을 구조화하기 위한 아키텍처 방법론이다.

</aside>

> 코드를 구성하는 규칙과 관례를 모아놓은 것으로, 이 방법론의 주요 목적은 끊임없이 변화하는 **비즈니스 요구사항에 직면해서도 프로젝트를 더 이해하기 쉽고 안정적으로 만드는 것**이다.

## Tutorial

### 간단한 예시

> 자, 가장 부모 단계에 있는 폴더부터 점차적으로 알아가보자.

```jsx
📁 app
	📁 routes
	📁 analytics
📁 pages
	📁 home
	📁 article-reader
		📁 ui
		📁 api
📁 shared
	📁 ui
	📁 api
```

- pages폴더는 *slices*라고 불린다.
  - 해당 폴더는 도메인을 기준으로 레이어를 나눈다. (위의 경우 페이지를 기준으로 레이어를 나눈다.)
- app, shared 폴더, 그리고 pages/article-reader 폴더는 *segments*라고 불린다.
  - 해당 폴더는 기술적인 목적을 기준으로 slices(또는 레이어)를 나눈다.

### 개념

Layers, slices, 그리고 segments는 아래와 같이 계층을 형성한다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/793761ee-1da2-4c3c-8209-ce8f1147ec2d/ca8b350b-5415-45bd-923d-89f16090ec95/image.png)

자, 여기부터는 내가 프로젝트를 진행하면서 활용했던 FSD 폴더구조 방식과 연관지어 말해보겠다.

**✓ 우선 위 사진의 Layers부터 보자.**

- app - app을 실행하기 위한 모든 것들이 여기에 들어간다.
  - 예를 들면, routing, entrypoints, global styles, providers
  - **실제 프로젝트 활용 폴더 구조**
    ```jsx
    📁 app
    	📁 fonts
    	📁 i18n
    	📁 Layouts
    	📁 providers
    		📁 router
    			📁 routes
    	📁 style
    ```
- processes - 이젠 없는 개념…deprecated!!
- pages - 실제 라우팅에 들어가는 전체 페이지나 큰 단위의 페이지
  - **실제 프로젝트 활용 폴더 구조** (라우팅 단위별 페이지로 구분)
    ```jsx
    📁 pages
    	📁 dashboard
    	📁 source
    ```
- widgets - 큰 단위의 독립적인 기능이나 UI (주로 전체적인 유스케이스를 전달한다)
  - **실제 프로젝트 활용 폴더 구조**
    - widgets/layout에는 많은 페이지에서 공통적으로 쓰이는 컴포넌트나 헤더, GNB 및 LNB에 들어가는 widget에 대한 폴더들을 모아뒀다.
    - 그 외 다른 폴더들은 큰 단위의 독립적인 UI나 기능 기준으로 분류했다.
      - 근데 해당 기준으로 모든 폴더를 나눠버리면 너무 방대해져서 pages기준으로 같은 페이지에 해당하는 widgets은 같은 폴더로 분류했다.
    ```jsx
    📁 widgets
    	 📁 layouts
    	 📁 source
    		 📁 source-widgetA
    		 📁 source-widgetB
    ```
- features - 전체 프로덕트 feature들 중 재사용되는 부분들 (즉, 대중적인 의미로 생각하면 컴포넌트라고 생각하면 된다. → 버튼 컴포넌트, 모달 컴포넌트 등등)
  - **실제 프로젝트 활용 폴더 구조**
    - 나의 경우에는 widgets에 들어가는 자잘한 기능들을 feature라고 생각하고 구현했다.
    - 그리고 widgets과 마찬가지로 너무 방대해질 폴더량을 고려하여 특정 기준으로 (widgets과 달리 무조건 페이지 기준은 아니었다. - 레이아웃에 들어가는 feature인 경우도 있었으니…) features를 폴더별로 분류했다.
    ```jsx
    📁 features
    	📁 auth
    		📁 auth-featuresA
    	📁 lnb
    		📁 lsb-item
    		📁 lsb-router-item
    ```
- entities - user, product와 같은 비즈니스 엔티티에 대한 폴더이다
  - **실제 프로젝트 활용 폴더 구조**
    - 나의 경우에는 api (백엔드와 통신하여 데이터를 받아오기 위한 api 구현)와 store (상태 관리), type(typescript를 활용하여 타입 정의는 필수였다)을 page별로 분류했다.
    ```jsx
    📁 entities
    	📁 auth
    		📁 model
    			- store
    			- types
    		📁 api
    ```
- shared - 재사용 가능한 함수들, 특히 프로젝트 세부 사항과 거리가 있지만 필요한 함수들에 대해 구현하는 폴더이다.
  - **실제 프로젝트 활용 폴더 구조**
    - 나의 경우에는 데이터 테이블을 구현하기 위한 재사용 가능한 데이터 관련 훅, 그리고 기타 재사용되는 utils들(date, alert, component 등등)을 분류했다.
      ```jsx
      📁 shared
      	📁 hooks
      	📁 utils
      ```

출처: https://feature-sliced.design/docs/get-started/overview

> 다음 글에 이어서 한 단계 더 들어간 Slices 단계에 대해서 이야기하겠다.
