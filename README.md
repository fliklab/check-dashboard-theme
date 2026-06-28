# CHECK Dashboard Theme

Hermes Agent dashboard용 YAML 테마입니다.

- Theme ID: `check`
- Display label: `CHECK`
- 구성: `dashboard-themes/check.yaml` 하나만 복사해도 이식 가능
- 프론트엔드 rebuild: 보통 불필요

## 파일 구성

```text
dashboard-themes/
└── check.yaml
```

## 적용 조건

받는 쪽 Hermes 홈 기준으로 아래 위치에 파일을 넣어야 합니다.

```text
$HERMES_HOME/dashboard-themes/check.yaml
```

예시:

```text
~/.hermes/dashboard-themes/check.yaml
```

프로필 전용으로 쓰는 경우에는:

```text
~/.hermes/profiles/<profile-name>/dashboard-themes/check.yaml
```

그리고 config에서 대시보드 테마를 `check`로 선택해야 합니다.

```bash
hermes --profile <profile-name> config set dashboard.theme check
```

또는 `config.yaml` 기준:

```yaml
dashboard:
  theme: check
```

## 중요한 주의점

현재 Agent Lab 대시보드처럼 실행 프로세스가:

```bash
HERMES_HOME=/Users/test/.hermes
```

를 읽고 있으면, 프로필 내부 파일만으로는 라이브 대시보드에 안 보일 수 있습니다.

이 경우에는 아래 위치가 실제 적용 위치입니다.

```text
/Users/test/.hermes/dashboard-themes/check.yaml
```

반대로 특정 프로필 홈으로 실행되는 대시보드라면:

```text
/Users/test/.hermes/profiles/check-bot-public/dashboard-themes/check.yaml
```

이 위치가 적용 대상입니다.

## 받는 쪽 설치 순서

1. `dashboard-themes/check.yaml` 파일을 `$HERMES_HOME/dashboard-themes/check.yaml` 위치에 복사합니다.
2. `dashboard.theme`을 `check`로 설정합니다.
3. 대시보드를 새로고침합니다.
4. 서버가 테마 목록을 캐시하거나 이미 떠 있는 프로세스가 파일 변경을 못 읽으면 대시보드를 재시작합니다.

## License

MIT
