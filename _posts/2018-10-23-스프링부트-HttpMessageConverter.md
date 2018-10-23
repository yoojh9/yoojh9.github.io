---
layout: post
title: "스프링부트 HttpMessageConverters"
tags: [SpringBoot]
comments: true
---

# 스프링 프레임워크와 HttpMessageConverters
HttpMessageConverter를 커스텀하여 개발하다 이슈가 발생했다. spring framework가 default로 제공해주는 HttpMessageConverter의 instance들이 있는데, 이를 제대로 알아보지 않고 그냥 구글링하여 코드를 추가하다보니 이슈가 생겼다

<br><br>

## 1. 이슈 발생

- 나는 \<, \> 등의 특수문자를 처리하는 작업을 Java Object를 JSON으로 변환시켜 주는 MappingJackson2HttpMessageConverter에 추가했고, HttpMessageConverter List에 추가시켰다. 내가 작성한 코드는 아래와 같다.

```
// 오류가 발생한 코드
@Configuration
@EnableWebMvc
public class WebMvcConfig implements WebMvcConfigurer{

  @Override
  public void configureMessageConverters(List<HttpMessageConverter<?>> converters){
    converters.add(htmlEscapingConverter());
  }

  @bean
  public HttpMessageConverter<?> htmlEscapingConverter(){
  	ObjectMapper objectMapper = new ObjectMapper();
	  objectMapper.getFactory().setCharacterEscapes(new HTMLCharacterEscapes());
	  objectMapper.registerModule(new JavaTimeModule());
		objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
		objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

		MappingJackson2HttpMessageConverter htmlEscapingConverter =
				new MappingJackson2HttpMessageConverter();
		htmlEscapingConverter.setObjectMapper(objectMapper);

		return htmlEscapingConverter;
  }
}

```

- 문제는 @Override 하여 configureMessageConverters()를 다시 작성함으로서, spring framework에서 제공해주는 미리 등록된 default message converter의 instance들을 잃어버렸다.
- 내가 만든 API들은 주로 json 방식이 많아 문제가 바로 발견되지 않았지만, 기존에 잘 동작하던 file resource를 다운로드 받는 API가 406 Not Acceptable 에러를 뱉어내기 시작했다.


<br><br>

## 2. HttpMessageConverters
- Srping MVC는 Http request와 response를 변환하기 위해 HttpMessageConverter 인터페이스를 사용한다.
- 일반적으로 spring framework에서는 Spring MVC를 이용해서 컨트롤러에 값을 주고 받을 때는 HTTP 요청 프로퍼티를 분석하여 그에 따라 특정 클래스로 바인딩 되게끔 하고 특정 객체를 Model Object에 집어 넣어 View를 리턴하는 식으로 주고받게 된다.
- 그러나 메세지 컨버터는 그런 개념이 아니라 HTTP 요청 메세지 본문과 HTTP 응답 메세지 본문을 통째로 하나의 메세지로 보고 이를 처리한다.  Spring MVC에서 이러한 작업을 하는데 사용되는 어노테이션이 바로 @RequestBody와 @ResponseBody이다.
- @ResponseBody를 이용하여 파라미터를 받으면 HTTP 요청 메세지 본문을 분석하는 것이 아닌 그냥 하나의 통으로 받아서 이를 적절한 클래스 객체로 변환하게 되고 @ResponseBody를 사용하여 특정 값을 리턴하면 View 개념이 아닌 HTTP 응답 메세지에 그 객체를 바로 전달할 수 있다.

<br><br>

## 3. Default Message converters
HttpMessageConverter에 미리 등록되어 있는 converter들을 몇가지 정리해보았다.

#### 3-1. ByteArrayHttpMessageConverter
- 지원하는 오브젝트 타입은 byte[]이다. 미디어 타입은 모든 것을 다 지원한다. 이 때문에 @RequestBody로 전달받을 때는 모든 종류의 HTTP 요청 메세지 본문을 byte 배열로 가지고 올 수 있다. 또한 @ResponseBody로 응답을 보낼 때는 Content-type이 application/octet-stream 으로 설정된다.


#### 3-2. StringHttpMessageConverter
- 지원하는 오브젝트 타입이 String 타입이다. 미디어 타입은 모든 것을 다 지원하며, HTTP 요청의 본문을 그대로 스트링 타입으로 가져올 수 있다. 텍스트 기반 포맷으로 가공하지 않은 본문을 가지고 오고 싶은 경우 사용하는 것이 좋다. @ResponseBody로 응답을 보낼 경우 Content-Type이 text/html로 전달된다.

#### 3-3. FormHttpMessageConverter
- 미디어 타입이 application/x-www-form-urlencoded로 정의된 폼 데이터를 주고 받을 때 사용한다. 오브젝트 타입은 MultiValueMap<String, String>을 지원한다

#### 3-4. MappingJacksonHttpMessageConverter, MappingJackson2HttpMessageConverter2
- Jackson ObjectMapper를 이용해서 자바 오브젝트와 JSON 문서를 자동변환해주는 메시지 컨버터다. 지원하는 미디어 타입은 application/json타입이다.

#### 3-5. ResourceHttpMessageConverter
- 모든 유형의 octet stream에 대해  org.springframework.core.io.Resource로 변환해준다.

<br><br>

## 4. 결론
- 내가 작성했던 코드는 MappingJackson2HttpMessageConverter만을 커스텀 하여 HttpMessageConverter에 추가 함으로서, 기존에 등록되어 있던 message converter의 instance를 잃어버리게 되었다. 그러다보니 대부분의 json 데이터로 주고 받는 API는 정상적으로 동작하게 되고, File Resource를 response로 전달하는 API는 해당 MessageConverter가 등록되어 있지 않아 에러가 발생하게 되었다.
- 기존에 작성했던 코드에서 필요한 부분은 defaultHttpMessageConvert를 추가하는 작업이 필요하다.
- 다음과 같이 코드를 변경했다. 기존에 WebMvcConfigurer를 implements한 클래스 내에 있던 코드를 WebMvcConfigurationSupport를 extends한 클래스 내로 구현했다.

```
@Configuration
public class HttpMessageConverterConfig extends WebMvcConfigurationSupport {

	@Bean
	public HttpMessageConverter<?> htmlEscapingConverter() {
		ObjectMapper objectMapper = new ObjectMapper();
		objectMapper.getFactory().setCharacterEscapes(new HTMLCharacterEscapes());
		objectMapper.registerModule(new JavaTimeModule());
		objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
		objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

		MappingJackson2HttpMessageConverter htmlEscapingConverter =
				new MappingJackson2HttpMessageConverter();
		htmlEscapingConverter.setObjectMapper(objectMapper);

		return htmlEscapingConverter;
	}

	@Override
	protected void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
		converters.add(htmlEscapingConverter());
		super.addDefaultHttpMessageConverters(converters);  // default Http Message Converter  추가
	}
}

```

---
#### 참고
[Baeldung - Http Message Converters with the Spring Framework](https://www.baeldung.com/spring-httpmessageconverter-rest) <br>
[Spring Framework의 메세지 컨버터를 이용한 FileDownload 구현](https://zgundam.tistory.com/12)
[Spring 3 - @MVC 메시지 컨버터](http://springsource.tistory.com/89)
