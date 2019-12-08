# JAVA 8 Stream API

### 스트림과 컬렉션의 차이

1. 스트림은 요소들을 보관하지않음. 요소들은 하부의 컬렉션에 보관되거나 필요할 때 생성된다.

2. 스트림 연산은 원본을 변경하지않는다. 대신 결과를 담은 새로운 스트림을 반환한다.

3. 스트림 연산은 가능하면 지연 처리된다. 

   `지연처리`란 결과가 필요하기 전에는 실행되지않음을 의미한다.



### 병렬화

`stream`을 `parallenStream`으로 변경하는 것만으로 스트림 라이브러리가 필터링과 카운팅을 병렬로 수행 할 수 이다.

```java
long count = words.stream().filter(w -> w.length() > 12).count();
long count = words.parallelStream().filter(w -> w.length() > 12).count();
```

parallenStream 목적은..?



### Stream API 3단계

1. 스트림 생성 

2. 초기 스트림을 다른 스트림으로 변환하는 중간 연산들

3. 결과를 산출하기 위한 최종 연산

   이 연산은 앞선 지연 연산들의 실행을 강제한다. 이후 해당 스트림은 더는 사용 할 수 없다.





### flatMap()

스트림들을 하나의 스트림으로 합쳐서 하나의 새로운 스트림을 반환.

```java
Stream<Character> flatMapStream = stream.flatMap(w -> characterStream(w));

private Stream<Character> characterStream(String s) {
		List<Character> result = new ArrayList<>();
		for (char c : s.toCharArray()) {
			result.add(c);
		}
		return result.stream();
	}
```



### 리덕션

리덕션 메소드는 스트림을 프로그램에 사용할 수 있는 값으로 리듀스한다.

리덕션은 최종연산이다. 

이들 메서드는 전체 스트림을 검사하지만 여전히 병렬 실행(`parallel()`)을 통해 이점을 얻을 수 있다.

1. stream.count()
2. stream.max()
3. stream.min()
4. stream.findFirst()
5. stream.findAny()
6. stream.anyMath() //스트림에서 일치하는 요소가 있는지 여부를 반환



#### 리덕션 연산

##### Stream.reduce()

스트림의 요소들을 다른 방법으로 결합하고 싶은 경우 reduce 메소드를 사용

여러 개가 있는데 가장 단순한 형태는 이항 함수를 받아 처음 두 요소부터 시작하여 계속해서 해당 함수를 적용한다.



reduce 메서드가 리덕션 연산 op를 가지면, 해당 리덕션은 V0 op V1 op V2 op V3 op … 를 돌려준다. 연산 op는 결합 법칙을 지원해야 한다. 즉 결합하는 순서는 문제가 되지 않아야 한다.

```java
Optional<T> reduce(BinaryOperator<T> accumulator);

Optional<Integer> sum = stream.reduce((x, y) -> x + y);
Optional<Integer> sum = stream.reduce(Integer::sum);
```



e op x = x와 같은 항등값이 존재할 때는 첫번째 인자로 항등값을 넣어줄 수 있다. 그러면 반환 값으로 Optional<T>가 아닌 T를 받을 수 있다.

```java
T reduce(T identity, BinaryOperator<T> accumulator);

int sum = stream.reduce(0, Integer::sum);

//i.e
int rs= Stream.of(1, 2, 3, 4, 5, 6).reduce(-1, Integer::sum);
//output rs=20
```



스트림은 병렬화가 쉽다는 장점이 있는데, 값이 누적되는 연산의 경우에는 대부분 바로 병렬화를 할 수 없다. 이 경우에는 각 부분의 결과를 결합하도록 사용하는 함수를 3번째 인자로 넣어주어야 한다.

```java
<U> U reduce(U identity,
			 BiFunction<U, ? super T, U> accumulator,
			 BinaryOperator<U> combiner);
```



실전에서는 reduce 메소드를 많이 사용하지않을 것이다. 보통은 숫자 스트림에 매핑 한 후에 각 값을 계산해주는 메서드를 이용하는 것이 더 쉽다.



### Stream.forEach(), Stream.forEachOrdered()

forEach의 경우에는 병렬스트림에서 순서를 보장할 수 없다.

스트림 순서대로 조회하고 싶은 경우에는 forEachOrdered를 사용해야 한다. 하지만 이경우는 병렬성이 주는 대부분의 이점을 포기해야 한다.

최종 연산에 해당하므로 스트림을 재사용할 수 없다. 만약 재사용하고 싶다면 peek 메소드를 사용해야한다.

```java
Object[] powers = Stream.iterate(1.,0, p -> p * 2)
	.peek(e -> System.out.println("Fetching " + e))
	.limit(20).toArray();
```

