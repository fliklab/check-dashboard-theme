# CHECK / FROST2 대시보드 테마 구현 노트

이 문서는 Hermes Agent Dashboard용 CHECK/FROST2 계열 테마를 만들면서 확인한 적용 지점, 의도, 문제와 해결 방식을 정리한 작업 노트입니다. 목표는 **Hermes 소스 코드를 수정하지 않고 `dashboard-themes/check.yaml` 파일 하나만 이전해도 테마가 구현되는 것**입니다.

## 1. 테마 파일과 적용 범위

현재 저장소의 핵심 파일은 아래 하나입니다.

```text
dashboard-themes/check.yaml
```

이 파일은 Hermes Dashboard의 사용자 YAML 테마 포맷을 사용합니다.

```yaml
name: check
label: CHECK
palette: ...
typography: ...
layout: ...
componentStyles: ...
colorOverrides: ...
customCSS: |
  ...
```

설치 대상 Hermes 홈에는 아래처럼 복사합니다.

```text
$HERMES_HOME/dashboard-themes/check.yaml
```

그리고 설정에서 대시보드 테마를 `check`로 선택합니다.

```bash
hermes config set dashboard.theme check
```

프로필 홈을 쓰는 경우에는 다음 위치가 대상입니다.

```text
~/.hermes/profiles/<profile-name>/dashboard-themes/check.yaml
```

단, 실제 실행 중인 dashboard 프로세스가 어떤 `HERMES_HOME`을 읽는지 먼저 확인해야 합니다. Agent Lab처럼 기본 홈 `/Users/test/.hermes`에서 dashboard가 떠 있으면 프로필 내부에만 넣어서는 반영되지 않을 수 있습니다.

## 2. 어디를 바꾸면 어디가 바뀌는가

### 2.1 `palette`

`palette`는 dashboard의 기본 배경/전경 계열 색을 정합니다.

```yaml
palette:
  background:
    hex: '#f6f8fb'
  midground:
    hex: '#111827'
  foreground:
    hex: '#ffffff'
  warmGlow: rgba(255, 255, 255, 0)
  noiseOpacity: 0
```

적용 효과:

- `background`: 전체 앱 캔버스 배경
- `midground`: 기본 진한 텍스트/강조색 계열
- `foreground`: 일부 theme 계산에서 쓰이는 표면색
- `warmGlow`, `noiseOpacity`: 기존 장식성 glow/noise를 줄이는 용도

CHECK/FROST2 방향은 화려한 glow보다 **밝은 SaaS형 배경 + 선명한 텍스트**이므로 배경은 `#f6f8fb`, 텍스트는 `#111827`로 잡았습니다.

### 2.2 `typography`

```yaml
typography:
  fontSans: -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif
  fontMono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono', 'Courier New', monospace
  fontDisplay: -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif
  baseSize: 15px
  lineHeight: '1.5'
  letterSpacing: '0'
```

적용 효과:

- 일반 텍스트, 버튼, 입력 필드, 메뉴의 기본 글꼴을 시스템 UI 폰트로 맞춤
- 기존 theme/source에 남아 있던 decorative/mono/uppercase 느낌을 줄임
- `baseSize: 15px`로 너무 작은 dashboard 텍스트를 완화

문제:

- YAML의 typography token만으로는 Tailwind utility class(`font-mono`, `uppercase`, `tracking-*`)가 항상 덮이지 않았습니다.

해결:

- `customCSS`에서 일반 UI 텍스트에 시스템 폰트를 강제로 적용하고, `code/pre/kbd/samp` 같은 실제 코드성 요소만 monospace를 유지했습니다.

### 2.3 `layout`

```yaml
layout:
  radius: 0.75rem
  density: comfortable
layoutVariant: standard
```

적용 효과:

- 카드, 패널, 입력, 버튼의 기본 radius 기준값
- 전체 dashboard의 밀도와 spacing 방향

주의:

- `radius`를 크게만 잡으면 테이블 행, grid row, segmented control까지 모두 pill처럼 둥글어져 구조가 흐트러집니다.

해결:

- `customCSS`에서 요소 종류별 shape scale을 다시 분리했습니다.
  - 카드/패널: 8~12px 정도의 둥근 모서리
  - 일반 버튼/입력: 8~10px
  - sidebar active/hover, badge, chip, switch: pill
  - 테이블/row/grid/segmented 내부 셀: square 유지

### 2.4 `componentStyles`

