---
title:  "[Java] 다중 필터에서 AOP Logging 구현하기(Parameter, RequestBody)"
excerpt: "다중 시큐리티 환경에서 Aop 를 이용해 Request 에 저장 된 Parameter 와 RequestBody 출력하기"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[Logging]"
 
date: 2023-01-30
last_modified_at: 2023-01-30
---

- 참고사이트
  - [블로그](https://taetaetae.github.io/2019/06/30/controller-common-logging/){:target="_blank"}
  - [블로그](https://hirlawldo.tistory.com/42){:target="_blank"}

### 환경 설명

시작하기 전 본인 환경에 대해 설명을 하겠다. <br>
<br>
현재 작업중인 [사이드 프로젝트](https://github.com/ymkmoon/toyseven){:target="_blank"} 의 환경은 아래와 같다.

- SpringBoot 2.4.4
- java 1.8
- Gradle 7.2
- h2 In-memory DB
- JPA
- AWS Cognito


유저와 관리자가 존재하며, `유저는 AWS Cognito 를 이용하여 발급 받은 JWT`를 사용하고 `관리자는 서버에서 생성 한 JWT` 를 사용한다. <br>
<br>
각 토큰 발급 방식에 따라 `암호화 키`, `만료시간` 등 차이가 있기 때문에 Security 환경 구성을 인증 방식에 따라 별도로 구성해야 하며, 각 구성에 따라 다른 `Filter` 를 사용해야 한다. <br>
<br>
하지만, 하나의 시큐리티 구성만 사용하고 있더라도 본문을 참고해서 구현 가능 할거라고 생각한다. 

<hr/>


### Parameter 출력하기

일반적인 규약이 존재한다. (조회는 Get, 입력은 Post, 수정은 Patch 와 Put ,,, 등) <br>

사이드 프로젝트 에서 Parameter 는 Get Method 에서만 이용하기 때문에 아래와 같이 코드를 작성했다. 

> LogAspect.java

```java
@Around("getMapping()")
public Object aroundGet(ProceedingJoinPoint pjp) throws Throwable {
    HttpServletRequest request = ((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest();
    Map<String, String[]> paramMap = request.getParameterMap();
    if (logger.isInfoEnabled() && !paramMap.isEmpty()) {
        logger.info("[ {} ]", paramMapToString(paramMap));
    }
    return around(pjp);
}

public String paramMapToString(Map<String, String[]> paramMap) {
    return paramMap.entrySet().stream()
            .map(entry -> String.format("%s -> (%s)",entry.getKey(), Joiner.on(",").join(entry.getValue())))
            .collect(Collectors.joining(", "));
}
```


위 코드를 추가 후 Postman 을 이용하여 요청하게 될 경우 아래와 같이 내가 요청한 Parameter 가 출력된다.

![CentOS7](/assets/image/java/Logging_01.PNG)

<hr/>


### Request Body 출력하기

Request Body 출력하기 위한 핵심은 Request 를 읽을 수 있도록 Wrapper 를 구현해야 한다. <br>
InputStream 은 한번밖에 읽을 수 없어 자칫 잘못하다간 body의 내용이 유실될 수도 있기 때문 이 필터에서 스트림을 읽는 대신, 래퍼 구현으로 새 스트림 생성해야만 한다. <br>

<hr/>


#### 의존성추가

> build.gradle

구현에 필요한 의존성을 추가

```gradle
// https://mvnrepository.com/artifact/commons-io/commons-io
implementation 'commons-io:commons-io:2.6'
```

<hr/>


#### Wrapper 클래스 생성

> ReadableRequestWrapper.java

HttpServletRequest 를 상속받을 class 를 추가하자

```java
import java.io.BufferedReader;
import java.io.ByteArrayInputStream;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.util.Collections;
import java.util.Enumeration;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.stream.Collectors;

import javax.servlet.ReadListener;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;

import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.apache.http.entity.ContentType;
import org.json.simple.JSONArray;
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class ReadableRequestWrapper extends HttpServletRequestWrapper { // 상속
	
	private final Charset encoding;
	private byte[] rawData;
	private Map<String, String[]> params = new HashMap<>();

	public ReadableRequestWrapper(HttpServletRequest request) {
		super(request);
		this.params.putAll(request.getParameterMap()); // 원래의 파라미터를 저장

		String charEncoding = request.getCharacterEncoding(); // 인코딩 설정
		this.encoding = StringUtils.isBlank(charEncoding) ? StandardCharsets.UTF_8 : Charset.forName(charEncoding);

		try {
			InputStream is = request.getInputStream();
			this.rawData = IOUtils.toByteArray(is); // InputStream 을 별도로 저장한 다음 getReader() 에서 새 스트림으로 생성

			// body 파싱
			String collect = this.getReader().lines().collect(Collectors.joining(System.lineSeparator()));
			if (StringUtils.isEmpty(collect)) { // body 가 없을경우 로깅 제외
				return;
			}
			if (request.getContentType() != null && request.getContentType().contains(
				ContentType.MULTIPART_FORM_DATA.getMimeType())) { // 파일 업로드시 로깅제외
				return;
			}
			JSONParser jsonParser = new JSONParser();
			Object parse = jsonParser.parse(collect);
			if (parse instanceof JSONArray) {
				JSONArray jsonArray = (JSONArray)jsonParser.parse(collect);
				setParameter("requestBody", jsonArray.toJSONString());
			} else {
				JSONObject jsonObject = (JSONObject)jsonParser.parse(collect);
				@SuppressWarnings("unchecked")
				Iterator<String> iterator = jsonObject.keySet().iterator();
				while (iterator.hasNext()) {
					String key = iterator.next();
					setParameter(key, jsonObject.get(key).toString().replace("\"", "\\\""));
				}
			}
		} catch (Exception e) {
			log.error("ReadableRequestWrapper init error", e);
		}
	}

	@Override
	public String getParameter(String name) {
		String[] paramArray = getParameterValues(name);
		if (paramArray != null && paramArray.length > 0) {
			return paramArray[0];
		} else {
			return null;
		}
	}

	@Override
	public Map<String, String[]> getParameterMap() {
		return Collections.unmodifiableMap(params);
	}

	@Override
	public Enumeration<String> getParameterNames() {
		return Collections.enumeration(params.keySet());
	}

	@Override
	public String[] getParameterValues(String name) {
		String[] result = null;
		String[] dummyParamValue = params.get(name);

		if (dummyParamValue != null) {
			result = new String[dummyParamValue.length];
			System.arraycopy(dummyParamValue, 0, result, 0, dummyParamValue.length);
		}
		return result;
	}

	public void setParameter(String name, String value) {
		String[] param = {value};
		setParameter(name, param);
	}

	public void setParameter(String name, String[] values) {
		params.put(name, values);
	}

	@Override
	public ServletInputStream getInputStream() {
		final ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(this.rawData);

		return new ServletInputStream() {
			@Override
			public boolean isFinished() {
				return false;
			}

			@Override
			public boolean isReady() {
				return false;
			}

			@Override
			public void setReadListener(ReadListener readListener) {
				// Do nothing
			}

			public int read() {
				return byteArrayInputStream.read();
			}
		};
	}

	@Override
	public BufferedReader getReader() {
		return new BufferedReader(new InputStreamReader(this.getInputStream(), this.encoding));
	}
}
```


<hr/>


#### Filter 수정

상속받아 사용 할 구현체는 완성 됐다. <br>
Filter 에서의 핵심은 `HttpServletRequest 대신 Wrapper 클래스를 사용`하도록 변경을 해줘야한다. <br>
기존에는 `chain.doFilter(request, response)` 로 사용되고 있었고, wrapper 클래스를 사용하도록 변경해야 한다.


> JwtRequestFilter.java

```java
import java.io.IOException;
import java.io.PrintWriter;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.http.MediaType;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.OncePerRequestFilter;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.toyseven.ymk.common.ReadableRequestWrapper;
import com.toyseven.ymk.common.error.ErrorCode;
import com.toyseven.ymk.common.error.ErrorResponse;
import com.toyseven.ymk.common.util.JwtUtil;
import com.toyseven.ymk.jwt.JwtService;

import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.MalformedJwtException;
import io.jsonwebtoken.SignatureException;
import lombok.RequiredArgsConstructor;

public class JwtRequestFilter extends OncePerRequestFilter {
	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
		
		ReadableRequestWrapper wrapper = new ReadableRequestWrapper(request);
		chain.doFilter(wrapper, response);
	}
}
```

<hr/>


#### Security 환경 구성

해당 내용에 대해서는 이해하기 쉽도록 원본 코드를 추가 했다.

> SecurityConfig.java

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.jwt.NimbusJwtDecoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.web.cors.CorsConfiguration;

import com.toyseven.ymk.common.error.exception.JwtAccessDeniedHandler;
import com.toyseven.ymk.common.error.exception.JwtAuthenticationEntryPoint;
import com.toyseven.ymk.common.filter.DefaultRequestFilter;
import com.toyseven.ymk.common.filter.JwtRequestFilter;
import com.toyseven.ymk.common.filter.OAuth2RequestFilter;

import lombok.RequiredArgsConstructor;

@EnableWebSecurity
public class SecurityConfig {
	
	@Value("${spring.security.oauth2.resourceserver.jwt.jwk-set-uri}")
	private String jwkSetUri;
	
	@Bean
	public PasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}
	
	@Bean
	public JwtDecoder jwtDecoder(){
		return NimbusJwtDecoder.withJwkSetUri(jwkSetUri).build();
	}
	
	@Configuration
	@Order(1)
	@PropertySource(value = "classpath:application.yml")
	@RequiredArgsConstructor
	public class AdminSecurityConfig {
		
		private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
		private final JwtAccessDeniedHandler jwtAccessDeniedHandler;
		private final JwtRequestFilter jwtRequestFilter;
	    
		@Bean
		protected SecurityFilterChain adminFilterChain(HttpSecurity http) throws Exception {
			
			http
				.antMatcher("/voc/answer")
				.authorizeRequests()
				.antMatchers(HttpMethod.POST, "/voc/answer").hasAnyRole("ADMIN", "ADMIN2")
				.and().cors();
			http.csrf().disable(); 
	    	http.headers()
	    		.frameOptions().sameOrigin(); 
	    	http.exceptionHandling()
	    		.authenticationEntryPoint(jwtAuthenticationEntryPoint)
	    		.accessDeniedHandler(jwtAccessDeniedHandler)
	    		.and()
				.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS); // 토큰 기반 인증이므로 세션 사용 x
	    	http.httpBasic().disable()
				.cors().configurationSource(request -> new CorsConfiguration().applyPermitDefaultValues());
	    	http.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);
	    	
	    	return http.build();
		}
	    
	    @Bean
		public AuthenticationManager authenticationManager(
				AuthenticationConfiguration authenticationConfiguration) throws Exception {
			return authenticationConfiguration.getAuthenticationManager();
		}
		

	}
	
	@Configuration
	@Order(2)
	@PropertySource(value = "classpath:application.yml")
	@RequiredArgsConstructor
	public class UserSecurityConfig {
		
		private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
		private final JwtAccessDeniedHandler jwtAccessDeniedHandler;
		private final OAuth2RequestFilter oauth2RequestFilter;
		
		@Bean
		protected SecurityFilterChain userFilterChain(HttpSecurity http) throws Exception {
			http
            	.authorizeRequests()
	            .antMatchers(HttpMethod.POST, "/voc/question")
                .authenticated()
                .and()
                .cors() // cross-origin
                .and()
				.oauth2ResourceServer()
					.authenticationEntryPoint(jwtAuthenticationEntryPoint)
					.accessDeniedHandler(jwtAccessDeniedHandler)
					.jwt()
					.jwkSetUri(jwkSetUri);
			
			http.csrf().disable(); 
			http.headers()
				.frameOptions().sameOrigin(); 
			http
				.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS); // 토큰 기반 인증이므로 세션 사용 x
			http.httpBasic().disable()
				.cors().configurationSource(request -> new CorsConfiguration().applyPermitDefaultValues());
			http.addFilterBefore(oauth2RequestFilter, UsernamePasswordAuthenticationFilter.class);
			
			return http.build();
		}
		
		@Bean
		public AuthenticationManager authenticationManager(
				AuthenticationConfiguration authenticationConfiguration) throws Exception {
			return authenticationConfiguration.getAuthenticationManager();
		}
	}
	
	@Configuration
	@Order(3)
	@PropertySource(value = "classpath:application.yml")
	@RequiredArgsConstructor
	public class DefaultSecurityConfig {
		
		private final DefaultRequestFilter defaultRequestFilter;
		
		@Bean
		protected SecurityFilterChain defaultFilterChain(HttpSecurity http) throws Exception {
			http
            	.authorizeRequests()
	            .antMatchers("/**/**")
	            	.permitAll()
                .anyRequest().authenticated()
                .and()
                .cors() // cross-origin
                .and();
			
			http.csrf().disable(); 
			http.headers()
				.frameOptions().sameOrigin(); 
			http
				.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS); // 토큰 기반 인증이므로 세션 사용 x
			http.httpBasic().disable()
				.cors().configurationSource(request -> new CorsConfiguration().applyPermitDefaultValues());
//			http.addFilter(defaultRequestFilter);
			http.addFilterBefore(defaultRequestFilter, UsernamePasswordAuthenticationFilter.class);
			
			return http.build();
		}
		
		@Bean
		public AuthenticationManager authenticationManager(
				AuthenticationConfiguration authenticationConfiguration) throws Exception {
			return authenticationConfiguration.getAuthenticationManager();
		}
	}
}

