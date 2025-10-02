# NodeKit

NodeKit은 Node.js 앱, 스크립트, 라이브러리를 위한 간단한 툴킷입니다. 로깅, 텔레메트리, 설정, 오류 처리 등의 기능을 제공하여 다양한 프로젝트에서 일관된 기반을 마련할 수 있게 해줍니다.

## 시작하기

프로젝트에 의존성을 추가하세요:

```bash
npm install --save @gravity-ui/nodekit
```

그 다음, 애플리케이션에서 NodeKit을 가져와 초기화하세요:

```typescript
import {NodeKit} from '@gravity-ui/nodekit';

const nodeKit = new NodeKit();
nodekit.ctx.log('App is ready');
```

## 문서

추가 문서는 `docs/` 디렉토리를 참조하세요:

- [`docs/configuration.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/configuration.md) nodekit 자체와 이를 기반으로 한 애플리케이션을 구성하는 방법을 설명합니다
- [`docs/contexts.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/contexts.md) NodeKit 컨텍스트, 로깅 및 추적 개념을 설명합니다
- [`docs/app-error.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/app-error.md) NodeKit이 애플리케이션에 제공하는 유용한 커스텀 오류 클래스의 설명을 포함합니다
- [`docs/utils.md`](https://github.com/gravity-ui/nodekit/blob/main/docs/utils.md) NodeKit과 함께 번들된 추가 헬퍼 함수 목록을 나열합니다

## 기여하기

### 시작하기

NodeKit 저장소와 예제 애플리케이션의 복사본을 가져오세요:

```bash
git clone git@github.com:gravity-ui/nodekit
git clone git@github.com:gravity-ui/nodekit-examples
```

nodekit을 npm에 링크하고 컴파일러를 시작하세요:

```bash
cd nodekit && npm link && npm run dev
```

그 다음, 다른 터미널에서 examples로 이동해 관심 있는 예제를 열고, nodekit을 링크한 후 앱을 시작하세요:

```bash
cd nodekit-examples/basic-app && npm i && npm link @gravity-ui/nodekit
npm run dev
```

이 시점에서 NodeKit과 데모 앱 모두에 변경을 가하고 실시간으로 결과를 확인할 수 있습니다.