```yaml
componentStyles:
  header:
    background: rgba(255, 255, 255, 0.92)
    borderColor: '#dde5ef'
    boxShadow: none
  sidebar:
    background: rgba(255, 255, 255, 0.97)
    borderColor: '#e5eaf1'
    boxShadow: none
  card:
    background: rgba(255, 255, 255, 0.96)
    borderColor: '#e1e7ef'
    boxShadow: none
```

적용 효과:

- 상단 header, 좌측 sidebar, 카드 표면의 기본 배경/테두리/그림자
- CHECK/FROST2의 “밝고 얇은 경계선이 있는 업무용 dashboard” 느낌을 만드는 1차 token

의도:

- 그림자는 거의 제거하고, 얇은 border와 미묘한 surface 차이로 구획을 구분합니다.
- 지나치게 밝거나 낮은 대비의 색은 피했습니다.

### 2.5 `colorOverrides`

```yaml
colorOverrides:
  card: '#ffffff'
  cardForeground: '#111827'
  primary: '#2563eb'
  muted: '#eef2f7'
  mutedForeground: '#4b5563'
  border: '#d4dde8'
  input: '#d4dde8'
  ring: '#2563eb'
```

적용 효과:

- shadcn/Tailwind 계열 CSS variable에 연결되는 기본 semantic color
- 버튼 primary, card, muted text, input border, focus ring 등에 영향

주의:

- 이것만으로는 기존 component class의 hover glow, shadow, `group-hover`, `arc-border` 같은 장식이 완전히 사라지지 않았습니다.

해결:

- 장식 제거는 `customCSS` 후반부에서 별도 처리했습니다.

## 3. `customCSS`에서 해결한 주요 영역

YAML token만으로 구현 가능한 범위를 최대화했지만, Hermes Dashboard는 이미 Tailwind utility와 컴포넌트별 class가 많아서 `customCSS`가 필요했습니다. 이 저장소는 소스 패치 대신 `customCSS`에 이식 가능한 보정값을 담는 방향입니다.

### 3.1 scope 전략: `[data-theme="check"]` + fallback

많은 rule이 아래처럼 작성되어 있습니다.

```css
:is([data-theme="check"], html:not([data-theme])) ... {
  ...
}
```

이유:

- 정상적으로는 dashboard가 `<html data-theme="check">`를 세팅할 때 `[data-theme="check"]`가 매칭됩니다.
- 일부 배포/버전에서는 customCSS는 주입되지만 `<html data-theme>`가 비어 있는 경우가 있었습니다.
- 그 경우 `[data-theme="check"] ...`만 쓰면 CSS가 로드되어도 실제로 아무 것도 바뀌지 않습니다.

해결:

- critical rule에는 `html:not([data-theme])` fallback을 함께 두었습니다.
- 이렇게 하면 customCSS가 주입된 환경에서 data-theme 누락이 있어도 CHECK 스타일이 적용됩니다.

### 3.2 sidebar nav

주요 selector:

```css
#app-sidebar nav a
#app-sidebar nav a:hover
#app-sidebar nav a[aria-current="page"]
#app-sidebar nav a[class*="bg-midground"]
```

바꾼 것:

- 좌측 메뉴 active/hover를 inset pill fill로 통일
- side accent line, decorative absolute span, plus-lighter blend, glow 제거
- hover와 active 상태가 서로 다른 디자인처럼 보이지 않게 중립색 fill로 정리

발견한 문제:

- sidebar nav 안쪽에 decorative child span이 있어서 parent에 배경을 주면 이중 배경/안쪽 박스/옆줄처럼 보였습니다.

해결:

- sidebar nav item 내부의 `span.absolute`, `span.pointer-events-none`, `span[style*="plus-lighter"]`를 숨기고 parent link 하나만 fill을 담당하게 했습니다.

### 3.3 hover/focus glow 제거

주요 selector:

```css
.arc-border
[class*="group-hover"]
[class*="hover:shadow"]
[class*="shadow-"]
button:hover
[role="button"]:hover
a[href]:hover
```

바꾼 것:

- hover/focus/active 상태에서 box-shadow, filter, text-shadow, background-image 제거
- hover는 장식 효과가 아니라 배경색/테두리색 변화로만 표현

발견한 문제:

- token에서 `boxShadow: none`을 줘도 component class의 hover shadow가 나중에 적용되어 glow가 남았습니다.
- `arc-border` 계열 pseudo/child가 glow처럼 보였습니다.

