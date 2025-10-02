## 간단한 설명

`DFDialog` 컴포넌트의 목적은 폼 생성을 간편하게 하는 것입니다. 내부적으로 react-final-form을 사용합니다.
여러 미리 정의된 필드 유형을 지원하지만, 사용자는 `registerDialogControl`을 사용하여 새로운 필드를 등록하여 확장할 수 있습니다.

### 컨트롤

- [기본 컨트롤](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-00-base-controls)
  - `plain` - 정적 텍스트
  - `text` - 편집 가능한 텍스트
  - `multi-text` - 사용자 정의 문자열 배열
  - `checkbox` - 체크박스
  - `tumbler` - 토글 스위치
  - `radio` - 라디오 버튼
  - `editable-list` - 제거 가능한 문자열 목록
  - `multi-editable-list` - 제거 가능한 문자열의 다중 목록
  - `text area` - 텍스트 영역
  - `select` - 선택 상자
  - `block` - ReactNode 추가 허용
- [커스텀 컨트롤 등록](https://preview.yandexcloud.dev/dialog-fields/?path=/story/tutorials-custom-control-registration)

### 기능

- 인플레이스 및 모달 뷰
- [단일 탭](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-01-one-tab) 및 [여러 탭 폼](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-02-several-tab--horizontal-tabs)
- [세로/가로 탭](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-02-several-tab)
- [숨겨진 필드 및 탭](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-04-visibility-condition)
- [값으로 연결된 필드](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-05-extras-and-linked-fields)
- [필드 수준 유효성 검사](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-06-field-validators)
- [폼 수준 유효성 검사](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-07-form-validation)
- [가상화된 탭](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-08-virtualized-tabs)
- [복제 가능한 탭](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-08-cloneable-tabs-)
- [그룹화된 필드](https://preview.yandexcloud.dev/dialog-fields/?path=/story/demo-03-sections)

## 설치

```bash
$ npm install @gravity-ui/dialog-fields
# 아직 설치하지 않았다면 필요한 버전의 react/react-dom을 설치하세요
$ npm install @gravity-ui/dialog-fields react@18 react-dom@18
```

패키지 매니저에 따라 `peerDependencies`를 수동으로 설치해야 할 수 있습니다.

## 사용법

```ts
import {DFDialog, FormApi} from '@gravity-ui/dialog-fields';

interface FormValues {
  firstName: string;
  lastName: string;
}

function MyForm() {
  return (
    <DFDialog<FormValues>
      visible={true}
      headerProps={{
        title: 'My form',
      }}
      onAdd={(form) => {
        console.log(form.getState().values);
        return Promise.resolve();
      }}
      fields={[
        {
          name: 'firstName',
          type: 'text',
          caption: 'First name',
          tooltip: 'Description for first name field',
        },
        {
          name: 'lastName',
          type: 'text',
          caption: 'LastName',
          tooltip: 'Description for last name field',
        },
      ]}
    />
  );
}
```

더 많은 예시는 [storybook](https://preview.yandexcloud.dev/dialog-fields)에서 확인하세요.