---
title: 리액트 useDebounce Hook 사용해보기
layout: post
date: '2021-04-10 15:06:00'
author: 진혀쿠
tags: useDebounce debounce throttle React 리액트 hook useState useEffect 사용법 typescript
cover: "/assets/instacode.png"
categories: WEB
---

공부를 하며 블로그 글을 쓰기 때문에 틀린 부분이 있을 수 있습니다. 혹시라도 틀린 부분이 있을 경우 바로 잡아주시면 감사드립니다.

### 문제 상황 🤷‍♂️

로그인, 회원가입 기능을 구현하던 중 예외 처리를 체크해야 할 일이 생겼습니다.  
로직 자체는 그렇게 어렵지 않았으나 사용자가 입력할 때 마다 상태값이 변하면서 리렌더링이 될 것이고 예외 처리를 하는 함수가 그 때마다 호출된다면 굉장히 비효율적이기 때문에 debounce를 적용시키기로 했습니다.  
debounce는 간단하게 설명하자면 특정 시간이 지난 후에 한 번만 이벤트가 실행되도록 하는 것입니다.  
입력이라는 사용자의 이벤트가 발생하고 특정 시간이 지난 후 예외 처리를 한 번만 실행시키는 것이 저의 목적입니다.  

### useDebounce hook 제작하기 🔨

리액트에서 기본적으로 제공해주는 hooks인 useState와 useEffect를 활용할 예정입니다. 코드는 다음과 같습니다.  
```
// utils/useDebounce.ts

import { useState, useEffect } from 'react';

interface debounceProps<T> {
  value: T;
  delay: number;
}

function useDebounce<T>({ value, delay }: debounceProps<T>): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value]);

  return debouncedValue;
}

export default useDebounce;
```

어떤 Type의 value를 받아서 사용하게 될 지 모르기 때문에 제네릭 타입으로 선언했습니다.  
위의 코드를 이해하기 위해서는 useEffect에 대한 이해가 필요합니다. (useState는 이미 알고 있다고 가정하겠습니다.)  
아주 간단하게 설명하겠습니다.  
useEffect는 컴포넌트가 렌더링 될 때 실행되는 함수입니다. 따라서 useEffect 내부의 코드는 컴포넌트가 렌더링 될 때 마다 실행된다고 생각하시면 됩니다. 위의 코드에서는 setTimeout을 사용하여 특정 시간이 지난 후 전달 받은 value의 값을 변경하는 것을 알 수 있습니다.  
useEffect 내부에서 return을 하게 되면 컴포넌트가 제거 될 때 해당 코드들이 실행됩니다. 위의 코드에서는 컴포넌트가 제거 될 때 clearTimeout을 활용하여 setTimeout의 시간을 초기화 시킵니다.  
useEffect의 두 번째 인자로 배열을 전달하게 되면 배열에 포함된 값이 변할 때만 useEffect를 호출하게 됩니다. 위에서는 value값이 변할 때만 useEffect를 호출하게 될 것입니다.  

이를 토대로 코드를 분석해보면 다음과 같습니다.  
0.5초 안에 다른 이벤트가 발생하지 않는다면 setDebouncedValue가 실행되어 바뀐 값이 return 되고 다른 이벤트가 발생하면 기존 값이 그대로 return 됩니다. (컴포넌트에서 실행된다고 가정해주세요.)

### 컴포넌트에서 사용해보기 😊

컴포넌트에서 사용하기 위해 hook을 제작했으니 적용시켜보겠습니다. 저는 다음과 같은 형태로 코드를 짰습니다.  

```
// components/molecules/LoginModalContent.tsx

...

const LoginModalContent = ({ onModalClose }: LoginModalContentProps): JSX.Element => {
const [email, setEmail] = useState('');
const [password, setPassword] = useState('');
const [isValidEmail, setIsValidEmail] = useState(true);
const [isValidPassword, setIsValidPassword] = useState(true);
const onEmailChangeHandler = (e: React.ChangeEvent<HTMLInputElement>) => {
  setEmail(e.target.value);
};
const onPasswordChangeHandler = (e: React.ChangeEvent<HTMLInputElement>) => {
  setPassword(e.target.value);
};

const debouncedEmail = useDebounce({ value: email, delay: 500 });
const debouncedPassword = useDebounce({ value: password, delay: 500 });

useEffect(() => {
  if (debouncedEmail) {
    if (debouncedEmail.length < 8 || debouncedEmail.length > 12) setIsValidEmail(false);
    else setIsValidEmail(true);
  }
  if (debouncedPassword) {
    if (debouncedPassword.length < 8 || debouncedPassword.length > 12) setIsValidPassword(false);
    else setIsValidPassword(true);
  }
}, [debouncedEmail, debouncedPassword]);

...

```

입력이 발생할 때 마다 state가 변하기 때문에 리렌더링이 발생하게 되고 debouncedEmail과 debouncedPassword가 호출됩니다.  
useDebounce 내부 로직에 따라 0.5초 안에 다른 이벤트가 발생하지 않으면 정상적으로 변한 value가 리턴되어 값을 갖게 될 것이고 그렇지 않으면 기존 값을 그대로 리턴하게 됩니다.  
useEffect의 두 번째 인자로 debouncedEmail과 debouncedPassword를 담은 배열을 넘겨주었기 때문에 기존 값이 그대로 리턴 된다면 useEffect가 호출되지 않을 것이고 변경된 값을 리턴받게 된다면 useEffect가 호출되어 원하는 로직을 수행하게 됩니다.  
저같은 경우 입력이 끝난 후 0.5초 후에 이메일과 패스워드의 길이가 원하는 조건을 충족시키는지 검사하는 로직을 useEffect 내부 코드에 넣어주었습니다.  