해결:

- 후순위 `customCSS`에서 `!important`로 hover/focus 장식 소스를 제거했습니다.
- 접근성용 focus 자체를 완전히 없애지 않고, 필요한 경우 border/ring 색은 유지할 수 있게 했습니다.

### 3.4 readable text selection

주요 selector:

```css
::selection
::-moz-selection
```

바꾼 것:

- 선택 영역 배경을 `#bfdbfe`, 글자를 `#0f172a`로 지정

발견한 문제:

- 기존 조합에서는 텍스트 드래그 시 선택 색이 너무 흐리거나 글자가 안 보이는 경우가 있었습니다.

해결:

- theme scope 안에서 selection 색을 명시적으로 지정했습니다.

### 3.5 fonts/logo override

바꾼 것:

- 일반 UI 텍스트는 system font
- 코드성 요소만 monospace 유지
- top-left logo/header text가 흰색/낮은 대비로 남는 경우를 강제로 어두운 색으로 보정

발견한 문제:

- `typography` token을 바꿔도 `.font-mono`, `tracking-*`, `uppercase`, `text-midground` utility가 남아 CHECK답지 않은 인상이 유지됐습니다.

해결:

- customCSS에서 일반 UI와 코드 UI를 분리해서 override했습니다.

### 3.6 buttons, chips, badges, status

바꾼 것:

- 일반 버튼: modest radius, 얇은 border, no glow
- badge/chip/status: pill radius 유지
- success/warning/danger/info 계열 색을 CHECK palette에 맞게 안정화

발견한 문제:

- broad selector로 `span[class*="border-"]`처럼 잡으면 내부 텍스트 span까지 chip처럼 변할 수 있습니다.

해결:

- badge/status에 가까운 class 조합 위주로 scope를 좁히고, 일반 row/text span에는 과한 pill styling이 번지지 않도록 했습니다.

### 3.7 table/list/grid shape correction

바꾼 것:

- 테이블 wrapper, log/list row, grid row, segmented/radio control 내부 셀은 square에 가깝게 유지
- outer card/panel만 적당히 둥글게 처리

발견한 문제:

- 전역 radius/pill rule을 넓게 적용하면 파일 목록, 로그 테이블, 설정 row들이 각각 둥근 카드처럼 보여 데이터 구조가 약해졌습니다.

해결:

- row/grid/segmented 내부 item에는 radius를 낮추거나 0으로 보정했습니다.
- pill은 chip/sidebar/switch처럼 “상태/선택” 의미가 분명한 곳에만 남겼습니다.

### 3.8 dropdown clipping / parent overflow

바꾼 것:

- YAML에서 가능한 범위에서는 dropdown host 주변 `overflow: visible`, `z-index`를 보정했습니다.

발견한 문제:

- Models page의 compact dropdown처럼 부모 카드가 `overflow-hidden`이면 `z-index`만 올려서는 메뉴가 부모 밖으로 빠져나오지 못합니다.
- 이런 경우는 theme CSS만으로 완전 해결이 어렵고, upstream Hermes에서는 `document.body` portal + fixed positioning 패턴을 사용합니다.

관련 upstream 패턴:

- Models page `Use as` menu clipping: portal로 body에 렌더링하는 PR 패턴
- Profile model dropdown clipping: scroll container 밖으로 dropdown을 빼는 PR 패턴
- Theme picker: 긴 목록은 `max-height` + `overflow-y-auto`
- Sidebar footer language picker: 아래가 아니라 위로 열리게 drop-up 처리

정리:

- 단순 긴 목록 문제: `max-height`, `overflow-y-auto`
- viewport 하단 문제: drop-up 또는 viewport clamp
- 부모 `overflow-hidden`/stacking context 문제: portal이 정석
- YAML에서는 host overflow/z-index 완화까지만 가능하며, 완전한 floating UI 수정은 TSX component 변경이 필요할 수 있습니다.

## 4. 만들면서 생겼던 문제와 해결 방식

