제네릭을 사용하기 시작하면 수많은 컴파일러 경고를 볼 수 있음.<br>
컴파일러 경고 예시) 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등

- 자바 7부터 지원하는 다이아몬드 연산자(<>)를 사용하면 타입 매개변수를 추론 가능. 
- 할 수 있는 한 비검사 경고를 제거해서 ClassCastException이 발생하지 않도록 하기.

## @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기자.

- 단, 타입이 안전한 지 확인한 후 달기.
- @SuppressWarnings는 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에서도 달 수 있다. 
- <strong>하지만 가능한 좁은 범위에서 적용하라</strong>
	- 한 줄이 넘는 메서드가 생성자에 이 애너테이션이 달린다면 지역 변수 선언쪽으로 옮겨라.
- 참고) [@SuppressWarnings](https://www.ibm.com/docs/ko/developer-for-zos/9.5.1?topic=code-excluding-warnings)    

#### ArrayList의 toArray 메서드

```

    public <T> T[] toArray(T[] a) {
        if (a.length < size
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

```

#### 지역 변수를 추가해 @SuppressWarnings의 범위를 좁힌다

```

    public <T> T[] toArray(T[] a) {
        if (a.length < size) {
            // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같다.
            @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elementData, size, a.getClass());
            return result;
        }
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

```

- @SuppressWarnings("unchecked") 애너테이션을 사용할 땐 그 경고를 무시해도 안전한 이유를 항상 주석으로 남길 것.

### 핵심 정리

- 비검사 경고는 중요하니 무시하지 말기.
- 모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있음.
- 경고를 없앨 방법을 찾지 못하겠다면, 그 코드가 타입 안점함을 증명하고 가능한 한 범위를 좁혀 @SuppressWarnings("unchecked") 애터네이션으로 경고를 숨기기.
- 숨기기로 한 이유를 주석으로 남기기.
