# 항해 플러스 프론트엔드 4기 과제 3주차 <br/>: Chapter 1-3. React, Beyond the Basics
- [과제 체크포인트](#과제-체크포인트)
  - [기본과제](#기본과제)
  - [심화과제](#심화과제)
- [과제 셀프회고](#과제-셀프회고)
  - [기술적 성장](#기술적-성장)
  - [코드 품질](#코드-품질)
  - [학습 효과 분석](#학습-효과-분석)
  - [과제 피드백](#과제-피드백)
- [리뷰 받고 싶은 내용](#리뷰-받고-싶은-내용)

## 과제 체크포인트

### 기본과제
- [x] shallowEquals 구현 완료
- [x] deepEquals 구현 완료
- [x] memo 구현 완료
- [x] deepMemo 구현 완료
- [x] useRef 구현 완료
- [x] useMemo 구현 완료
- [x] useDeepMemo 구현 완료
- [x] useCallback 구현 완료

### 심화과제
- [x] 기본과제에서 작성한 hook을 이용하여 렌더링 최적화를 진행하였다.
- [x] Context 코드를 개선하여 렌더링을 최소화하였다.

## 과제 셀프회고

### 기술적 성장

> 🙌 새로 학습한 개념

#### 참조 동일성
- 두 값이 메모리 상에서 동일한 객체를 가리키고 있는지 비교하는 개념입니다.
  - 문자열, 숫자, boolean과 같은 원시 타입은 값 그대로 저장, 할당되고 복사됩니다.
  - 객체, 배열, 함수와 같은 참조 타입은 객체가 아닌 메모리상의 주소인 참조가 저장되어 객체가 할당된 변수를 복사하면 객체가 아닌 객체의 참조가 복사됩니다.

#### shallowEquals
1. 참조가 같은 경우 두 객체는 완전히 동일하므로 `true`를 반환 (=== 연산자로 비교)
2. 객체가 아닌 값이거나 `null`이 포함된 경우 `false`를 반환 (객체인지 확인)
3. 두 객체의 key 목록을 가져와 키 갯수가 다르다면 `false`를 반환
4. objA의 key를 순회하여 objB에 key가 없거나 objA의 키와 objB의 키가 다르다면 `false`를 반환

```typescript
export function shallowEquals<T>(objA: T, objB: T): boolean {
  if (objA === objB) return true;

  if (
    typeof objA !== "object" ||
    objA === null ||
    typeof objB !== "object" ||
    objB === null
  )
    return false;

  const keysA = Object.keys(objA);
  const KeysB = Object.keys(objB);

  if (keysA.length !== KeysB.length) return false;

  for (const key of keysA) {
    if (
      !objB.hasOwnProperty(key) ||
      objA[key as keyof T] !== objB[key as keyof T]
    ) {
      return false;
    }
  }
  return true;
}
```

#### deepEquals
1. 참조가 같은 경우 두 객체는 완전히 동일하므로 `true`를 반환 (=== 연산자로 비교)
2. 객체가 아닌 값이거나 `null`이 포함된 경우 `false`를 반환 (객체인지 확인)
3. 두 값이 배열인지 확인
4. 하나라도 배열이 아니라면 `false`반환
5. 두 값이 모두 배열이면 배열의 길이를 비교 후 배열의 각 요소를 **`deepEquals`함수로 재귀적 비교**
6. 두 객체의 key 목록을 가져와 키 갯수가 다르다면 `false`를 반환
7. objA의 key를 순회하여 objB에 key가 존재하는지 확인하고, **키의 값이 재귀적으로 동일한지 `deepEquals`함수로 비교**

```typescript
export function deepEquals(objA: unknown, objB: unknown): boolean {
  if (objA === objB) return true;

  if (
    typeof objA !== "object" ||
    objA === null ||
    typeof objB !== "object" ||
    objB === null
  )
    return false;

  const isArrayA = Array.isArray(objA);
  const isArrayB = Array.isArray(objB);

  if (isArrayA !== isArrayB) return false;

  if (isArrayA && isArrayB) {
    if (objA.length !== objB.length) return false;

    return objA.every((Value, index) => deepEquals(Value, objB[index]));
  }

  const keysA = Object.keys(objA);
  const KeysB = Object.keys(objB);

  if (keysA.length !== KeysB.length) return false;

  for (const key of keysA) {
    if (!Object.prototype.hasOwnProperty.call(objB, key)) return false;

    if (
      !deepEquals(
        (objA as Record<string, unknown>)[key],
        (objB as Record<string, unknown>)[key],
      )
    )
      return false;
  }

  return true;
}
```
**=> 깊은 비교는 재귀적 호출을 사용하여 중첩 구조를 비교하고,
이를 통해 중첩된 객체, 배열까지 포함한 모든 깊이를 비교할 수 있습니다.**

> 🤟 기존 지식의 재발견/심화

#### useRef
- 주요 특징
  - 렌더링에 필요하지 않은 값을 참조할 수 있습니다.
  - DOM 요소에 접근하거나 이전 상태를 저장하는 용도로 쓰입니다.
- `useRef(initialValue)`
  - `initialValue`: ref 객체의 `current` 프로퍼티의 초기 설정값으로 어떤 유형의 값이든 지정할 수 있으며, 이 인자는 초기 렌더링 이후부터 무시됩니다.
  - `current`: useRef의 단일 프로퍼티로 다른 값으로 변경할 수 있습니다.

#### useMemo
- 주요 특징
  - 재렌더링 사이에 계산 결과를 캐싱해줍니다.
  - 불필요한 재계산을 방지하고 성능을 최적화할 수 있습니다.
  - 의존성 배열의 값이 변경되지 않는 한, 이전에 계산된 값을 재사용합니다.
  - 초기 렌더링에서는 계산 결과를 반환하고, 다음 렌더링에서는 저장된 값을 반환하거나(종속성이 변경되지 않은 경우), 계산 함수를 다시 호출하고 반환된 값을 저장합니다.
- `useMemo(calculateValue, dependencies)`
  - `calculateValue`: 계산 함수, 순수해야 하며 인자를 받지 않고 모든 타입의 값을 반환할 수 있어야 합니다.
  - `dependencies`: 계산 함수 내에서 참조된 모든 반응형 값의 목록

#### useCallback
- 주요 특징
  - 리렌더링 간에 함수 정의를 캐싱해줍니다.
  - 자식 컴포넌트의 콜백을 전달할 때 사용합니다.
  - 최초 렌더링에서는 함수를 그대로 반환하고, 다음 렌더링에서는 저장된 함수를 반환하거나(의존성이 변하지 않은 경우), 렌더링 중에 전달한 함수를 그대로 반환합니다.
- `useCallback(fn, dependencies)`
  - `fn`: 캐싱할 함수값
  - `dependencies`: 함수 내에서 참조된 모든 반응형 값의 목록

> 🎈 구현 과정에서의 기술적 도전과 해결

타입스크립트 사용이 익숙치 않아 `shallowEquals` 와 `deepEquals` 함수를 구현할 때 타입 오류가 많이 났습니다. (objA와 objB 인자)

처음에는 `deepEquals` 의 함수 타입을 아래와 같이 줬습니다.<br/>
`<T extends Record<string, unknown>>(objA: T, objB: T): boolean`

그러자 재귀적으로 배열의 키를 비교하는 과정에서 타입 오류가 발생했습니다.<br/>
`'unknown' 형식의 인수는 'Record<string, unknown>' 형식의 매개 변수에 할당될 수 없습니다`

그래서 `deepEquals` 함수의 매개변수 타입을 더 일반화해서 `unknown` 값을 받을 수 있도록 수정하였습니다.

- AS-IS
```typescript
export function deepEquals<T extends Record<string, unknown>>(objA: T, objB: T): boolean {
 // 배열 확인 부분 코드 생략
 ...
 
  const keysA = Object.keys(objA);
  const KeysB = Object.keys(objB);

  if (keysA.length !== KeysB.length) return false;

  for (const key of keysA) {
    if (!Object.prototype.hasOwnProperty.call(objB, key)) return false;

    **if (!deepEquals(objA[key], objB[key])) return false; // 타입 오류 발생**
  }

  return true;
}
```

- TO-BE
```typescript
export function deepEquals<T extends Record<string, unknown>>(objA: T, objB: T): boolean {
 // 배열 확인 부분 코드 생략
 ...
 
  const keysA = Object.keys(objA);
  const KeysB = Object.keys(objB);

  if (keysA.length !== KeysB.length) return false;

  for (const key of keysA) {
    if (!Object.prototype.hasOwnProperty.call(objB, key)) return false;

    **if (
	    !deepEquals(
	      (objA as Record<string, unknown>)[key],
	      (objB as Record<string, unknown>)[key],
	    )
    )
    return false;**
  }

  return true;
}
```

### 코드 품질

> ✨ 특히 만족스러운 구현

#### memo
1. 컴포넌트가 렌더링되면 useRef로 이전 props와 생성된 컴포넌트를 저장
2. 다음 렌더링 시 이전 props와 현재 props를 비교
3. props가 동일하면 새로 컴포넌트를 생성하지 않고 기존 컴포넌트를 반환
4. props가 달라지면 새로운 컴포넌트를 생성하여 저장하고 반환

```typescript
import { shallowEquals } from "../equalities";
import { ComponentType, createElement, ReactElement, useRef } from "react";

export function memo<P extends object>(
  Component: ComponentType<P>,
  _equals = shallowEquals,
) {
  const MemoizedComponent = (props: P) => {
    const prevPropsRef = useRef<{ component: ReactElement; props: P } | null>(
      null,
    );

    if (!prevPropsRef.current || !_equals(prevPropsRef.current.props, props)) {
      prevPropsRef.current = {
        component: createElement(Component, props),
        props,
      };
    }

    return prevPropsRef.current.component;
  };

  return MemoizedComponent;
}
```

> ⚡️ 리팩토링이 필요한 부분

심화과제를 수행할 때 context로 관심사를 분리해 봤는데,<br/>
기본적인 부분만 구현하여서 기능이나 성능 측면에서 좀 더 고민해보고 코드를 더 디벨롭 해보고 싶습니다. 

### 학습 효과 분석

> 🌈 가장 큰 배움이 있었던 부분

#### useContext

- 주요 특징
  - Context는 앱 안에서 전역적으로 사용되는 데이터를 여러 컴포넌트끼리 쉽게 공유할 수 있는 방법을 제공합니다.
  - Context를 사용하면 Props로 데이터를 일일이 전달해 주지 않아도 해당 데이터를 가지고 있는 상위 컴포넌트에 그 데이터가 필요한 하위 컴포넌트가 접근할 수 있습니다.
  - 특히 사용자 정보, 테마, 언어 등 전역적인 데이터를 전달할 때 편리합니다.
- `useContext(someContext)`
  - `someContext`: `createContext`로 생성한 context로 컴포넌트에서 제공하거나 읽을 수 있는 정보의 종류를 나타냅니다.

> ❤️‍🔥 추가 학습이 필요한 영역

#### 타입 스크립트

**1. 타입스크립트의 역할**
- 런타임에서 발생할 오류를 컴파일 단계에서 표시해준다.
- 의도와 다르게 작성된 코드를 에러 표시 해준다.

**2. 타입 시스템**
- 구조적 서브 타이핑 (structural sub typing)
  - 속성 기반 타이핑
  - 타입 이름이 달라도 같은 타입으로 인식될 수 있다.
  - 같은 속성을 가지기만 한면 같은 타입으로 인식될 수 있다.
  - sub type ⊂ super type
  - 타입을 집합 관점에서 바라봐야 한다.
  - 타입은 곧 할당 가능한 값들의 집합
```
// 1. 집합 중 가장 작은 집합은?
// never

// 2. 집합의 원소가 하나인 타입은? 
// literal
type One = 2025
type One = true

// 3. 집합의 원소가 2개인 타입은?
// Onion으로 정의
type Two = 2025 | "Jan"
type Two = true | false

// 4. 그 다음 큰 집합들
// string, number, object ....

// 5. 모든 타입의 상위 집합인 타입은?
// unknown
let unknownType: unknown;
unknownType = 1;
unknownType = "typescript";
unknownType = true;
unknownType = { name: "react" };
unknownType = () => {};
```

- 잉여 속성 체크(excess property check)
  - 구조적 서브 타이핑을 거스른다.
  - 정의한 속성 이외에 추가적인(잉여) 속성이 있는지 체크하고, 있다면 에러를 띄운다.
  - 속성 이름의 오타와 같은 실수를 잡아준다.

- 어떤 경우에 구조적 서브 타이핑이 적용 되지 않고, 잉여 속성 체크가 수행하는가?
  - 객체 리터럴을 사용할 때!
```
// 객체 리터럴 => 잉여 속성 체크 수행
return({
  title: "제목",
  darkmode: true,
});

// 변수 할당 => 구조적 서브 타이핑
const options = {
  title: "제목",
  darkmode: true,
};
returnOptions(options);
```

- `Type A is not assignable to Type B`
  - 구조적 서브 타이핑에 의한 타입 에러 => Type A가 Type B의 서브 타입이 아니다.
  - 잉여 속성 체크에 의한 타입 에러 => Type A에 잉여 속성이 있다.

- any
  - any는 모든 타입에 할당이 가능하다. => any는 최하위 집합이다.
  - 모든 타입이 any에 할당이 가능하다. => any는 최상위 집합이다.
  - any는 기본적으로 타입 시스템을 따르지 않는다.
  - 어떤 타입이 들어올지 모르는 경우나 모든 타입이 들어올 수 있는 경우는 `any`가 아닌 `unknown`을 사용하는 것이 적절하다.

**3. 타입 스크립트를 더 잘 쓰는 방법**
- 함수의 반환타입을 명시하여 의도를 표현하기
  - 타입 추론에 의존하지 않고 의도 타입으로 명시한다. 
```typescript
const addZero = (num: number): string => {
  return Math.floor(num / 10) === 0 ? 0${num} : String(num);
}
```
- 규별된 유니온 사용하기
  - 유니온의 인터페이스 보다는 인터페이스의 유니온을 사용하라
```typescript
export declare type QueryObserverResult<TData = unknown, TError = unknown> = 
  |  QueryObserverIdleResult<TData, TError>,
  |  QueryObserverLoadingErrorResult<TData, TError>,
  |  QueryObserverLoadingResult<TData, TError>,
  |  QueryObserverRefetchErrorResult<TData, TError>,
  |  QueryObserverSuccessResult<TData, TError>,
```

- any 잘 쓰기
  - 함수 안으로 any를 감추고, 반환 타입만 잘 명시해두면 any가 전파되지 않는다.
```typescript
const parseMember = (
  member: Member[]
): Record<"frontend" | "backend", Member[]>  =>  {
  return members.reduce((prev, member)) => {
    const key = member.type;
    return {...prev, [key]: [...(prev as []) [key as any], member]};
  }, {}) as any;
};
```

> 🐾 실무 적용 가능성

실무에서는 대부분 유지보수를 하는 프로젝트에서 타입스크립트를 사용했기에 타입 선언 및 지정 등 간단하고 기본적인 것만 알고 있었고,<br/>
타입 유틸리티나 타입 단언, 추론 등 심화 부분은 아직 숙지가 안된 상태입니다. 

이번 과제를 통해 다양한 유틸리티 타입들을 접하고 사용해 볼 수 있어서 타입스크립트에 조금 가까워졌다는 느낌도 받았고,<br/>
타입스크립트를 좀 더 공부해야겠다는 깨달음을 얻은 계기가 되었습니다.

### 과제 피드백

> 🌝 과제에서 좋았던 부분

평소에 실무에서 React Hooks를 자주 사용하고 있지만 100% 이해하고 사용하는건 아니였습니다.<br/>
이번 과제가 useRef, useMemo, useCallBack 등 리액트에서 제공해주는 Hooks를 직접 커스텀 훅으로 구현하는 것이기 때문에<br/>
기존 Hooks 개념 등 기본기의 중요성을 한번 더 깨달았습니다.

더 나아가 리액트의 참조 동일성 방법, 불변성을 사용하는 방식, 메모이제이션 기법 등 좀 더 깊이 있는 학습 필요성을 얻게된 계기가 되었습니다.

## 리뷰 받고 싶은 내용

### 1. 성능 프로파일링 확인 방법

이번 과제 때 성능 프로파일링, 렌더링 비용 등의 개념을 배우고 혼자 확인해 보기 위해 실행 해보았습니다.<br/>
어떤 컴포넌트가 렌더링 되고 렌더링 순서나 단계는 직관적으로 확인할 수 있었지만,

정확히 어떤 컴포넌트에서 렌더링 비용이 많이 발생하는 건지,<br/>
어떤 식으로 그래프가 떠야 비용이 많이 발생해 성능에 이슈가 있는 건지,<br/>
성능 최적화 후 실행 했을 때 어떠한 렌더링 변화가 생기는 지 등<br/>
정확한 성능 프로파일링을 확인하고 사용하는 방법을 잘 모르겠습니다.

이에 관련하여 자세히 설명된 자료나 영상 등이 있으면 추천 부탁드립니다!

### 2. Context Provider 선언 순서

현재 App에 각 Context Provider를 아래와 같은 순서대로 감쌌습니다.

`NotificationProvider` > `ThemeProvider` > `UserProvider`

컴포넌트의 순서를 조금 다르게 구성하면 오류가 나서 위와 같은 순서로 하니 정상 작동이 되었습니다.

감싸는 순서가 단순히 UI적으로 보여지는 측면일까요?<br/>
아니면 어떤 이유로 위와 같이 구현해야지만 정상 작동하는지 궁금합니다.

```typescript
import {
  ComplexForm,
  Header,
  ItemList,
  NotificationSystem,
} from "./components";
import { NotificationProvider, ThemeProvider, UserProvider } from "./context";

const App: React.FC = () => {
  return (
    <NotificationProvider>
      <ThemeProvider>
        <UserProvider>
          <div className="min-h-screen">
            <Header />
            <div className="container mx-auto px-4 py-8">
              <div className="flex flex-col md:flex-row">
                <div className="w-full md:w-1/2 md:pr-4">
                  <ItemList />
                </div>
                <div className="w-full md:w-1/2 md:pl-4">
                  <ComplexForm />
                </div>
              </div>
            </div>
            <NotificationSystem />
          </div>
        </UserProvider>
      </ThemeProvider>
    </NotificationProvider>
  );
};

export default App;
```
