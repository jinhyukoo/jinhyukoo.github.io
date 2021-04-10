---
title: 리액트 useDebounce Hook 만들어보기
layout: post
date: '2021-04-10 15:06:00'
author: 진혀쿠
tags: useDebounce debounce throttle React 리액트 hook useState useEffect
cover: "/assets/instacode.png"
categories: WEB
---

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

### 참고자료
- [제네릭 타입에 관한 설명](https://hyunseob.github.io/2017/01/14/typescript-generic/)
- [useDebounce hook 제작](https://dev.to/gabe_ragland/debouncing-with-react-hooks-jci)