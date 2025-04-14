# 서론
[토스ㅣSLASH 21 - 실무에서 바로 쓰는 Frontend Clean Code](https://www.youtube.com/watch?v=edWbHp_k_9Y)을 읽고 노션에 정리해 보았습니다([링크](https://ionian-breath-903.notion.site/8925e7eb179e48b389e47d1143e429d4?pvs=4)) 
해당 내용을 토대로 모달 컴포넌트를 설계하였습니다

# 모달 컴포넌트의 추상화
`모달` 컴포넌트를 추상화하는 목적은 복잡한 핵심 개념만을 남기고, 재사용 가능하며 유지 보수가 용이한 코드를 작성하는 것입니다. 모달 컴포넌트를 추상화 했을 때 핵심 개념은 `열기`와 `닫기`입니다

### 모달 컴포넌트 추상화 과정

1. **상태 관리**
   - 모달의 상태를 전역에서 관리하기 위해 `Recoil`의 `atom`을 사용합니다
   - 모달의 상태는 열려 있는 모든 모달의 리스트로 관리됩니다

2. **모달 열기와 닫기 함수**
   - 모달을 열고 닫는 로직을 재사용 가능하도록 훅으로 추상화합니다
   - `useModal` 훅을 만들어 모달을 열고 닫는 기능을 제공합니다

### 상태 관리

모달의 상태를 관리하기 위해 `Recoil`을 사용하여 전역 상태를 설정합니다

```typescript
import { ComponentProps, FunctionComponent } from 'react';
import { atom } from 'recoil';

export const modalState = atom<
  Array<{
    Component: FunctionComponent<any>;
    props: ComponentProps<FunctionComponent<any>>;
  }>
>({
  key: '#modal',
  default: [],
});
```

`modalState`는 현재 열려 있는 모달의 리스트를 관리합니다. 각 모달은 컴포넌트와 해당 컴포넌트에 전달될 props로 구성됩니다

### 모달 열기와 닫기 함수

모달을 열고 닫는 로직을 `useModal` 훅으로 추상화합니다

```typescript
import { modalState } from '@states/modal';
import { ComponentProps, FunctionComponent, useCallback } from 'react';
import { useRecoilState } from 'recoil';

const useModal = () => {
  const [modals, setModals] = useRecoilState(modalState);

  const openModal = useCallback(
    <T extends FunctionComponent<any>>(Component: T, props: ComponentProps<T>) => {
      setModals((modals) => [...modals, { Component, props }]);
    },
    [setModals],
  );

  const closeModal = useCallback(
    <T extends FunctionComponent<any>>(Component: T) => {
      setModals((modals) => modals.filter((modal) => modal.Component !== Component));
    },
    [setModals],
  );

  return {
    modals,
    openModal,
    closeModal,
  };
};

export default useModal;
```

- `openModal`: 특정 모달 컴포넌트를 열기 위해 사용됩니다. 모달 컴포넌트와 그에 전달될 props를 받아 상태에 추가합니다
- `closeModal`: 특정 모달 컴포넌트를 닫기 위해 사용됩니다. 모달 컴포넌트를 상태에서 제거합니다

### 모달 사용 예시

추상화된 모달을 사용하는 방법을 보여줍니다
```typescript
interface Props {
  text: string;
}

const TestModalComponent = ({ text }: Props) => {
  return (
    <div>
      <p>Modal 입니다</p>
      <p>{text}</p>
    </div>
  );
};

export default TestModalComponent;
```

```typescript
import useModal from '@hooks/useModal';
import TestModalComponent from './ui/TestModalComponent';

const Home = () => {
  const { openModal, closeModal } = useModal();

  const onOpen = () => {
    openModal(TestModalComponent, {
      text: 'CONTENT !!!',
    });
  };

  const onClose = () => {
    closeModal(TestModalComponent);
  };

  return (
    <div>
      <button onClick={onOpen}>모달 열기</button>
      <button onClick={onClose}>모달 닫기</button>
    </div>
  );
};

export default Home;
```

- `openModal` 함수는 `TestModalComponent`와 그에 전달될 props를 받아 모달을 엽니다
- `closeModal` 함수는 `TestModalComponent`를 닫습니다
