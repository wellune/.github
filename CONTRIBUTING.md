# Bitsensing RABUWLTS 개발 컨벤션

## 브랜칭 가이드

- WIP인 (master에 머지되지 말아야 할 상태의) 결과물을 공유하는 경우 커밋 메시지의 타입에 `wip:` 로 표시해주시고, 브랜치에 사용할 경우 `WIP/` 로 시작해주세요 (예: `WIP/RABU-549/health-mywellune`)

## 공통 코딩 가이드

### 로케일 코드 사용

- 기본 컨벤션은 `{로케일}_{서브태그}`를 사용함. (ex: `en_US`)

### 사용되지 않는 코드의 주석 처리 금지

- 설명을 위한 용도가 아니라면 주석 내에 프로그램 코드를 작성 금지

### REST API 작성 지침

- API의 성공/실패 여부는 HTTP 상태 코드를 사용해 교환
- API가 요구사항을 모두 충족해 정상적인 응답을 할 수 있을 때에만 2XX 상태 코드를 리턴

### Async API 작성 지침

- MOM을 사용하는 Consumer 서비스는 리포 내부에 [Async API Spec](https://studio.asyncapi.com/)을 작성
- 토픽명은 `<유형>.<도메인>.<서비스>.<토픽>.<버전>` 형태로 작성 (예: `fct.wellune.sleepcard.created.1`)
- 메시지 페이로드 인코딩은 [Protobuf](https://protobuf.dev/)를 사용
- 메시지의 페이로드가 변경되는 경우 다른 버전 넘버를 사용

### 예외처리

- 발생할 수 있는 예외에 대해 사전에 인지하고 예외 코드를 작성
- 예외 처리 시나리오는 사용자 경험 기획에 기반해 작성
- 마지막으로, 예상하지 못한 예외는 Sentry를 사용해 수집

## Node.js 코딩 가이드

- 언어는 TypeScript 기본
- [Node Version Manager](https://github.com/nvm-sh/nvm) 를 이용해서 `.nvmrc` 파일에 프로젝트에서 요구하는 Node.js 버전을 명시

## TypeScript 코딩 가이드

- 기본 린터는 [prettier](https://prettier.io/) 사용

## Java / Spring Boot 코딩 가이드

### 컨트롤러별 기본 예외 처리 전략

```java
@RestController
public class UserController {

    @PostMapping("/rest/user/updateTimezone")
    public CommonMap updateUserTimezone(@RequestBody Map<String, Object> request) {
        // 요청 본문에서 "user_id"와 "timezone" 파라미터가 존재하는지 확인
        // 만약 누락된 경우 IllegalArgumentException 예외를 발생시켜, 기본 예외 핸들러가 처리
        if (!request.containsKey("user_id") || !request.containsKey("timezone")) {
            throw new IllegalArgumentException("Missing required parameters: user_id and/or timezone");
        }
        
        CommonMap params = new CommonMap();
        params.put("id", request.get("user_id"));
        params.put("tz", request.get("timezone"));
        return userService.updateUserTimezone(params);
    }

    // 예외 처리 메서드: IllegalArgumentException 예외 발생 시 호출
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<CommonMap> handleIllegalArgumentException(IllegalArgumentException e) {
        // 에러 응답 정보
        CommonMap errorResponse = new CommonMap();
        errorResponse.put("instance", "/rest/user"); // 에러 발생한 URI 정보
        errorResponse.put("status", 400);              // HTTP 상태 코드 400 (Bad Request)
        errorResponse.put("title", "Illegal Argument"); // 에러 제목
        errorResponse.put("detail", e.getMessage());     // 발생한 예외의 상세 메시지
        errorResponse.put("type", "https://example.com/errors/illegal-argument"); // 에러 유형에 대한 URL 또는 설명

        // 400 Bad Request 상태와 함께 에러 응답을 반환합니다.
        return ResponseEntity.badRequest().body(errorResponse);
    }
}
```

1. API로 제공하고자 하는 리소스/컨트롤러별 요구사항에 맞추어 예외 상황과 그에 따른 코드가 준비돼야 함
2. 이 프로토콜은 가능하면 실제 API 컨트롤러 작성 전에 협의가 필요하며, 협의에 따라 스펙화한 예외만 처리하는 것이 기본 전략임
3. 나머지 협의되지 않은 상황은 처리되지 않은 에러 (`500 Internal Server Error`)로 리턴
4. 미리 작업되지 않은 에러에 대한 처리는 정기적인 코드 퀄리티 작업을 통해 처리