다음과 같이 잘 실행되는 것을 확인할 수 있습니다.

<img src="{{ site.baseurl }}/assets/useDebounce/debounce.gif" alt="debounce example" title="debounce example" class="picture">

### 좀 더 범용적인 util 함수로 만들어보기 👨‍🏫

위와 같은 형태의 useDebounce는 컴포넌트 내에서 hook으로 밖에 사용할 수 없다는 한계가 있습니다. setTimeout과 clearTimeout을 그대로 사용하되 약간 형태를 바꿔 범용적으로 사용할 수 있게 해보겠습니다.
useDebounce 파일을 아래와 같이 바꿔주세요.

```
// utils/useDebounce.ts

const useDebounce = (func: any, wait: number) => {
  let timeout: NodeJS.Timeout | null;
  return (...args: any) => {
    const context = this;
    if (timeout) clearTimeout(timeout);
    timeout = setTimeout(() => {
      timeout = null;
      func.apply(context, args);
    }, wait);
  };
};

export default useDebounce;
```

원리는 동일합니다. 이미 이벤트가 진행되고 있을 경우 이벤트를 초기화 시키는 방식입니다.

```
// components/molecules/LoginModalContent.tsx

...

const LoginModalContent = ({ onModalClose }: LoginModalContentProps): JSX.Element => {
const classes = useStyles();
const [email, setEmail] = useState('');
const [password, setPassword] = useState('');
const [isValidEmail, setIsValidEmail] = useState(true);
const [isValidPassword, setIsValidPassword] = useState(true);

const onDebouncedEmailChangeListener = useDebounce((e: React.ChangeEvent<HTMLInputElement>) => {
  setEmail(e.target.value);
  if ((e.target.value.length !== 0 && e.target.value.length < 8) || e.target.value.length > 12) setIsValidEmail(false);
  else setIsValidEmail(true);
}, 500);

const onDebouncedPasswordChangeListener = useDebounce((e: React.ChangeEvent<HTMLInputElement>) => {
  setPassword(e.target.value);
  if ((e.target.value.length !== 0 && e.target.value.length < 8) || e.target.value.length > 12) setIsValidPassword(false);
  else setIsValidPassword(true);
}, 500);

...

```

LoginModalContent같은 경우 위와 같이 코드를 변경해주었습니다. 이 때 상태값인 email이나 password가 아닌 e.target.value를 활용하여 값 비교를 해주었는데 useState hook을 활용하여 상태를 변경시킬 경우 비동기로 처리 되기 때문에 email이나 password를 대상으로 비교 연산을 진행하면 이전 값과 비교를 하게 되어 원하는 결과를 얻을 수 없기 때문입니다. 만약 email이나 password를 사용하여 로직을 구성하고 싶으시다면 useEffect를 활용하여 setState가 완료 된 후 리렌더링이 진행될 때 로직을 실행시키시면 됩니다.

제가 진행하고 있는 프로젝트 기준 두 방법의 차이는 다음과 같습니다.
- 첫 번째 방법의 경우 email의 상태는 계속해서 변화하지만 email이 변화하는 동안에 유효성 검사 로직이 실행되지 않습니다. 즉 비교 연산에만 debounce가 적용되었습니다.
- 두 번째 방법은 email의 상태 변화에도 debounce를 적용시킨 경우입니다. 유저가 입력 이벤트 지속하는 동안 email의 상태 자체가 변하지 않으며 비교 연산 역시 진행되지 않습니다.

따라서 성능 측면에서는 두 번째 방법이 유리합니다. 리액트 개발자 도구를 활용하여 렌더링 상태를 보면 다음과 같이 차이가 난다는 것을 확인할 수 있습니다.

#### 첫번째 방법
<img src="{{ site.baseurl }}/assets/useDebounce/debounce_first.gif" alt="first debounce example" title="first debounce example" class="picture">

#### 두번째 방법
<img src="{{ site.baseurl }}/assets/useDebounce/debounce_second.gif" alt="second debounce example" title="second debounce example" class="picture">

첫 번째 방법과 같은 경우 입력이 일어날 때마다 state가 변화하기 때문에 리렌더링이 계속되는 것을 알 수 있습니다. 반면 두 번째 방법은 입력이 끝난 후 state가 변경되기 때문에 리렌더링이 입력이 끝날 때 한 번만 수행됩니다.

### 결론

사용자의 이벤트가 모두 발생한 후 로직이 처리되는 경우라면 성능 측면에서도 유리한 두 번째 방법이 좋아보입니다.

하지만 이벤트에 따라 변경되는 값이 계속해서 로직이 쓰이게 되는 상황이라면 첫번째 방법을 활용해서 코드를 짜셔야 될 것 같습니다.

대부분의 경우 두 번째 방법이 성능적으로 좀 더 좋을 것으로 예상합니다.

### 참고자료
- [제네릭 타입에 관한 설명](https://hyunseob.github.io/2017/01/14/typescript-generic/)
- [useDebounce hook 제작](https://dev.to/gabe_ragland/debouncing-with-react-hooks-jci)