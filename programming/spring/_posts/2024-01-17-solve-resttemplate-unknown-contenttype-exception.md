---
title: RestTemplate에서 UnknownContentTypeException 오류 해결하기
# toc: true
# toc_label: "Contents"
# toc_sticky: true
date: 2024-01-17
tags:
  - RestTemplate
  - Problem Solving
  - HttpMessageConverter
  - UnknownContentTypeException
---
### 선 요약
외부 Rest API 서버의 응답의 *ContentType*이 *html/text*이며 본문(Body)이 *json*인 경우. 
1. *MappingJackson2HttpMessageConverter*에 처리할 수 있는 ContentType으로 *html/text*을 추가한다.
2. 위의 1.에서 만든 *MappingJackson2HttpMessageConverter*를 *RestTemplate*에 추가한다.
3. 그러면 *ResteTemplate*에서 헤더의 ContentType이 *html/text*이고, 본문이 *json*인 응답을 변환할 수 있다.

# 오류 발생 상황
## 배경
토이 프로젝트에서 외부 Rest API를 호출하는 기능을 구현했습니다. 내부 구현으로 *RestTemplate*을 사용해서 외부 API를 호출합니다. 아래와 같이 응답을 *String.class*로 받을때는 정상동작 합니다.  


```java
// String.class을 ResponseType인자로 사용할 때 정상 동작 코드
String response = restTemplate
    .postForEntity(URL, HttpEntity, String.class).getBody();

System.out.println(response);
// {"success":true,"status":"normal","api_ver":"2.6","action":"info"}
```

응답을 새로운 DTO클래스(*CallInfoResponse*)로 받고 싶어서 DTO클래스를 추가합니다. 그리고 응답을 *CallInfoResponse.class*로 받게 아래와 같이 리팩토링을 합니다.

```java
@Getter @ToString
public class CallInfoResponse {
  private boolean success;
  private String status;
  private String api_ver;
  private ActionType action;
  private String error;
  
  @Override
  public boolean equals(Object o) { ... }
  @Override
  public int hashCode() { ... }
}
```
```java
// CallInfoResponse.class을 ResponseType인자로 사용하도록 리팩토링
CallInfoResponse response = restTemplate
    .postForEntity(URL, HttpEntity, CallInfoResponse.class).getBody();
// UnknownContentTypeException 발생
```
그러나 response 변수에 응답 값(Json)은 바인딩되지 않고 다음 오류가 발생합니다.
## 로그 메시지
```text
org.springframework.web.client.UnknownContentTypeException: Could not extract response: no suitable HttpMessageConverter found for response type [class io.github.kimseunghyunbg.dto.CallInfoResponse] and content type [text/html;charset=UTF-8]
  at org.springframework.web.client.HttpMessageConverterExtractor.extractData(HttpMessageConverterExtractor.java:126) ~[spring-web-5.3.17.jar:5.3.17]
  at org.springframework.web.client.RestTemplate$ResponseEntityResponseExtractor.extractData(RestTemplate.java:1037) ~[spring-web-5.3.17.jar:5.3.17]
  at org.springframework.web.client.RestTemplate$ResponseEntityResponseExtractor.extractData(RestTemplate.java:1020) ~[spring-web-5.3.17.jar:5.3.17]
  at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:778) ~[spring-web-5.3.17.jar:5.3.17]
  at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:711) ~[spring-web-5.3.17.jar:5.3.17]
  at org.springframework.web.client.RestTemplate.postForEntity(RestTemplate.java:468) ~[spring-web-5.3.17.jar:5.3.17]
```
<br>

# 원인 분석 1
## 가설 1
**"DTO 클래스 개발이 잘못되어서 응답 값(Json) 바인딩에 실패한다."**

위 가설은 단위테스트를 작성해서 확인해 보겠습니다.  
*RestTemplate*는 *Json*을 변환할 때 *Jackson* 라이브러리를 기본으로 사용합니다. 그래서 Jackson 라이브러리를 사용해서 응답 값(Json)을 DTO클래스(CallInfoResponse)에 바인딩하는 테스트를 합니다.
## 가설 1 검증 
```java
// 단위테스트
import static org.assertj.core.api.Assertions.assertThat;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.github.kimseunghyunbg.ActionType;
import java.io.IOException;
import org.junit.jupiter.api.Test;

class CallInfoResponseTest {
  @Test
  public void testDeserialize() throws IOException {
    //given
    final String json = """
        {"success":true,"status":"normal","api_ver":"2.6","action":"info"}""";
    //when
    CallInfoResponse actual = new ObjectMapper().readValue(json, CallInfoResponse.class);
    //then
    assertThat(actual.isSuccess()).isTrue();
    assertThat(actual.getStatus()).isEqualTo("normal");
    assertThat(actual.getApi_ver()).isEqualTo("2.6");
    assertThat(actual.getAction()).isEqualTo(ActionType.INFO);
    assertThat(actual.getError()).isNull();
  }
}
```
테스트결과 응답 값(Json)이 DTO클래스(CallInfoResponse)에 바인딩이 됩니다. **가설 1 은 틀렸습니다.**
<br>

