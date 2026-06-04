# Windows 검증 결과 — 2026-11-18

수업: https://www.slog.gg/p/14175

## 환경과 의존성

- Windows x64, Node.js 26.13.1 LTS, pnpm 12.21.0.
- Next.js / eslint-config-next 16.4.6, React / React DOM 19.3.1.
- Tailwind CSS / @tailwindcss/postcss 4.3.5.
- @types/node 26.6.6, @types/react / @types/react-dom 19.3.0.
- babel-plugin-react-compiler 1.0.0.
- TypeScript 7.0.2 native CLI + @typescript/typescript6 6.0.2 compiler API alias.
- ESLint 9.39.5. Registry latest는 10.15.0이지만 eslint-plugin-import 2.32.0, eslint-plugin-jsx-a11y 6.10.2, eslint-plugin-react 7.37.5의 peer 범위를 벗어납니다.

2026-11-18 npm registry의 dist-tags.latest를 다시 확인했습니다. 모든 직접 의존성과 pnpm 버전을 고정하고 모든 강의 태그에 동일한 pnpm-lock.yaml을 포함합니다. strictPeerDependencies를 활성화했습니다. sharp와 unrs-resolver의 설치 스크립트는 명시적으로 실행하지 않으며 Windows 사전 빌드 바이너리를 사용합니다.

## 검증 범위

| 대상 | 검사 | 결과 |
| --- | --- | --- |
| 69개 전체 강의 태그 | TypeScript 7 `tsc --noEmit --incremental false` | 69/69 통과 |
| 69개 전체 강의 태그 | ESLint `eslint . --max-warnings 0` | 69/69 통과 |
| 001, 028-1, 051 | `pnpm install --frozen-lockfile --strict-peer-dependencies` | 3/3 통과 |
| 001, 028-1, 051 | Next.js 16.4.6 Turbopack production build | 3/3 통과 |
| 백엔드 068 / Java 25 | Gradle test, bootJar | 테스트 14개 통과, JAR 생성 성공 |
| 프론트 apiFetch + 실제 백엔드 068 | 글·댓글 목록/상세/작성/수정/삭제와 CORS | 11개 확인 통과 |
| production frontend + 실제 백엔드 068 | 브라우저에서 글·댓글 조회/작성/수정/삭제 | 통과, 생성한 테스트 데이터 삭제 확인 |
| 강의 이력 | 원본과 같은 69개 메시지/순서/태그, 모든 태그가 main의 조상 | 통과 |

각 태그의 타입 검사에서는 나중 강의의 생성된 route 타입이 섞이지 않도록 기본 next-env.d.ts를 사용했습니다. 대표 단계의 프로덕션 빌드는 각 단계의 route 타입을 실제로 생성하며 Next.js의 compiler API 검사도 수행했습니다. 전체 태그에 대한 프로덕션 빌드와 브라우저 검사는 수행하지 않았습니다.

## 변경과 학습 목적

각 강의 스냅샷에서 의존성, lockfile, 패키지 매니저 설정을 갱신했습니다. 최종 단계에 최신화 커밋을 추가하는 방식이 아닙니다. 원본 저장소는 수정하지 않았습니다.

동적 경로를 사용하는 effect에는 id/postId 의존성을 추가했습니다. 응답 콜백의 any는 실제 사용하는 `{ msg: string }`으로 명시했습니다. 13강의 최초 상태 갱신 예제는 취소할 수 있는 비동기 타이머를 사용해 effect 안의 동기 setState 경고를 해소했습니다.

학습 목적을 보존하기 위한 제한된 lint 예외가 있습니다. 006의 일반 `<a>`는 007의 `Link`와 비교하기 위한 예제이므로 해당 줄에 이유를 기록했습니다. 013-3~013-5의 any 인덱스 시그니처는 타입 비교 수업의 내용이므로 해당 선언 줄에만 이유를 기록했습니다. 전역 lint 규칙은 완화하지 않았습니다.

## 빌드 환경 제한

첫 001 빌드는 Windows의 가용 가상 메모리가 적은 상태에서 Turbopack 메모리 할당 실패로 중단됐습니다. 해당 시도는 실패로 기록했습니다. 다음 환경 변수로 병렬도와 Node 메모리를 제한한 뒤 대표 3단계 모두 빌드를 통과했습니다.

```powershell
$env:RAYON_NUM_THREADS = '2'
$env:NODE_OPTIONS = '--max-old-space-size=2048'
pnpm build
```

이 설정은 이번 검증 프로세스에 적용했으며 운영체제나 보안 프로그램 설정은 변경하지 않았습니다.

백엔드 068의 개발 DB는 `jdbc:h2:./db_dev`, 테스트 DB는 `jdbc:h2:mem:db_test`입니다. AUTO_SERVER 또는 MODE=MySQL을 추가하지 않았습니다. db_dev.lock.db는 이미 백엔드 .gitignore에 포함되어 있어 백엔드 원격 코드를 수정할 필요가 없었습니다.

## 공식 자료

- [Node.js 26.13.1 LTS](https://nodejs.org/en/blog/release/v26.13.1)
- [Next.js 설치와 Windows 지원](https://nextjs.org/docs/app/getting-started/installation)
- [Microsoft TypeScript 7/6 alias 안내](https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/)
- [Tailwind PostCSS](https://tailwindcss.com/docs/installation/using-postcss)
- [pnpm 설정](https://pnpm.io/settings)