```


하나의 클래스 안에 각 인증에 맞는 Security 를 구성하였고, 총 3개(관리자전용, 유저전용, 모두)의 Config Class가 존재한다. <br>
<br>
기존 DefaultSecurityConfig 에서는 따로 Filter 를 이용한 작업이 필요하지 않아, 추가해주지 않았지만 Reuqest Body 출력을 위해서는 모든 Request 에 대해 Wrapper 생성은 필수적이기 때문에 이번에 추가를 한 부분이다. <br>

DefaultRequestFilter 의 코드는 아래와 같다.

> DefaultRequestFilter.java

```java
import java.io.IOException;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import com.toyseven.ymk.common.ReadableRequestWrapper;

@Component
public class DefaultRequestFilter extends OncePerRequestFilter {

	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws ServletException, IOException {
		ReadableRequestWrapper wrapper = new ReadableRequestWrapper((HttpServletRequest)request);
		chain.doFilter(wrapper, response);
	}
}
```

<hr/>


#### Aop 구현

이제 RequestBody 를 출력 할 준비는 모두 끝났다. Aop 를 이용하여 RequestBody 를 출력하자.<br>
여기서 실제 로그를 출력하는 부분은 `logger.info("[ {} ]", IOUtils.toString(request.getReader()));` 이다. <br>
getReader() 에서 Wrapper 를 사용하지 않고 HttpServletRequest 를 사용하면 이미 request 를 한 번 읽었기 때문에 예외가 발생하게 된다. 즉 body 가 유실 되어 로깅을 남길 수 없고, 이걸 해결하기 위해서 본문에 대한 내용을 선행해주어야 한다.



> LogAspect.java

```java
import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.apache.commons.io.IOUtils;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import com.toyseven.ymk.common.util.DataParsingUtil;

