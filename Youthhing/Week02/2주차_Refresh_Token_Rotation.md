# Refresh Token 탈취 문제 해결을 위한 Rerfesh Token Rotation

일반적으로 인증과 인가를 구현할 때, AccessToken과 RefreshToken을 사용한다. 우리 프로젝트에서도 둘을 활용한 인증을 구현했는데 이번 글에선, COTATO 프로젝트의 인증, 인가를 구현하며 고민한 RefreshToken 탈취 문제와 관련된 내용을 정리하겠다.

### 배경 지식

해당 글을 위해선 인증과 인가, 토큰 인증 방식에 대한 이해가 필요한데 공부가 급하다면 우선은 아래와 같은 내용으로 이해하자.

-   인증 : 너가 너가 맞는지 확인하는 것
-   인가: 너가 자격이 있는지 확인하는 것
-   토큰: 서버의 부담을 덜기 위해, 사용자에게 입장권을 주고 입장권이 적절한지 확인하는 방식
-   세션: 인증된 사용자의 목록을 서버에서 관리해 인증하는 방식

### AccessToken

AccessToken이란, 일반적인 웹 어플리케이션에서 사용자를 인증하는 목적으로 사용된다.

서버는 사용자 인증 시 (로그인과 같은 경우) AccessToken을 발급해 사용자에게 전달하고 클라이언트는 서버에 접근하기 위해 요청을 보낼때마다 본인이 인증된 사용자임을 알리기 위해 AccessToken을 서버에 전달한다.

서버는 해당 토큰 디코딩을 통해 해당 토큰이 우리 서버에서 발급한 것이 맞는지, 만료된 토큰은 아닌지 간단한 확인 후 맞다면 이후 요청을 처리한다.

이렇게 서버는 AccessToken의 만료시간을 설정해 일정 시간이 지나면 사용자에게 다시 인증을 하라는 ‘액세스 토큰 재발급’요청을 요구한다.

이 때, AccessToken의 만료시간에 대해 생각해보자.

너무 짧으면 → 액세스 토큰을 자주 재발급해야한다. (5분에 한번씩 로그인해야하는 상황..)

너무 길면 → 액세스 토큰은 사용자가 관리하기 때문에 언젠간 탈취된다. 탈취된 AccessToken을 통해 인증되지 않은 제 3자가 서버에 접근할 수 있다.

따라서, 보통은 AccessToken을 적당히 짧게 (30분 ~ 1시간) 설정하고 AccessToken을 재발급 받기 위한 토큰으로 RefreshToken을 활용한다.

### RefreshToken

AccessToken이 만료될 경우, 사용자가 매번 다시 로그인해서 재발급을 받아야한다면 상당히 불편할 것이다. 이걸 방지하기 위해서 ‘액세스 토큰을 재발급’을 목적으로 RefreshToken이 존재한다.

사용자는 AccessToken이 만료되면 RefreshToken을 서버에 전달해 서버가 RefreshToken을 검증하고 RefreshToken이 우리 서버에서 발급한 토큰이 맞는지등 검증을 한다. 이후 검증이 성공하면 사용자에게 새로운 AccessToken과 RefreshToken을 발급해준다.

RefreshToken의 만료시간을 적당히 길게 (3일 ~ 1주일) 설정하면, 아래와 같은 플로우로 적당히 사용자 인증을 유지할 수 있다.

1.  AccessToken 만료
2.  클라이언트의 RefreshToken을 포함한 재발급 요청
3.  **JWT 시그니처**를 통한 서버의 RefreshToken 검증
4.  AccessToken, RefreshToken 재발급

가령, AccessToken이 탈취되어도 비교적 짧은 시간에 토큰이 만료되기 때문에 공격자가 공격을 할 수 있는 시간은 아주 짧은 시간일 뿐이다.

### RefreshToken 탈취 문제

RefreshToken을 통해 AccessToken과 새로운 RefreshToken을 발급 받을 수 있다. RefreshToken은 생각해보면 만능인 것인데 그렇다면, RefreshToken이 탈취되었다면 어떻게 될까?

탈취된 RefreshToken을 통해 공격자가 AccessToken을 발급 받아 서버에 접근하고, RefreshToken을 새로 발급 받아 무제한으로 서버의 인증을 성공할 수 있다.

