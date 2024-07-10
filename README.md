# lynn-spec

lynn 구현에 대한 스펙 문서

---

CCv3L, Lorebook, 프롬프팅 서버 + 를 포함

Lynn 개발 소스에서 CCv3와 관련된 코드는 전부 위 레포의 스펙 url을 주석 바람.

## General Code Rule for Lynn

- When the function is *NOT* case-sensitive handle `String` as uppercase.
- All *NONE* network functions must be done under 25ms. (M1 Mac, Safari)

e.g.
```typescript
let LoremIpsum?: String[]
```