# 원인 분석 2
## 가설 2

**"ContentType 이 text/html이기 때문에 오류가 발생한다."**

로그를 다시 한번 봅니다.

```text
org.springframework.web.client.UnknownContentTypeException: Could not extract response: no suitable HttpMessageConverter found for response type [class io.github.kimseunghyunbg.dto.CallInfoResponse] and content type [text/html;charset=UTF-8]
```

*ResponseType*과 *ContentType*을 위한 적절한 *MessageConverter*를 찾을 수 없다고 합니다.

앞서 *가설 1 검증*에서 *ResponseType*(CallInfoResponse.class)이 적절하게 개발되어서, 응닶 값(Json)이 바인딩되는 것을 확인했습니다. 그래서 *ResponseType*은 문제가 없습니다.

다음으로 *ContentType을* 살펴보겠습니다. **ContentType이란 HTTP의 본문의 원본 타입이 무엇인지 알려주는 역할을 하는 HTTP의 헤더(Header)입니다.** *ContentType*이 *text/html*라니 무엇인가 이상합니다. json값을 주고 받을때 ContentType은 보통 *application/json* 이었습니다.

이제 위 가설을 확인해보죠.

## 가설 2 검증을 위한 사전 조사
오류 로그를 따라서 *HttpMessageConverterExtractor*의 *extractData*메소드를 추적해봅니다.
```java
// 설명을 위해 소스코드를 축약했습니다.
public class HttpMessageConverterExtractor<T> implements ResponseExtractor<T> {
  
  private final List<HttpMessageConverter<?>> messageConverters;

  public T extractData(ClientHttpResponse response) throws IOException {
    MessageBodyClientHttpResponseWrapper responseWrapper = new MessageBodyClientHttpResponseWrapper(response);
    
    // 1. 응답 헤더의 contentType
    MediaType contentType = getContentType(responseWrapper);

    try {
      // 2. 여기서부터 MessageConverter가 contentType를 읽을 수 있는지 판별하는 구문.
      for (HttpMessageConverter<?> messageConverter : this.messageConverters) {
        if (messageConverter instanceof GenericHttpMessageConverter) {
          GenericHttpMessageConverter<?> genericMessageConverter =
              (GenericHttpMessageConverter<?>) messageConverter;
          // 3. MessageConverter가 contentType를 읽을 수 있는지 판단.
          //    읽을 수 있으면 return, 없으면 pass.
          if (genericMessageConverter.canRead(this.responseType, null, contentType)) {
            return (T) genericMessageConverter.read(this.responseType, null, responseWrapper);
          }
        }
        if (this.responseClass != null) {
          // 3. MessageConverter가 contentType를 읽을 수 있는지 판단.
          //    읽을 수 있으면 return, 없으면 pass.
          if (messageConverter.canRead(this.responseClass, contentType)) {
            return (T) messageConverter.read((Class) this.responseClass, responseWrapper);
          }
        }
      }
    }
    catch (IOException | HttpMessageNotReadableException ex) {
      throw new RestClientException("Error while extracting response for type [" +
          this.responseType + "] and content type [" + contentType + "]", ex);
    }

    // 4. UnknownContentTypeException 발생 부분.
    throw new UnknownContentTypeException(this.responseType, contentType,
        responseWrapper.getRawStatusCode(), responseWrapper.getStatusText(),
        responseWrapper.getHeaders(), getResponseBody(responseWrapper));
  }
}
```
위 소스코드를 제가 붙인 주석을 따라서 필요한 부분만 읽어봅시다.
1. getContentType 메소드를 사용해 응답(responseWrapper)에서 *ContentType*을 가져옵니다.
2. messageConverters에서 *messageConverter*를 하나씩 꺼내는 반복문을 실행합니다.
3. 위에 *2.* 에서 꺼낸 messageConverter가 if문에 따라 어느쪽으로 가든지 *canRead*메소드를 실행합니다. 여기서 messageConverter가 *responseType*과 *contentType*을 읽을 수 있는지 확인합니다. 그리고 읽을 수 있다면 read메소드를 실행해서 결과를 반환(return) 합니다.
4. 만약에 모든 messageConverter가 *3.* 조건에 부합하지 않으면 *UnknownContentTypeException*를 발생시킵니다. 오류 로그의 마지막은 이 부분을 가르키고 있습니다.