[##_Image|kage@bSocmL/btsIIiphrhW/30hFIkTgTCUY8JwyphQHl0/img.png|CDM|1.3|{"originWidth":1376,"originHeight":889,"style":"alignCenter","caption":"이런 플로우가 가능해진 것"}_##]

### RefreshToken Rotation (RTR)

서버에서 토큰을 검증할 때, 일반적으로 JWT를 사용한다고 가정하면, JWT의 Signature를 통해 해당 토큰이 우리 서버에서 발급한 내용이 맞는지 확인한다.

RefreshToken 탈취 문제가 발생했던 이유는 RefreshToken 검증을 할 때, “우리 서버에서 발급한 것이 맞는지” 만 확인했기 때문에 공격자가 탈취한 것이든, 사용자가 정상 발급한 것이든 상관없이 토큰 검증이 성공했다.

따라서, 공격자의 무분별한 재발급 요청을 막기 위해 RefreshToken 을 재발급 하는 과정에 “발급한 리프레시 토큰 외의 기존 리프레시 토큰을 무효화”하는 과정을 추가해 문제를 해결할 수 있다.

공격자가 RefreshToken을 탈취해 재발급 요청을 하는 과정에서 원래 사용자가 정상적으로 재발급 요청을 보낸 시나리오를 생각해보자.

1.  공격자가 RefreshToken을 탈취해 AccessToken을 발급받아 서버에 자유롭게 접근한다. (토큰명A)
2.  사용자가 RefreshToken을 재발급 요청한다. (토큰명 B)
3.  서버는 사용자의 Refresh Token이 B임을 저장해놓는다.
4.  공격자가 RefreshToken을 통해 재발급 요청을 한다.
5.  서버는 사용자의 RefreshToken이 B가 아님을 확인하고 잘못된 요청이라 인지하고 토큰을 재발급하지 않는다.

즉, 이렇게 서버에 사용자의 RefreshToken값을 저장하고, 해당 값과 재발급 요청한 RefreshToken이 맞는지를 확인한다면 사용자가 서비스에 자주 접근하면 공격자의 공격을 자연스럽게 막을 수 있게 된다.

### 구현

따라서, 우리는 서버가 사용자의 RefreshToken을 인지할 수 있게 하기 위해 인메모리 서버인 Redis를 활용했다.

사용자의 id를 key로 마지막으로 저장된 RefreshToken을 value로 저장한다.

```
@NoArgsConstructor
@AllArgsConstructor
@Getter
@RedisHash(value = "jwtToken", timeToLive = 60 * 60 * 24 * 3) //만료시간 3일
public class RefreshToken {

    @Id
    private Long id; // 사용자 id

    private String refreshToken; // 사용자의 refreshToken

    public void updateRefreshToken(String refreshToken) {
        this.refreshToken = refreshToken;
    }
}
```

또한 Reissue하는 과정에서 아래와 같이 저장된 RefreshToken이 맞는지 확인 후 아니라면 에러를 발생 시킨다.

```
RefreshToken findToken = refreshTokenRepository.findById(memberId)
                .orElseThrow(() -> new AppException(ErrorCode.EMAIL_NOT_FOUND));
```

### 쿠키Cookie 활용

하지만, 이 방법 또한 사용자가 서비스에 접근을 하지 않는다면, 공격자만 계속 Refresh Token을 갱신하기 때문에 문제가 될 수 있다는 생각이 들었다.

이를 방지하기 위해 RefreshToken을 쿠키를 통해 발급하고 HttpOnly 설정을 통해 공격자가 의도적인 토큰 조작을 하지 못하게 설정했다.

쿠키를 사용하면 아래와 같은 면에서 보안적 이점을 볼 수 있다.

**HttpOnly 속성 활용**

HttpOnly 옵션을 사용하면 JavaScript에선 쿠키를 읽지도, 조작하지도 못한다. 자연스레 XSS 공격으로부터 토큰을 보호할 수 있다. 따라서, 공격자가 어떠한 방식으로라도 RefreshToken을 탈취해도 별도로 할 수 있는 작업이 없다.

**Secure 속성 활용**

Secure 속성을 통해 오직 HTTPS의 경우에만 쿠키를 전송한다. 이를 통해 네트워크 도청을 통한 공격을 방지할 수 있다.

이 외에도 쿠키를 활용하면, 브라우저를 닫았다 켜도 로그인이 유지될 수 있다는 점, 브라우저의 자동 요청을 통해 클라이언트가 별도로 쿠키를 조작할 필요가 없다는 점의 이점도 있어 쿠키를 선택했다.

최종 코드

```
@Transactional
public ReissueResponse reissue(String refreshToken, HttpServletResponse response) {
  // 만료 여부 및 블랙리스트에 등록된 토큰인지 확인
    if (jwtTokenProvider.isExpired(refreshToken) || blackListRepository.existsById(refreshToken)) {
        log.warn("블랙리스트에 존재하는 토큰: {}", blackListRepository.existsById(refreshToken));
        throw new AppException(ErrorCode.REISSUE_FAIL);
    }
    // Redis에 저장된 토큰이 맞는지 확인, 일치하지 않으면 예외처리
    Long memberId = jwtTokenProvider.getMemberId(refreshToken);
    String role = jwtTokenProvider.getRole(refreshToken);

    RefreshToken findToken = refreshTokenRepository.findById(memberId)
            .orElseThrow(() -> new AppException(ErrorCode.EMAIL_NOT_FOUND));
    log.info("[브라우저에서 들어온 쿠키] == [DB에 저장된 토큰], {}", refreshToken.equals(findToken.getRefreshToken()));

    if (!refreshToken.equals(findToken.getRefreshToken())) {
        log.warn("[쿠키로 들어온 토큰과 DB의 토큰이 일치하지 않음.]");
        throw new AppException(ErrorCode.REFRESH_TOKEN_NOT_EXIST);
    }
    
    // 기존 토큰 블랙리스트 처리
    jwtTokenProvider.setBlackList(refreshToken);
    
    // 새로운 토큰 발급
    Token token = jwtTokenProvider.createToken(memberId, role);
    findToken.updateRefreshToken(token.getRefreshToken());
    refreshTokenRepository.save(findToken);
		
		// 쿠키를 통한 전달
    Cookie refreshCookie = new Cookie(REFRESH_TOKEN, token.getRefreshToken());
    refreshCookie.setMaxAge(refreshTokenAge / 1000);
    log.info("[리프레시 쿠키 발급, 발급시간 : {}]", refreshTokenAge / 1000);
    refreshCookie.setPath("/");
    refreshCookie.setHttpOnly(true);
    refreshCookie.setSecure(true);
    response.addCookie(refreshCookie);
    return ReissueResponse.from(token.getAccessToken());
}
```

이렇게, RefreshToken 탈취 문제를 해결하기 위해 RTR, 쿠키를 통해 보안 강화를 선택했다. 단순, 인증과 인가 개념을 넘어 세부적인 예외까지 고려하는 과정이 어려워 구현에 시간이 조금 걸렸다. 보안은 강할수록 좋다고 비용적 문제가 허용하는한 서버의 보안에 더욱 신경써야겠다.