| 문제 | 원인 | 해결 |
| --- | --- | --- |
| customCSS가 로드되는데 스타일이 안 먹음 | `<html data-theme="check">`가 없는 환경 | critical selector에 `html:not([data-theme])` fallback 추가 |
| sidebar active가 박스 안에 또 박스처럼 보임 | nav item 내부 decorative absolute span과 parent background가 중첩 | decorative child span 숨김, parent link fill만 사용 |
| hover 시 glow/shadow가 계속 남음 | Tailwind `group-hover`, `hover:shadow`, `.arc-border`가 token보다 후순위/구체적으로 적용 | 후순위 customCSS에서 shadow/filter/background-image 제거 |
| 드래그 선택 시 글자가 안 보임 | selection color 대비 부족 | `::selection`, `::-moz-selection` 색 명시 |
| YAML typography만 바꿨는데 폰트가 그대로 남음 | utility class(`font-mono`, `uppercase`, `tracking-*`)가 우선 | 일반 UI 폰트 override + code element 예외 처리 |
| 모든 게 너무 둥글어짐 | radius/pill rule이 table/list row까지 번짐 | shape scale 분리, 테이블/row/segmented 내부는 square 보정 |
| dropdown이 부모 박스에 갇힘 | parent `overflow-hidden` 또는 stacking context | YAML에서는 overflow/z-index 완화, 근본 해결은 portal/fixed positioning 필요 |
| built-in `check`와 user YAML `check`가 충돌 가능 | Hermes 내장 theme ID와 사용자 YAML ID가 같으면 우선순위 혼동 | 설치 대상에 따라 `check-portable` 같은 별도 ID 사용 가능. 이 경우 `name`, `dashboard.theme`, CSS scope를 함께 변경 |
| API customCSS 길이 제한/잘림 우려 | served customCSS가 일정 길이에서 잘릴 수 있음 | critical import fix를 customCSS 앞쪽에 배치 |

## 5. 왜 YAML 파일 이전만으로 구현하려 했는가

Hermes dashboard source를 직접 수정하면 다음 문제가 생깁니다.

- dashboard rebuild가 필요합니다.
- upstream update 때 충돌이 납니다.
- 다른 서버/프로필로 옮기기 어렵습니다.
- CHECK/FROST2 같은 운영용 테마를 여러 profile에 배포하기 번거롭습니다.

그래서 이 저장소의 방향은 다음과 같습니다.

1. 가능한 것은 `palette`, `typography`, `layout`, `componentStyles`, `colorOverrides` token으로 처리합니다.
2. token만으로 안 되는 Tailwind/component 잔여 스타일은 `customCSS`에 넣습니다.
3. source-level 수정이 필요한 부분은 문서화하고, YAML에서 가능한 완화까지만 합니다.
4. 최종 산출물은 `dashboard-themes/check.yaml` 단일 파일로 유지합니다.

## 6. 이식/검증 체크리스트

새 Hermes 홈에 적용할 때는 아래를 확인합니다.

```bash
mkdir -p "$HERMES_HOME/dashboard-themes"
cp dashboard-themes/check.yaml "$HERMES_HOME/dashboard-themes/check.yaml"
hermes config set dashboard.theme check
```

검증 항목:

- Dashboard theme picker/API에 `CHECK`가 보이는가
- active theme이 `check`인가
- `customCSS`가 실제로 주입되는가
- sidebar active/hover가 inset pill fill인가
- hover glow/shadow가 남지 않는가
- text selection이 읽히는가
- table/list/grid row가 과하게 pill화되지 않았는가
- dropdown이 단순 z-index 문제인지, parent overflow/portal이 필요한 구조적 문제인지 구분했는가

브라우저 console에서 확인할 수 있는 예:

```js
document.documentElement.dataset.theme
```

그리고 theme API가 있는 환경에서는 `/api/dashboard/themes` 응답에서 `check` theme의 `definition.customCSS`가 비어 있지 않은지 확인합니다.

## 7. 유지보수 원칙

- CHECK/FROST2는 장식보다 가독성과 운영 안정성을 우선합니다.
- 색상 변경은 token에서 먼저 시도합니다.
- 특정 component 문제가 반복되면 broad selector보다 좁은 selector로 보정합니다.
- dropdown clipping처럼 layout 구조 문제는 CSS로 무리하게 숨기지 말고 portal 필요 여부를 문서화합니다.
- built-in theme와 user YAML theme ID가 충돌하는 환경에서는 `check-portable`처럼 ID를 바꾸되, `name`, `dashboard.theme`, `[data-theme="..."]` CSS scope를 모두 함께 바꿉니다.

## 8. 관련 파일

```text
README.md

dashboard-themes/check.yaml

docs/check-theme-implementation-notes.md
```

`README.md`는 설치/적용 요약, 이 문서는 구현 상세와 트러블슈팅 기록입니다.
