---
title:  "[AWS] SpringBoot 에서 AWS Cognito OAuth2 와 JWT 동시에 사용하기"
excerpt: "AWS Cognito OAuth2 와 JWT 토큰을 이용한 인증처리"

tags:
  - [AWS, Cognito, OAuth2, Java, Springboot]

toc: true
toc_sticky: true
toc_label: "[AWS Cognito #4]"
 
date: 2022-05-31
last_modified_at: 2022-05-31
---

- AWS Cognito 활용하기
  1.  [AWS Cognito 사용자 풀 생성하기](https://ymkmoon.github.io/Aws-02-Cognito/)
  2.  [AWS Cognito OAuth2(Google) 설정하기](https://ymkmoon.github.io/Aws-03-Cognito-OAuth2/)
  3.  [React 에서 Cognito 사용하기](https://ymkmoon.github.io/Aws-04-Cognito-React/)
  4.  [SpringBoot(Restful API) 에서 Cognito 로 토큰 검증하여 인증받기](https://ymkmoon.github.io/Aws-05-Springboot-Cognito/)

<hr/>

JWT 을 이용한 자체 인증 + Cognito 의 OAuth2 Access token 


> 모든 소스코드는 [여기](https://github.com/ymkmoon/toyseven){:target="_blank"} 에서 확인 가능합니다.


- 참고사이트
  - [블로그](https://recordsoflife.tistory.com/144){:target="_blank"}
  - [stackoverflow](https://stackoverflow.com/questions/52606720/issue-with-having-multiple-websecurityconfigureradapter-in-spring-boot){:target="_blank"}
  - [Git Issue](https://github.com/spring-projects/spring-security/issues/5593){:target="_blank"}

<br>

## 시작하기 전에 

SpringBoot 기반의 RestAPI 에 AWS 의 Cognito 를 연동하는 부분을 진행합니다.

제 소스코드는 `관리자 유저데이터는 DB`로, `회원 데이터는 AWS의 Cognito` 를 이용하여 관리 합니다. 모두 토큰 인증 기반이며, Request Header 의 Authorization 에 대한 값으로 검증하기 때문에 `관리자일때와 일반유저일때 필터를 구분` 해 줄 필요가 있었습니다. 그러기 위해 Security 설정을 나누었고 그에대한 진행 상황을 기록하였으니 참고 부탁드립니다.


### ``의존성 추가``

OAuth2 에 대한 의존성을 추가해준다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'
implementation 'org.springframework.security.oauth.boot:spring-security-oauth2-autoconfigure:2.5.2'
```

<br>


### ``Exclude Url 설정``

`OncePerRequestFilter` 를 상속받는 Class 인 `JwtRequestFilter.java` 파일에서 제외 할 url 을 추가해준 후 shouldNotFilter 를 오버라이드 한다.



```java
@Component
@RequiredArgsConstructor
public class JwtRequestFilter extends OncePerRequestFilter {
  
  // 여기서 제외할 엔드포인트를 작성한다.
  private static final List<String> EXCLUDE_URL =
          Collections.unmodifiableList(
                  Arrays.asList(
                      "/cognito/payload/sub"
                  ));

  // EXCLUDE_URL 에 명시 된 엔드포인트에서는 doFilterInternal 가 작동하지 않도록 한다.
  @Override
  protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
      return EXCLUDE_URL.stream().anyMatch(exclude -> exclude.equalsIgnoreCase(request.getServletPath()));
  }

}

```

<br>


### ``Security 설정``

```java
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
	public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
		
		private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
		private final JwtAccessDeniedHandler jwtAccessDeniedHandler;
		private final JwtRequestFilter jwtRequestFilter;
		private final JwtServiceImpl jwtService;
	    
	    @Override
		protected void configure(HttpSecurity http) throws Exception {
			http
				.antMatcher("/voc/answer")
				.authorizeRequests()
					.anyRequest().hasAnyRole("ADMIN", "ADMIN2")
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
		}
	    
		
		@Override
		public void configure(AuthenticationManagerBuilder auth) throws Exception {
			auth.userDetailsService(jwtService).passwordEncoder(passwordEncoder());
		}
		
		@Bean
		@Override
		public AuthenticationManager authenticationManagerBean() throws Exception {
			return super.authenticationManagerBean();
		}

	}
	
	@Configuration
	@Order(2)
	@PropertySource(value = "classpath:application.yml")
	@RequiredArgsConstructor
	public class ApiSecurityConfig extends WebSecurityConfigurerAdapter {
		
		@Override
		protected void configure(HttpSecurity http) throws Exception {
			http
            	.authorizeRequests()
	            .antMatchers("/**/**")
	            	.permitAll()
                .anyRequest().authenticated()
                .and()
                .cors() // cross-origin
                .and()
				.oauth2ResourceServer().jwt()
					.jwkSetUri(jwkSetUri);
			
			http.csrf().disable(); 
			http.headers()
				.frameOptions().sameOrigin(); 
			http
				.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS); // 토큰 기반 인증이므로 세션 사용 x
			http.httpBasic().disable()
				.cors().configurationSource(request -> new CorsConfiguration().applyPermitDefaultValues());
		}
		
		@Bean
		@Override
		public AuthenticationManager authenticationManagerBean() throws Exception {
			return super.authenticationManagerBean();
		}
	}
}

```


<br>


### ``OAuth2 Resources server 정보 설정``

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://toyseven-domain.auth.ap-northeast-2.amazoncognito.com/ap-northeast-2_xxxxxxxxx
          jwk-set-uri: https://cognito-idp.ap-northeast-2.amazonaws.com/ap-northeast-2_xxxxxxxxx/.well-known/jwks.json
        opaquetoken:
          client-id: xxxxxxxxxxxxxxxxxx
```

<br>


### ``컨트롤러 생성``

```java
@RestController
@RequestMapping("cognito")
public class CognitoController {

  @GetMapping("/payload/sub")
  public ResponseEntity<String> message(Principal principal) {
	  return new ResponseEntity<>(principal.getName(), HttpStatus.OK);
  }
}
```

<br>


### ``테스트``

아래와 같이 직접 발급한 `JWT 인증`도, Cognito 의 `OAuth2 Access token` 을 이용한 인증처리도 작동 되는 걸 확인 할 수 있다.

Cognito 에서 발급받은 access token 이 필요하다면 [여기](https://github.com/ymkmoon/toyseven-react){:target="_blank"} 에서 레포를 클론 한 뒤 `amplifyConfig.js` 파일에서 Cognito 설정을 본인에 맞게 변경 후 로그인 하면 브라우저 콘솔에서 확인 할 수 있다.

![COGNITO](/assets/image/springboot/Springboot_cognito_01.PNG)

![COGNITO](/assets/image/springboot/Springboot_cognito_02.PNG)


<br>


### ``마치며``

여기서 중요한점은 Security 구성을 OAuth2 를 사용 할 때와 아닐 떄 총 두가지의 Security 구성을 해야하고, `@Order` 어노테이션을 이용해 우선순위도 함께 지정해줘야 한다. 

별도로 구성하지 않으면 OAuth2 Resources Server 만을 이용해 객체를 생성하려고 해서 필터가 정상적으로 처리 돼도 아래와 같이 토큰 복호화 부분에서 시그니처 오류가 발생한다.

Response Hedaer <br>
Key : WWW-Authenticate <br>
Value : Bearer error="invalid_token" <br>
error_description : "An error occurred while attempting to decode the Jwt: Signed JWT rejected: Another algorithm expected, or no matching key(s) found", 
error_uri="https://tools.ietf.org/html/rfc6750#section-3.1"


위 내용은 Response 에서 Body 가 아닌 Header 에 담겨오는 내용이라 에러를 한 눈에 파악하기 어렵기 때문에, OAuth2 부분에 예외 핸들링을 할 수 있게 아래 내용을 추가해주자.


```java
http
	.authorizeRequests()
	.antMatchers("/**/**")
		.permitAll()
	.anyRequest().authenticated()
	.and()
	.cors() // cross-origin
	.and()
	.oauth2ResourceServer()
		.authenticationEntryPoint(jwtAuthenticationEntryPoint) // 추가 된 내용
		.accessDeniedHandler(jwtAccessDeniedHandler) // 추가 된 내용
		.jwt()
		.jwkSetUri(jwkSetUri);
```