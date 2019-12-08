## java.net.URLEncoder

이 클래스에는 문자열을 `application / x-www-form-urlencoded` MIME 형식으로 변환하기위한 정적 메소드가 포함되어 있습니다.

인코딩 적용 규칙

- a-z, A-Z, 0-9은 그대로 사용
- `.`,`-`,`*`,`_` 그대로 사용
- 띄어쓰기는 `+`로 변환
- 모든 문자는 안전하지 않으며, 일부 인코딩 체계를 사용하여 먼저 하나 이상의 바이트로 변환된다.

##### 테스트

```
띄어 쓰기 : %EB%9D%84%EC%96%B4+%EC%93%B0%EA%B8%B0
특+수%문&자-테/스*트#입$니@다 : %ED%8A%B9%2B%EC%88%98%25%EB%AC%B8%26%EC%9E%90-%ED%85%8C%2F%EC%8A%A4*%ED%8A%B8%23%EC%9E%85%24%EB%8B%88%40%EB%8B%A4
```



## org.springframework.web.util.UriUtils

 RFC 3986를 기본으로 인코딩,디코딩하는 유틸리티 클래스

다양한 URI component 지원하기 위해 인코딩 메소드를 제공한다.

`encode*(String,String)`메소드들은 다음과 같이 작동한다.

	-  RFC 3986 에 정의된 유효한 문자열들은 그대로 남는다.
	-  모든 문자열들은 일부 인코딩 체계를 사용하여 먼저 하나 이상의 바이트로 변환된다.



##### 테스트

```
# UriUtils.encode
띄어 쓰기 : %EB%9D%84%EC%96%B4%20%EC%93%B0%EA%B8%B0
특+수%문&자-테/스*트#입$니@다 : %ED%8A%B9%2B%EC%88%98%25%EB%AC%B8%26%EC%9E%90-%ED%85%8C%2F%EC%8A%A4%2A%ED%8A%B8%23%EC%9E%85%24%EB%8B%88%40%EB%8B%A4


# UriUtils.encodeQueryParam
띄어 쓰기 : %EB%9D%84%EC%96%B4%20%EC%93%B0%EA%B8%B0
특+수%문&자-테/스*트#입$니@다 : %ED%8A%B9+%EC%88%98%25%EB%AC%B8%26%EC%9E%90-%ED%85%8C/%EC%8A%A4*%ED%8A%B8%23%EC%9E%85$%EB%8B%88@%EB%8B%A4


# UriUtils.encodePathSegment
띄어 쓰기 : %EB%9D%84%EC%96%B4%20%EC%93%B0%EA%B8%B0
특+수%문&자-테/스*트#입$니@다 : %ED%8A%B9+%EC%88%98%25%EB%AC%B8&%EC%9E%90-%ED%85%8C%2F%EC%8A%A4*%ED%8A%B8%23%EC%9E%85$%EB%8B%88@%EB%8B%A4


# UriUtils.encodePath
띄어 쓰기 : %EB%9D%84%EC%96%B4%20%EC%93%B0%EA%B8%B0
특+수%문&자-테/스*트#입$니@다 : %ED%8A%B9+%EC%88%98%25%EB%AC%B8&%EC%9E%90-%ED%85%8C/%EC%8A%A4*%ED%8A%B8%23%EC%9E%85$%EB%8B%88@%EB%8B%A4


# UriUtils.encodeQuery
띄어 쓰기 : %EB%9D%84%EC%96%B4%20%EC%93%B0%EA%B8%B0
특+수%문&자-테/스*트#입$니@다 : %ED%8A%B9+%EC%88%98%25%EB%AC%B8&%EC%9E%90-%ED%85%8C/%EC%8A%A4*%ED%8A%B8%23%EC%9E%85$%EB%8B%88@%EB%8B%A4

```



## org.yaml.snakeyaml.util.UriEncoder

'%'로 특수 문자 피한다?



## PHP의 rawurlencoder

Java URLEncoder는 공백을 `+`로 변환하는데 반해 PHP에는 urlencoder와 rawurlencoder가 있다.

urlencoder는 Java의 URLEncoder처럼 공백을 `+`로 인코딩한다.

rawurlencoder는 `-`,`_`,`,`를 제외하고 모든 영숫가자 아닌 문자를 퍼센트사인에 이어지는 두 16진수로 교체한 문자열을 반환한다.

```java
public static String rawurlencode(String input) {
	try {
		return URLEncoder.encode(input, "UTF-8").replace("*", "%2A").replace("+", "%20").replace("%7E", "~");
	} catch (UnsupportedEncodingException e) {
		return null;
	}
}
```

출처 :  [https://zetawiki.com/wiki/%EC%9E%90%EB%B0%94_rawurlencode()](https://zetawiki.com/wiki/자바_rawurlencode()) 

##### 테스트

```
# UriUtils.encode와 동일
띄어 쓰기 : %EB%9D%84%EC%96%B4%20%EC%93%B0%EA%B8%B0
특+수%문&자-테/스*트#입$니@다 : %ED%8A%B9%2B%EC%88%98%25%EB%AC%B8%26%EC%9E%90-%ED%85%8C%2F%EC%8A%A4%2A%ED%8A%B8%23%EC%9E%85%24%EB%8B%88%40%EB%8B%A4
```



------

### @PathVariable인 파라미터에 '/'가 포함된 경우 에러 발생

경로 변수로 사용하여 디코딩 한 다음에 서버가 경로로 인식하여 매치하려는데 존재하지 않는 경로라 에러가 발생.

-> 보안상의 이유로 슬래쉬를 변수로 허용하지 않음.