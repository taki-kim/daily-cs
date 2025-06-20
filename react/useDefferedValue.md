# useDefferedValue

useDefferedValue는 의도적으로 특정 상태의 렌더링을 지연하는 훅이다.

리액트 프로그래밍을 하며 **값은 즉시 바꾸되, 그 값에 의해 발생하는 무거운 연산이나 렌더링을 느긋하게 처리하고 싶은 경우에 활용**할 수 있다. 이로써 우선순위가 낮은 상태의 렌더링 반응을 지연하여 성능을 최적화하고, 사용자에게 더 쾌적한 UX를 제공할 수 있다.

## useDefferedValue vs useTransition

useDefferedValue는 [지난 포스팅](https://www.takitown.com/post/[React]%20useTransition)에서 다룬 useTransition과 유사한 개념을 가진다. 두 훅 모두 무겁거나 우선순위가 낮은 작업을 지연시켜 UI 블로킹을 줄이고 쾌적한 UX를 설계하는데 목적을 둔다. 그렇기에 두 기능 모두 구체적으로 UX와 성능 측면에서 민감하고 덜 민감한 작업을 분리하는데 공통적으로 사용될 수 있다.

하지만 두 훅은 분명한 차이점이 있다.
**useDeferredValue는 상태 업데이트 함수가 아닌, 상태 값 자체를 지연시킨다.** 즉, 특정 상태 값을 기준으로 그 값에 의한 리렌더링을 React가 느슨하게 처리하도록 설정하는 방식이다.

반면, useTransition은 지연할 작업(상태 업데이트 함수)을 `startTransition()`으로 명시적으로 감싸며, 해당 작업의 우선순위를 낮춰 실행 시점을 뒤로 미룬다.
이처럼 개발자가 '무엇을 지연시킬지' 명확히 제어할 수 있는 점이 특징이다.

요약하면,

- useDeferredValue는 “지연할 상태 값을 넘기고 처리 시점을 React에 맡기는” 자동 제어 방식이고,
- useTransition은 “지연할 작업을 개발자가 명시적으로 지정하는” 수동 제어 방식이다.

따라서, **지연의 대상**과 **제어 방식**의 차이를 이해한 뒤, 의도에 따라 적절한 훅을 선택하는 것이 중요하다.

## useDefferedValue 사용 예시

### 어느 상황에 사용되나

useDefferdValue는 값은 즉시 바꾸되, 그 값에 의해 발생하는 무거운 연산이나 렌더링을 느긋하게 처리하고 싶은 경우에 활용할 수 있다.
주로 입력창, 버튼과 같이 바로바로 반응되어야할 이벤트와 그로 인해 이어지는 무거운 작업들을 분리할 때 사용할 수 있다.

> - 검색 입력과 결과 필터링
> - 카테고리 버튼 클릭과 결과 필터링

### 검색 입력과 결과 필터링 예시

검색창에 텍스트를 입력하면, 그 결과와 상응하는 다량의 데이터를 필터링 하는 기능을 가정하자. 이때 빠르게 업데이트 되는 검색결과와 동일한 우선순위로 다량의 데이터를 필터링하는 로직이 작동하면 심한 버벅거림이 발생한다.

이러한 문제를 useDefferedValue를 사용하여 지연된 버전의 입력값을 지정하고, 필터링 로직의 수행을 최적화하여 해결하고자 한다.

#### 코드

```javascript
import React, { useState, useDeferredValue, useMemo } from "react";

// 부모 컴포넌트에서 items 배열을 props로 전달
function SearchWithDeferredValue({ items }: { items: string[] }) {
  const [input, setInput] = useState("");
  const deferredInput = useDeferredValue(input); // 입력값의 지연된 버전

  // deferredInput이 바뀔 때만 필터링 수행
  const filteredList = useMemo(() => {
    console.log("🔍 Filtering with:", deferredInput);
    if (!deferredInput) return items;
    return items.filter((item) =>
      item.toLowerCase().includes(deferredInput.toLowerCase())
    );
  }, [items, deferredInput]);

  return (
    <div>
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="검색어 입력"
      />
      <p>입력값: {input}</p>
      <p>지연된 입력값: {deferredInput}</p>

      <ul style={{ maxHeight: 200, overflowY: "auto" }}>
        {filteredList.map((item, i) => (
          <li key={i}>{item}</li>
        ))}
      </ul>
    </div>
  );
}

export default SearchWithDeferredValue;
```

#### 설명

- items(다량의 데이터) 배열은 props로 전달 받음
- useDefferedValue를 통한 `defferedInput` 분리
- useMemo로 무거운 연산 최적화하고, 불필요한 연산을 방지
- `defferedInput`을 통하여 자동으로 업데이트가 지연되며, 필터링 로직이 수행횟수 최적화

## 마치며

useDeferredValue는 빠르게 반응해야 하는 UI와, 성능 부담이 큰 연산을 자연스럽게 분리함으로써, 사용자 경험을 끊김 없이 유지할 수 있는 효과적인 도구다.
상황에 맞는 적절한 우선순위 조절은 리액트 성능 최적화의 핵심이며, useDeferredValue는 그 흐름 속에서 매우 유용한 선택지가 될 것이라 생각한다.
