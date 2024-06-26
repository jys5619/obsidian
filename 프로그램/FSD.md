![[## 들어가며

프런트엔드 개발자는 종종 애플리케이션 아키텍처와 관련된 문제에 직면합니다. 대부분 모듈 간의 느슨한 결합과 높은 응집력을 제공해야 하며 쉽게 확장할 수 있는 아키텍쳐를 필요로 합니다.

이 글은 기능 분할 설계(Feature-Sliced Design, FSD) 아키텍처에 대해 설명합니다. 제 생각에는 이 아키텍처가 사용 가능한 옵션 중 최고라고 생각합니다. 차례대로 FSD의 개념과 이 아키텍처 방법론이 해결하는 문제를 이야기하고, FSD를 기존 아키텍처 및 모듈식 아키텍처와 비교한 뒤 장단점에 대해 소개하겠습니다.

먼저 레이어(layer), 슬라이스(slice), 세그먼트(segment)의 세 가지 개념을 구분해 봅시다.
![[Pasted image 20240422205401.png]]

## 레이어

레이어는 최상위 디렉토리이자 애플리케이션 분해의 첫 번째 단계입니다. 레이어의 수는 최대 7개로 제한되어 있으며, 일부는 선택 사항이지만 표준화되어 있습니다. 현재 다음과 같이 레이어가 구분되어 있습니다.

⌊src
   ⌊ app (전역 설정/ Provider, Router, Client 같은 HOC가 Slice가됨)
   ⌊ processes/ (deprecated)
   ⌊ pages/ (주소별 페이지 / 각각의 주소별 페이지가 slice)
   ⌊ widgets/ (feature의 몪음/ 어떻게 묶을지는 재사용 여부에 따라)
   ⌊ features(특징)/ (행동/ 통사가 slice, api segment에서는 해당 행동을 요청함.)
   ⌊ entities/ (데이터 /데이터가 slice, aip segment에서는 데이터를 조회)
   ⌊ shared (공유 컴포넌트 / slice 없음)

각 레이어에는 고유한 책임 영역이 있으며 이는 비즈니스 지향적입니다.  
각 레이어를 개별적으로 살펴 보겠습니다.

- app: 애플리케이션 로직이 초기화되는 곳입니다. 프로바이더, 라우터, 전역 스타일, 전역 타입 선언 등이 여기에서 정의됩니다. 애플리케이션의 진입점 역할을 합니다.
- processes: 이 레이어는 여러 단계로 이루어진 등록과 같이 여러 페이지에 걸쳐 있는 프로세스를 처리합니다. 이 레이어는 더 이상 사용되지 않는 것으로 간주되지만 여전히 가끔씩 마주할 수 있습니다. 선택적 레이어입니다.
- pages: 이 레이어에는 애플리케이션의 페이지가 포함됩니다.
- widgets: 페이지에 사용되는 독립적인 UI 컴포넌트입니다.
- features: 이 레이어는 비즈니스 가치를 전달하는 사용자 시나리오와 기능을 다룹니다. 예를 들어 좋아요, 리뷰 작성, 제품 평가 등이 있습니다. 선택적 레이어입니다.
- entities: 이 레이어는 비즈니스 엔티티를 나타냅니다. 이러한 엔티티에는 사용자, 리뷰, 댓글 등이 포함될 수 있습니다. 선택적 레이어입니다.
- shared: 이 레이어에는 특정 비즈니스 로직에 종속되지 않은 재사용 가능한 컴포넌트와 유틸리티가 포함되어 있습니다. 여기에는 UI 키트, axios 설정, 애플리케이션 설정, 비즈니스 로직에 묶이지 않은 헬퍼 등이 포함됩니다.

이러한 레이어들은 코드베이스를 조직화하고, 모듈화되고 유지보수 용이한 확장 가능한 아키텍처를 촉진하는 데 도움이 됩니다.

[![레이어 종류](https://emewjin.github.io/static/cf9634e716ec1c0d73324a70868bf70e/8c557/image3.png "레이어 종류")](https://emewjin.github.io/static/cf9634e716ec1c0d73324a70868bf70e/5a190/image3.png)

기능 분할 설계의 주요 특징 중 하나는 계층 구조입니다. 이 구조에서 features 레이어가 entities 레이어보다 더 위에 있기 때문에 entities 레이어는 features 레이어의 기능을 사용할 수 없습니다. 마찬가지로 features 레이어는 widgets 레이어나 processes 레이어의 컴포넌트를 사용할 수 없으며, 위 레이어는 아래 레이어만 활용할 수 있습니다. 이는 한 방향으로만 향하는 선형적인 흐름을 유지하기 위함입니다.

[![레이어 계층 구조](https://emewjin.github.io/static/5da282349697a9b9bf7c08edd56243f0/8c557/image4.png "레이어 계층 구조")](https://emewjin.github.io/static/5da282349697a9b9bf7c08edd56243f0/5a190/image4.png)

계층 구조에서 레이어의 위치가 낮을수록 코드의 더 많은 곳에서 사용될 가능성이 높기 때문에, 레이어를 변경하는 것이 더 위험합니다. 예를 들어 shared 레이어의 UI 키트는 features, widgets, 심지어 pages 레이어에서도 사용됩니다.

## [](https://emewjin.github.io/feature-sliced-design/#%EC%8A%AC%EB%9D%BC%EC%9D%B4%EC%8A%A4)슬라이스

각 레이어에는 애플리케이션 분해의 두 번째 수준인 슬라이스라는 하위 디렉토리가 있습니다. 슬라이스에서 연결은 추상적인 것이 아니라 특정 비즈니스 엔티티에 대한 것입니다. 슬라이스의 주요 목표는 코드를 값별로 그룹화하는 것입니다.

슬라이스 이름은 프로젝트의 비즈니스 영역에 따라 직접 결정되므로 표준화되어 있지 않습니다. 예를 들어 사진 갤러리에는 사진, 앨범, 갤러리와 같은 섹션이 있을 수 있습니다. 소셜 네트워크에는 게시물, 사용자, 뉴스피드와 같은 슬라이스가 필요할 수 있습니다.

밀접하게 관련된 조각들은 구조적으로 디렉토리 내에 그룹지을 수 있지만 다른 슬라이스와 동일한 격리 규칙을 준수해야 하며, 이 디렉토리에 있는 코드는 직접적으로 공유되지 않아야 합니다.

[![슬라이스 디렉토리](https://emewjin.github.io/static/d3be5bb99bafc26fb4a5b49464052c53/22d0a/image5.png "슬라이스 디렉토리")](https://emewjin.github.io/static/d3be5bb99bafc26fb4a5b49464052c53/22d0a/image5.png)

## [](https://emewjin.github.io/feature-sliced-design/#%EC%84%B8%EA%B7%B8%EB%A8%BC%ED%8A%B8)세그먼트

각 슬라이스는 세그먼트로 구성됩니다. 세그먼트는 목적에 따라 슬라이스 내의 코드를 나누는 데 도움이 됩니다. 팀의 합의에 따라 세그먼트의 구성과 이름이 변경될 수 있습니다. 일반적으로 사용되는 세그먼트들은 다음과 같습니다.

- api - 필요한 서버 요청.
- UI - 슬라이스의 UI 컴포넌트.
- model - 비즈니스 로직, 즉 상태와의 상호 작용. actions 및 selectors가 이에 해당
- lib - 슬라이스 내에서 사용되는 보조 기능.
- config - 슬라이스에 필요한 구성값이지만 구성 세그먼트는 거의 필요하지 않음.
- consts - 필요한 상수.

## [](https://emewjin.github.io/feature-sliced-design/#%EA%B3%B5%EA%B0%9C-api)공개 API

각 슬라이스와 세그먼트에는 공개 API가 있습니다. 공개 API는 index.js 또는 index.ts 파일이며, 이 파일을 통해 슬라이스 또는 세그먼트에서 필요한 기능만 외부로 추출하고 불필요한 기능은 격리할 수 있습니다. 인덱스 파일은 진입점 역할을 합니다.

공개 API에 대한 규칙은 다음과 같습니다.

- 애플리케이션 슬라이스와 세그먼트는 공개 API 인덱스 파일에 정의된 슬라이스의 기능과 컴포넌트만 사용합니다.
- 공개 API에 정의되지 않은 슬라이스 또는 세그먼트의 내부 부분은 격리된 것으로 간주되며 슬라이스 또는 세그먼트 내부에서만 접근할 수 있습니다.

공개 API는 import 및 export로 단순하게 작동하므로 애플리케이션을 변경할 때 코드의 모든 곳에서 import를 변경할 필요가 없습니다.

[![image6](https://emewjin.github.io/static/18e77c77ddb809c9b73b2025811a1092/fc1a1/image6.png "image6")](https://emewjin.github.io/static/18e77c77ddb809c9b73b2025811a1092/fc1a1/image6.png)

## [](https://emewjin.github.io/feature-sliced-design/#%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EC%97%90-%EB%8C%80%ED%95%B4-%EB%8D%94-%EC%9E%90%EC%84%B8%ED%9E%88-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)아키텍처에 대해 더 자세히 알아보기

### [](https://emewjin.github.io/feature-sliced-design/#%EC%B6%94%EC%83%81%ED%99%94-%EB%B0%8F-%EB%B9%84%EC%A6%88%EB%8B%88%EC%8A%A4-%EB%A1%9C%EC%A7%81)추상화 및 비즈니스 로직

계층이 높은 레이어일수록 특정 비즈니스 노드에 더 많이 종속되고 더 많은 비즈니스 로직이 포함됩니다. 계층이 낮은 레이어일수록 추상화 수준이 높고 재사용성이 높으며 레이어 자체의 자율성이 적습니다.

[![image7](https://emewjin.github.io/static/c5a1ca7fade75d2b6e60b6bca7c8a54d/8c557/image7.png "image7")](https://emewjin.github.io/static/c5a1ca7fade75d2b6e60b6bca7c8a54d/5a190/image7.png)

### [](https://emewjin.github.io/feature-sliced-design/#fsd%EA%B0%80-%EB%AC%B8%EC%A0%9C%EB%A5%BC-%ED%95%B4%EA%B2%B0%ED%95%98%EB%8A%94-%EB%B0%A9%EC%8B%9D)FSD가 문제를 해결하는 방식

기능 분할 설계의 과제 중 하나는 결합을 느슨하게 하고 응집력을 높이는 것입니다. FSD가 이러한 결과를 어떻게 달성하는지 이해하는 것은 중요합니다.

OOP에서는 **다형성(polymorphism), 캡슐화(encapsulation), 상속(inheritance)** 및 **추상화(abstraction)** 와 같은 개념을 통해 이러한 문제들을 오랜 시간 동안 해결해 왔습니다. 이러한 개념들은 코드의 격리, 재사용성, 그리고 다양한 결과를 보장합니다. 이는 컴포넌트나 기능이 어떻게 사용되느냐에 따라 다른 결과를 얻을 수 있도록 합니다.

기능 분할 설계는 이러한 원칙들을 프런트엔드에 적용하는 데 도움을 줍니다.

**추상화**와 **다형성**은 레이어를 통해 달성됩니다. 낮은 레이어는 더 추상화 되어있기 때문에 더 높은 레이어에서 재사용될 수 있으며, 특정한 매개변수나 속성에 따라 컴포넌트나 기능이 다르게 작동할 수 있습니다.

**캡슐화**는 슬라이스와 세그먼트 외부에서 필요하지 않은 것을 격리시키는 공개 API를 통해 달성됩니다. 슬라이스의 내부 세그먼트에 대한 접근은 제한되며, 공개 API는 슬라이스 또는 세그먼트의 기능 및 컴포넌트에 접근할 수 있는 유일한 방법입니다.

**상속** 또한 레이어를 통해 달성됩니다. 더 높은 레이어는 낮은 레이어를 재사용할 수 있습니다.

### [](https://emewjin.github.io/feature-sliced-design/#%EA%B3%A0%EC%A0%84%EC%A0%81%EC%9D%B8-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EC%99%80%EC%9D%98-%EB%B9%84%EA%B5%90)고전적인 아키텍처와의 비교

여러분은 고전적인 아키텍처를 여러 번 접해 보셨을 것입니다. 고전적인 소프트웨어 아키텍처에 대한 내용은 그 단순함 때문에 교육적인 글이나 유튜브 영상에서 자주 다루어집니다. 고전적인 아키텍처에는 명확한 표준은 없습니다. 그러나 일반적으로 다음과 같은 형식을 볼 수 있습니다.

[![고전적인 아키텍처 디렉토리](https://emewjin.github.io/static/83ba764b402934b9b3d4ccf7553686d0/a3e09/image8.png "고전적인 아키텍처 디렉토리")](https://emewjin.github.io/static/83ba764b402934b9b3d4ccf7553686d0/a3e09/image8.png)

고전적인 아키텍처에는 눈에 띄는 단점이 있습니다. 가장 큰 단점은 컴포넌트 간의 암묵적인 연결과 모듈의 복잡성 때문에 프로젝트가 유지보수하기 어려워진다는 것입니다. 고전적인 아키텍처의 단점은 시간이 흐를수록 더욱 분명해집니다. 프로젝트가 진화할수록 애플리케이션 아키텍처는 엉망진창이 되어 버립니다.

고전적인 아키텍처는 지속적인 유지보수가 없는 작은 프로젝트나 개인 프로젝트에는 적합합니다.

기능 분할 설계는 그 개념과 표준 덕분에 고전적인 아키텍처의 문제를 해결할 수 있습니다. 그러나 FSD를 사용하는 개발자들은 고전적인 아키텍처를 다룰 때보다 더 높은 이해도와 기술 수준을 필요로 합니다. 일반적으로 2년 미만의 경력을 가진 개발자들은 FSD에 대해 들어보지 못한 경우가 많습니다.  
그러나 기능 분할 설계를 사용할 때는 문제를 “나중”이 아닌 “지금” 해결해야 합니다. 코드의 문제점과 개념에서 벗어난 부분들이 즉시 드러납니다.

### [](https://emewjin.github.io/feature-sliced-design/#%EB%8B%A8%EC%88%9C%ED%95%9C-%EB%AA%A8%EB%93%88%EC%8B%9D-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EC%99%80%EC%9D%98-%EB%B9%84%EA%B5%90)단순한 모듈식 아키텍처와의 비교

단순한 모듈식 아키텍처에는 몇 가지 단점이 있습니다.

- 기능을 어떤 모듈이나 컴포넌트에 넣을지 명확하지 않을 때가 있습니다.
- 다른 모듈 내에서 모듈 사용에 어려움이 있습니다.
- 비즈니스 엔티티를 저장하는 데 문제가 있습니다.
- 글로벌 함수의 암시적 종속성으로 인해 구조가 복잡해집니다.

복잡하거나 적당히 복잡한 프로젝트의 경우, 단순한 모듈식 아키텍처보다 기능 분할 설계를 선호해야 합니다. FSD는 많은 근본적인 아키텍처 문제를 해결하며 단점도 거의 없습니다.

단순함 및 개발 속도 측면에서 단순 모듈식 아키텍처가 FSD보다 유리할 수 있습니다. MVP가 필요하거나 수명이 짧은 프로젝트를 개발하는 경우에는 단순한 모듈식 아키텍처가 FSD보다 적합할 수 있습니다. 하지만 그 외의 경우에는 기능 분할 설계가 더 바람직해 보입니다.

### [](https://emewjin.github.io/feature-sliced-design/#nextjs%EC%99%80-fsd%EC%9D%98-%EC%B6%A9%EB%8F%8C)Next.js와 FSD의 충돌

최근에는 Next.js를 기능 분할 설계와 함께 사용하는 경향이 증가하고 있습니다. Next.js는 FSD와 잘 작동하지만, 페이지의 파일 라우팅과 앱의 부재라는 두 가지 영역에서 충돌이 발생합니다.

#### [](https://emewjin.github.io/feature-sliced-design/#pages)Pages

Next.js에서는 pages 디렉토리가 파일 라우팅을 담당하며, 각 컴포넌트가 하나의 라우트를 나타냅니다. FSD에서는 pages는 페이지들의 평면 목록을 담고 있는 레이어로 사용됩니다. 이로 인해 Next.js 페이지와 FSD 페이지를 어떻게 결합할지에 대한 충돌이 발생합니다.

Next.js와 FSD를 함께 사용할 때, `[root]/pages/`와 같이 Next.js 페이지를 애플리케이션 루트에 저장하는 방법이 있습니다. 그리고 FSD 페이지는 `[root]/src/pages/`와 같이 src 폴더 내에 저장합니다.

다른 해결 방법으로는 두 개의 디렉토리를 유지하는 것이 있습니다. FSD의 페이지들의 평면 목록을 다시 이름 지은 pages-flat과 Next.js의 중첩된 라우트를 위한 pages입니다. 실제 페이지 코드는 pages-flat에 저장되고, 그 후에 pages로 내보낼 수 있습니다.

두 가지 방법 모두 사용하셔도 됩니다만, 저는 첫 번째 방법을 선호합니다.

#### [](https://emewjin.github.io/feature-sliced-design/#app)App

app 레이어의 모든 기본적인 기능은 Next.js에서 처리합니다. 그러나 페이지와 독립적으로 전체 애플리케이션에 대해 무언가를 실행해야 하는 경우 레이아웃 패턴을 사용하여 전체 애플리케이션을 위한 레이아웃을 만들 수 있습니다.

레이아웃 패턴에 대한 Next.js의 문서 [Routing: Pages and Layouts](https://nextjs.org/docs/pages/building-your-application/routing/pages-and-layouts#layout-pattern)를 참고하세요.

### [](https://emewjin.github.io/feature-sliced-design/#%EA%B8%B0%EB%8A%A5-%EB%B6%84%ED%95%A0-%EC%84%A4%EA%B3%84%EC%9D%98-%EC%9E%A0%EC%9E%AC%EB%A0%A5)기능 분할 설계의 잠재력

FSD는 비교적 젊은 소프트웨어 아키텍처 방법론입니다. 그러나 이미 많은 은행, 핀테크, B2B, 전자상거래 등 다양한 기업들에서 사용되고 있습니다. 이 관련 기업 목록은 [Github Issue](https://github.com/feature-sliced/documentation/issues/131)에서 확인할 수 있습니다.

현재 이 글을 작성하는 시점에 공식 FSD 문서의 깃허브 저장소는 858개의 스타를 받았습니다. 이 문서는 활발하게 확장되고 있으며, FSD 개발팀, 텔레그램 및 디스코드 커뮤니티는 아키텍처 관련 질문이 있는 분들을 위해 24시간 연중무휴로 도움을 드리고 있습니다.

이 아키텍처의 잠재력은 매우 높게 평가되며, 전 세계의 다양한 대형 기업들 사이에서 널리 사용되고 있습니다. 적절한 도입이 이루어진다면, FSD는 프런트엔드 개발 분야에서 주도적인 아키텍처 솔루션으로 성장할 수 있는 잠재력을 가지고 있습니다.

## [](https://emewjin.github.io/feature-sliced-design/#%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EC%9D%98-%EC%9E%A5%EC%A0%90%EA%B3%BC-%EB%8B%A8%EC%A0%90)아키텍처의 장점과 단점

### [](https://emewjin.github.io/feature-sliced-design/#%EC%9E%A5%EC%A0%90)장점

- 아키텍처 구성 요소를 쉽게 교체, 추가, 제거할 수 있습니다.
- 아키텍처 표준화.
- 확장성.
- 방법론은 개발 스택과 독립적입니다.
- 예기치 않은 부작용 없이 모듈 간의 연결이 제어되고 명시적입니다.
- 아키텍처 방법론이 비즈니스 지향적입니다.

### [](https://emewjin.github.io/feature-sliced-design/#%EB%8B%A8%EC%A0%90)단점

- 다른 많은 아키텍처 솔루션들에 비해 높은 진입 장벽이 있습니다.
- 인식, 팀 문화 및 개념 준수가 필요합니다.
- 도전 과제와 문제를 나중이 아닌 즉시 해결해야 합니다. 코드 문제와 개념에서 벗어난 부분을 즉시 확인할 수 있습니다. 그러나 이는 장점으로도 볼 수 있습니다.

## [](https://emewjin.github.io/feature-sliced-design/#%EA%B2%B0%EB%A1%A0)결론

기능 분할 설계는 프런트엔드 개발자가 알고 사용할 수 있는 흥미롭고 가치 있는 발견입니다. FSD는 팀에 유연하고 표준화되며 확장 가능한 아키텍처와 개발 문화를 제공할 수 있습니다. 하지만 이 방법론의 긍정적인 측면을 활용하려면 팀 내에서 지식, 인식 및 규칙이 필요합니다.

FSD는 명확한 비즈니스 지향성, 엔티티 정의, 애플리케이션의 기능 및 컴포넌트 구성과 같은 특징들로 인해 다른 아키텍처들 사이에서 돋보입니다.

또한 개별적으로 프로젝트에서 FSD를 사용한 사례와, 기능 분할 설계의 공식 문서를 살펴볼 수도 있습니다.

[공식문서](https://feature-sliced.design/)  
[예제. Github Client](https://github.com/ani-team/github-client)  
[예제. Nike Sneaker and Footwear Store](https://github.com/noveogroup-amorgunov/nukeapp)  
[예제. Sudoku](https://github.com/Shiyan7/sudoku-effector)

이 글이 조금 길었지만, 여러분이 새로운 것을 배웠기를 바랍니다. 이 글을 끝까지 읽어 주셔서 감사합니다. ❤️

의견이나 질문이 있으시면 얼마든지 댓글을 남겨 주세요!