import lombok.RequiredArgsConstructor;

@Aspect
@Component
@RequiredArgsConstructor
public class LogAspect {
	
    Logger logger =  LoggerFactory.getLogger(LogAspect.class);
    
    // 특정 패키지에서 로그 출력
//    @Around("execution(* ~~~.~~~.~~~.~~~..*Service.*(..))")
    @Around("execution(* com.toyseven.ymk..*Service.*(..))")
    public Object logging(ProceedingJoinPoint pjp) throws Throwable {
    	return around(pjp);
    }
    // 특정 메소드에서 로그 출력
    @Around("execution(* com.toyseven.ymk.voc.VocController.saveVocAnswer(..))")
    public Object loggingWhenSaveVocAnswer(ProceedingJoinPoint pjp) throws Throwable {
    	// HttpSession session = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest().getSession();
    	Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    	logger.info("Answered by {}", auth.getName());
    	return pjp.proceed();
    }
    
    @Pointcut("@annotation(org.springframework.web.bind.annotation.GetMapping)")
    public void getMapping() {
    	logger.info("Pointcut GetMapping(): begin");
    	logger.info("Pointcut GetMapping(): end");
    }
    @Pointcut("@annotation(org.springframework.web.bind.annotation.PostMapping)")
	public void postMapping() {
		logger.info("Pointcut PostMapping(): begin");
		logger.info("Pointcut PostMapping(): end");
	}
	@Pointcut("@annotation(org.springframework.web.bind.annotation.RequestMapping)")
	public void requestMapping() {
		logger.info("Pointcut RequestMapping(): begin");
		logger.info("Pointcut RequestMapping(): end");
	}
    
	
	@Before("getMapping()") 
	public void beforeGetMethod(JoinPoint pjp) { 
		before(pjp);
	}
	@Before("postMapping()") 
	public void beforePostMethod(JoinPoint pjp) { 
		before(pjp);
	}
	@Before("requestMapping()") 
	public void beforeRequestMethod(JoinPoint pjp) { 
		before(pjp);
	}
	

