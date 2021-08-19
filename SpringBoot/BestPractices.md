# Restful Web Services - Best practices
## Consumer first
```
개발자 중심의 설계보다는 해당 API를 사용하는 소비자입장에서 간단명료하고 직관적인 API형태로 설계
```
## Make best use of HTTP
```
REST API를 설계함에있어서 HTTP METHOD와 Request, Response, header 값들과 같이 http 장점을 최대한 살려서 설계
```
## Request methods
```
적절한 http method를 사용해야한다.
- GET
- POST
- PUT
- DELETE
```
## Response Status
```
적절한 status code를 반환시켜줘야한다.
```
## No secure info in URI
```
우리가 제공하는 URI에 사용자 비밀번호와 같이 중요한 정보를 포함하면 안된다.
```
## Use plurals
```
복수형태 URI로 표시한다.
- /user -> /users
- /user/1 -> /users/1
```
## User nouns for resources
```
리소스 형태는 동사형태보다는 직관적인 명사형태를 사용하는 것이좋다.
```
## For exceptions
```
일관된 접근 엔드포인트를 사용하는 것이 좋다.
- PUT /gits/{id}/star
- DELETE /gits/{id}/star
```