## 가설 2 검증
디버깅해서 RestTemplate내부의 *messageConverters*가 처리할 수 있는 *ContentType*를 확인합니다.

![RestTemplate messageConverters](/assets/images/posts/resttemplate_messageconverter.png)

RestTemplate에는 messageConverter가 총 7개 있습니다. 각각의 *messageConverter*는 처리 할 수 있는 *ContentType*목록을 가지고 있습니다.
- 첫 번째 *messageConverter*는 *ByteArrayHttpMessageConverter*입니다. 그리고 처리 할 수 있는 *ContentType*으로 *application/octet-stream*, *\*/\** 가 있습니다.

제가 필요한 *messageConverter*는 7번째(index:6)에 있는 *MappingJackson2HttpMessageConverter*입니다.
- *MappingJackson2HttpMessageConverter*가 처리 할 수 있는 *ContentType*으로는 *application/json*, *application/+json* 두 가지가 있습니다.

종합해보면 응답의 *ContentType*이 *text/html*여서 Json을 변환하는 *MappingJackson2HttpMessageConverter*가 Json 응답을 변환하지 못한 겁니다.

## 해결 방안 모색

### 해결 방안 1
"응답을 String으로 받은 다음에 DTO 클래스 생성자의 파라미터로 넘기는 방법은 어떨까?"
- 고민사항
  - DTO마다 Json변환 로직을 넣어야 합니다. **DTO는 데이터 전송 객체라는 책임과 역할**에서 크게 벗어나 보이진 않습니다. 하지만 로직이 DTO에 들어갈수록 개발자는 다른 DTO에도 로직을 넣기가 심리적으로 쉽습니다. **결국엔 책임과 역할을 벗어나는 DTO 탄생의 시발점이 될 수 있습니다.**

### 해결 방안 2
"MappingJackson2HttpMessageConverter을 이용해서 ContentType이 text/html인 Json도 처리하는 방법은 없을까?"
- 고민사항
  - 첫 번째로 *RestTemplate*에 *messageConverter*를 추가하는 방법이 있어야 합니다.
  - 두 번째로 *MappingJackson2HttpMessageConverter*에 *ContentType*을 추가하는 방법이 있어야 합니다.

## 문제 해결
소스코드의 일관성과 중복을 고려해서 **해결 방안2**로 문제를 해결합니다. 아래는 *RestTemplate Bean*에 추가한 코드 입니다.
```java
@Bean
public RestTemplate restTemplate() {
  RestTemplate restTemplate = new RestTemplate();

  // 해결 방안2 를 적용한 부분 ///////////////////////
  MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
  converter.setSupportedMediaTypes(Collections.singletonList(MediaType.TEXT_HTML));
  restTemplate.getMessageConverters().add(converter);
  /////////////////////////////////////////////////

  HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();
  CloseableHttpClient httpClient = HttpClientBuilder.create()
      .setRedirectStrategy(new LaxRedirectStrategy()).build();
  factory.setHttpClient(httpClient);
  restTemplate.setRequestFactory(factory);
  return restTemplate;
}
```
위와 같이 코드를 수정하면 *RestTemplate*는 기존 7개의 messageConverter에 *MappingJackson2HttpMessageConverter* 1개가 더 추가되서 총 8개의 messageConverter를 가집니다. 그리고 새로 추가된 *messageConverter*는 *ContentType*이 *text/html*인 응답을 처리합니다.

## 회고

오류를 처리하는 과정은 실제로 조금 더 힘들었습니다. 외부 API 명세서에는 ContentType에 대한 내용이 없었고, 당연히 application/json이라 짐작하고 오류를 찾으려고 했습니다. 그래서 오류 로그가 "text/html"이라고 친절히 알려줘도 지나치고, 엉뚱한 데서 헤매다가 돌아왔습니다. 역시 로그 확인을 꼼꼼하게 하는 습관은 중요한 거 같습니다. 적은 비용으로 많은 정보를 알 수 있는 습관이니까요.  
지금은 RestTemplate이 Deprecated되었지만 아직까지 사용하기에 괜찮은 클래스입니다. 나중에 RestTemplate를 WebClient로 교체하게 될 때 그 내용도 포스팅하게 되면 좋겠습니다.

***
## 환경
OpenJDK17, SpringBoot 2.6.5, SpringWeb 5.3.17, Jackson 2.13.2, AssertJ 3.21.0, junit-jupiter 5.8.2