	@Around("getMapping()")
	public Object aroundGet(ProceedingJoinPoint pjp) throws Throwable {
		HttpServletRequest request = ((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest();
		Map<String, String[]> paramMap = request.getParameterMap();
		if (logger.isInfoEnabled() && !paramMap.isEmpty()) {
			logger.info("[ {} ]", DataParsingUtil.paramMapToString(paramMap));
		}
		return around(pjp);
	}
	
	@Around("postMapping()")
	public Object aroundPost(ProceedingJoinPoint pjp) throws Throwable {
		HttpServletRequest request = ((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest();
		if (logger.isInfoEnabled()) {
			logger.info("[ {} ]", IOUtils.toString(request.getReader()));
		}
		return around(pjp);
	}
	@Around("requestMapping()")
	public Object aroundRequest(ProceedingJoinPoint pjp) throws Throwable {
		return around(pjp);
	}
	
	private final Object around(ProceedingJoinPoint pjp) throws Throwable {
		logger.info("start @Arround - {} / {}", pjp.getSignature().getDeclaringTypeName(), pjp.getSignature().getName());
        Object result = pjp.proceed();
        logger.info("finished @Arround - {} / {}", pjp.getSignature().getDeclaringTypeName(), pjp.getSignature().getName());
        return result;
	}
	
	private final void before(JoinPoint pjp) {
		logger.info("start @Before - {} / {}", pjp.getSignature().getDeclaringTypeName(), pjp.getSignature().getName());
        logger.info("finished @Before - {} / {}", pjp.getSignature().getDeclaringTypeName(), pjp.getSignature().getName());
	}
    
}

```

<hr/>


### 마치며

로깅에 대해 작업하는 내용을 기록해봤고, 나 같은 경우는 시큐리티 환경 구성 때문에 삽질을 좀 했다. Default 환경에서의 Filter 추가 하는 부분을 삽질 끝에 발견하게 되었고 의미있는 시간이었다.



모든소스는 아래 github 을 클릭하여 확인 가능 합니다. <br>

[github](https://github.com/ymkmoon/toyseven){:target="_blank"}