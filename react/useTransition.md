# useTransition 살펴보기

useTransition은 리액트 18 이후에 도입된 주목할만한 훅이다. useTransition은 개발자가 상태 업데이트의 우선순위를 조정함으로써 사용자 경험을 향상시키고, 성능 문제를 해결하는데 도움을 주도록 한다.

리액트에서는 상태 업데이트들이 동일한 우선순위를 가진다. 따라서 상태들의 업데이트는 연산 처리 속도와 무관하게 일괄적(batching)으로 처리된다. 그렇기에 상황에 따라서 불필요한 UI 블로킹이 발생할 수 있으며 이는 결과적으로 UX를 저해하게 된다.

**useTranstion의 핵심적인 역할은 무거운 상태 업데이트를 낮은 우선순위로 처리하여, 사용자 중심의 UX 설계에 도움을 주는 것이다.** 앞선 상태 일괄처리로 인한 UI 블로킹의 상황에서, useTransition을 적용한다면 끊김없고 자연스러운 UX에 도움을 줄 수 있다.

## useTransition을 사용하는 상황

useTransition은 아래와 같이, 무거운 연산으로인해 UI 블로킹이 발생할 수 있는 상황들에서 사용할 수 있다.

> - 검색 자동완성 UI - 검색 결과가 많고 필터링이 복잡할수록 타이핑이 버벅임
> - 대량 데이터 기반의 페이지 전환 - 클릭과 동시에 대량 데이터 렌더링으로 인해 발생하는 일시정지 현상
> - 게시글 정렬 기능 - 정렬 연산이 느리면 버벅임 발생

위의 상황에서 '타이핑', '페이지/정렬 버튼' 클릭에 이어지는 무거운 연산에 useTransition을 적용하여 보다 자연스러운 UX를 설계할 수 있다.

## useTransition과 검색 자동완성 UI

검색 자동완성 UI와 같은 대표적인 상황에 useTransition을 적용할 수 있다. 해당 상황에서 타이핑 이벤트 처리와 함께, 검색어 필터링까지 함께 처리되어 버벅거리는 현상이 발생할 수 있다.

### 코드

타이핑을 하면 수많은 검색 결과를 보여주는 UI 코드가 있다고 가정하자. 이때 다량의 검색 결과를 업데이트하는 로직에 useTransition을 적용하여, 검색어 타이핑 이벤트에 걸리는 UI 블로킹을 제거할 수 있다.

```javascript
import React, { useState, useTransition } from "react";

function SearchingApp() {
  const [input, setInput] = useState("");
  const [list, setList] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    const value = e.target.value;
    setInput(value);

    // 검색어 필터링(무거운 연산)을 transition 안에서 처리
    startTransition(() => {
      const newList = Array.from(
        { length: 10000 },
        (_, i) => `${value} - ${i}`
      );
      setList(newList);
    });
  };

  return (
    <div>
      <input type="text" value={input} onChange={handleChange} /> // 검색어 입력
      {isPending && <p>Loading...</p>}
      <ul>
        {" "}
        // 검색 결과 정렬
        {list.map((item, i) => (
          <li key={i}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 설명

- useTransition은 `[isPending, startTransition]`의 형태로 사용된다.
- `startTransition( () => {...} )` 안에 무거운 연산에 대한 상태 업데이트를 작성한다 -> 낮은 우선순위로 처리
- `isPending`은 transition중일 때 `true`가 되며, 로딩 UI를 표시할 수 있다.

## 마치며

useTransition은 단순히 성능 최적화를 위한 도구를 넘어, 사용자 중심의 UX를 설계하는 데 중요한 역할을 한다. React 18 이후로 도입된 동시성 기능들은 개발자에게 "어떻게 보일 것인가"뿐 아니라 "어떻게 반응할 것인가"에 대한 고민을 던진다. 그 흐름 속에서 useTransition은 무거운 연산을 부드럽게 처리하고, 불필요한 UI 블로킹을 줄임으로써 더 나은 사용자 경험을 설계할 수 있게 돕는다. 무거운 UI를 다루는 프로젝트에서, 이 훅을 적절히 활용하면 충분히 큰 도움이 될 것이라 생각한다.
