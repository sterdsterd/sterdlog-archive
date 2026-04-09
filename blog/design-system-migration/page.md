---
title: 디자인 시스템이 바뀌면, 코드는 어떻게 따라가야 하나
date: 2026-04-09
summary: 디자인 시스템 마이그레이션 회고
tags:
  - React Native
  - NativeWind
  - Tailwind
  - Design System
  - ESLint
  - Typography
---

# 들어가며

디자인 시스템이 한 번 바뀌면, 프론트쟁이 입장에서 화면 몇 개 고치는 수준으로 끝나지 않는다.
기존 토큰을 어떻게 정리할지, 새 토큰은 또 어떻게 만들지, 이미 퍼져 있는 레거시는 어떻게 마이그레이션할지까지... 온갖 번뇌를 거치며 수습해내야 한다.

최근에 회사에서 타이포그래피 토큰을 중심으로 디자인 시스템을 갈아엎는 작업을 했는데, 막상 해보니 새 토큰을 만드는 것보다 이미 퍼져 있던 기존 코드를 어떻게 옮길지가 더 까다로웠다.

문제는 이 작업이 토큰 파일 몇 개 고친다고 끝나는 게 아니었다는 점이다.
지금 만들고 있는 앱은 태블릿이 메인 타깃인 수학 학습 앱이고, React Native + NativeWind(tailwindcss) 스택으로 `style`이 아닌 `className` 기반 스타일링을 쓰고 있다.
화면 수가 늘어날수록 타이포 토큰이 화면 곳곳에 퍼질 수밖에 없는 구조인거다.

그러다 보니 화면이 점점 늘어나면서 디자인 시스템 자체의 문제도 슬슬 드러나기 시작했다. (사실 처음부터 좀 이상하긴 했지만...)

# 기존 토큰의 문제

기존 타이포그래피 토큰은 이런 식이었다.

| 토큰          | 크기       | Weight   |
| ----------- | -------- | -------- |
| `text-32b`  | 32px     | Bold     |
| `text-24b`  | 24px     | Bold     |
| `text-20b`  | 20px     | Bold     |
| `text-20r`  | 20px     | Regular  |
| `text-18b`  | 18px     | Bold     |
| `text-18sb` | 18px     | Semibold |
| `text-18m`  | 18px     | Medium   |
| $\vdots$    | $\vdots$ | $\vdots$ |
| `text-12sb` | 12px     | Semibold |
| `text-12m`  | 12px     | Medium   |
| `text-12r`  | 12px     | Regular  |
| `text-10m`  | 10px     | Medium   |
| `text-10r`  | 10px     | Regular  |

사이즈 32px부터 10px까지, 그냥 폰트 크기와 weight 약자를 붙여놓은 이름이고 가능한 거의 모든 조합이 있었다. 여기서 더 모자란 게 있다고? 싶었는데, 작업 중간에 몇몇개가 더 추가된 적도 있었다. 그리고 이름만 봐서는 어디에 써야 하는지 전혀 감이 안 온다.

사실 이쯤되면 이건 디자인 시스템이라 보긴 어렵고 그냥 가능한 모든 경우를 나열한 것에 가깝다고 생각되지만...

이 타이포 토큰으로 제작된 화면이 50개를 넘기니까 디자이너 쪽에서도 문제가 확실히 보이기 시작했다.

같은 리스트 아이템 제목인데도 A 화면에서는 `text-14sb`, B 화면에서는 `text-14m`을 쓰고 있다든가,
같은 계층의 텍스트가 화면마다 다른 크기와 weight로 쓰이고 있었다.
토큰 이름에 의미가 없으니 디자이너도 기준을 잡기 어려웠다.
결국 매번 피그마를 열어서 확인해야 했고, 확인해봐야 일관성이 없는 건 마찬가지였다.

더 심각한 건, 접근성 측면에서 문제가 있는 `text-10r`이나 `text-12r` 같은 토큰이 본문 텍스트로 두루두루 쓰이고 있었다. 토큰 자체에 '이건 caption용이다' 같은 의미가 없으니 아무 데나 들어가고 있었다.
화면이 많아지니까 디자이너도 기준을 놓치는 경우가 생겼고, 개발 쪽에서도 그걸 일일이 잡아내기 어려웠다.

그래서 결국 내가 타이포 토큰 재설계를 밀게 됐다.

# 새로운 토큰을 설계해보자

디자이너에게 현재와 같은 단순 나열 수준의 토큰이 아닌, 시맨틱(Semantic; 의미 단위) 토큰 구조를 제안했고, 초안이 나왔는데 솔직히 초안은 문제가 여럿 있었다.

첫 번째 문제는 토큰과 weight의 관계였다. 토큰 별로 weight 범위를 가지는 게 아니라, `body-1`이 16px bold, `body-2`가 16px regular처럼 토큰 자체가 하나의 weight에 고정되어 있었다.
이 구조면 나중에 16px medium도 필요하면 어떡할건데? 바로 한 칸씩 다 밀리는 거다. weight마다 다른 토큰을 두다보니, 당시 나왔던 토큰이 body만 5개였다.
결국 같은 크기에 weight만 다른 토큰이 위 상황처럼 계속 늘어나거나, 최악이면 토큰을 포기하고 인라인 스타일로 덕지덕지 붙이는 꼴이 될 것 같았다.

두 번째는 행간과 자간이었다. 모든 토큰에 행간 150%, 자간 0이 일괄 적용되어 있었다. 큰 display 텍스트(24px)와 작은 caption(12px)이 같은 행간 비율을 가지면, display는 지나치게 넓어 보이고, body는 그럭저럭 괜찮고, caption은 조금 답답해보인다. 폰트 크기랑 역할이 다른데 행간, 자간이 다 똑같으면 보기 좋을 리가 없다.

이런 부분들을 피드백했고, 몇 번 수정 거쳐서 지금과 같은 구조가 나왔다.

| 토큰        | Weight           | Tablet |      |         | Mobile |      |         |
| --------- | ---------------- | ------ | ---- | ------- | ------ | ---- | ------- |
|           |                  | 크기     | 행간   | 자간      | 크기     | 행간   | 자간      |
| Display 1 | Bold             | 28px   | 36px | -0.84px | 24px   | 32px | -0.72px |
| Title 1   | Bold, SemiBold   | 24px   | 32px | -0.6px  | 22px   | 30px | -0.55px |
| Title 2   | Bold, SemiBold   | 20px   | 28px | -0.4px  | 18px   | 26px | -0.36px |
| Heading 1 | Bold, SemiBold   | 18px   | 28px | -0.18px | 17px   | 26px | -0.17px |
| Heading 2 | Bold, SemiBold   | 16px   | 26px | 0.00px  | 15px   | 24px | 0.00px  |
| Body 1    | Medium, Regular  | 16px   | 26px | 0.00px  | 15px   | 24px | 0.00px  |
| Body 2    | Medium, Regular  | —      | —    | —       | 14px   | 22px | 0.07px  |
| Label     | SemiBold, Medium | —      | —    | —       | 13px   | 20px | 0.13px  |
| Caption   | Medium, Regular  | —      | —    | —       | 12px   | 18px | 0.18px  |

`display → title → heading → body → label → caption` 순서로, 각 계층마다 쓸 수 있는 weight가 정해져 있는 구조다. `heading-1`이 필요하면 `heading-1-bold`와 `heading-1-semibold` 중에서 고르면 된다. 이제는 디자이너도 이건 Heading이니까 Heading 1 Bold 와 같은 식으로 잡으면 되고, 개발자도 이름만 보면 계층과 용도가 바로 보인다.

크기별로 행간과 자간도 다르게 잡혀 있다. display처럼 큰 텍스트는 자간을 좁게(-0.84px), caption처럼 작은 텍스트는 자간을 넓게(+0.18px). 이런 디테일을 토큰 레벨에서 해결해두면, 각 화면에서 일일이 신경 쓸 일이 줄어든다.

개발자가 디자인 시스템 구조에 의견을 내도 될까 싶었지만, 떠나버린 사람들이 남긴 레거시가 결국 전체의 발목을 잡는 상태에서, 누군가는 해결하긴 해야했다.

추가적으로, 토큰은 결국 코드로 구현되기 때문에... 구현 가능성이나 유지보수성을 고려한 피드백은 필요하다고 생각했다.

# 이제 반영해야 하는데...

문제는 기존 토큰을 쓰는 코드가 이미 170개 컴포넌트 파일, 46개 스크린에 걸쳐 퍼져 있었다는 거다.

또한, 전반적으로 디자인 통일성이 많이 약해져있던 상태라, 이에 맞게 모든 화면에 대해 일일이 의미에 맞는 토큰을 새로 부여했기 때문에... 기존 토큰과 새 토큰이 일대일 대응이 안 된다. (당연히 10px짜리 토큰을 새로 만들었을 리가 없으니까...) 

이렇게 바뀐 타이포 토큰에 맞춰서, 전반적인 컴포넌트들에도 padding, size, margin, radius에 대해 약간의 조정이 들어간 상태라, 피그마와 일일이 대조하며 맞춰나가야 하는 상태였다.

디자인 공수도 큰 만큼, 한번에 모든 화면이 리비전되지는 않아서, 한 번에 갈아엎는 대신, 점진적으로 마이그레이션 하기로 했다.

# 기존 토큰은 어떻게 해요?

먼저 기존 토큰들에 `@deprecated` 표시를 달았다.

```ts
/** @deprecated Use typo-* classes instead */
'16b': ['16px', { fontWeight: '700', lineHeight: '24px' }],
/** @deprecated Use typo-* classes instead */
'16sb': ['16px', { fontWeight: '600', lineHeight: '24px' }],
```

그런데 `@deprecated` JSDoc 주석은 `fontSize['16b']`처럼 직접 import해서 참조할 때만 IDE에서 취소선이 그어진다. 실제로 쓰는 건 `className="text-16b"` 같은 문자열이니, className에서는 아무런 경고도 뜨지 않는다. 사실상 별 의미가 없긴 하지만, 그래도 앞으로는 쓰면 안되는 게 맞으니 일단 달아는 놓는 것으로 했다.

 그럼 `className` 내부에서 사용되는 토큰들은 어떻게 처리해야 할까? ESLint를 이리저리 잘만 만져보면 기존 토큰을 사용할 경우 경고를 줄 수 있을거라 생각했고, `eslint-plugin-tailwindcss` docs를 찬찬히 살펴봤다.
 `classnames-order`나 `no-unnecessary-arbitrary-value` 같은 룰까지는 있지만, '특정 토큰을 deprecated로 표시'하거나 '특정 prefix를 강제'하는 룰은 없었다.
 우리한테 필요했던 건 프로젝트 내 토큰 정책을 강제하는 것이었고, 그래서 결국 ESLint 커스텀 룰을 하나 만들어보기로 했다.

생각보다 막 복잡하진 않았다. ESLint flat config 환경에서는 로컬 JS 파일 하나를 플러그인으로 바로 import할 수 있다.

```js
// eslint-plugin-typography.js

const DEPRECATED_TOKENS = [
  '32b', '24b', '20b', '20r', '18b', '18sb', '18m',
  '16b', '16sb', '16m', '16r', '14b', '14sb', '14m', '14r',
  '13b', '13m', '13r', '12sb', '12m', '12r', '10m', '10r',
];

const deprecatedPattern = new RegExp(
  `\\btext-(${DEPRECATED_TOKENS.join('|')})\\b`, 'g'
);
```

위 룰은 `className`에서 `text-16b` 같은 기존 토큰을 발견하면 경고를 띄운다. 이렇게 만든 룰을 아래와 같이 ESLint Config 파일에 추가해주면 된다.

```js
// eslint.config.mjs
import typographyPlugin from './eslint-plugin-typography.js';

// ...
{
  files: ['**/*.{jsx,tsx}'],
  plugins: { typography: typographyPlugin },
  rules: {
    'typography/no-deprecated-font-token': 'warn',  // 지금은 warn
  },
}
```

기존 토큰을 쓰는 파일이 아직 많아서 당장 `error`로 올리면 CI가 터질 것을 감안해 일단 지금은 `warn`으로 뒀는데, 모든 화면의 디자인 리비전이 완료되면 그때 `error`로 올릴 계획이다.

# 새로운 토큰을 만들어보자

## 토큰 정의

위에서 정리한 스펙을 코드로 옮기면 이런 모양이 된다.

```ts
// tokens.ts(일부)
'display-1-bold': ['24px', { fontWeight: '700', lineHeight: '32px', letterSpacing: '-0.72px' }],
'title-1-bold':   ['22px', { fontWeight: '700', lineHeight: '30px', letterSpacing: '-0.55px' }],
'heading-1-bold': ['17px', { fontWeight: '700', lineHeight: '26px', letterSpacing: '-0.17px' }],
'body-1-medium':  ['15px', { fontWeight: '500', lineHeight: '24px' }],
'label-semibold': ['13px', { fontWeight: '600', lineHeight: '20px', letterSpacing: '0.13px' }],
'caption-medium': ['12px', { fontWeight: '500', lineHeight: '18px', letterSpacing: '0.18px' }],
```

tailwind config의 `fontSize`에 등록하면 `text-title-1-bold`, `text-body-1-medium` 같은 식으로 바로 쓸 수 있다. 그런데 이것만으로는 부족했다.

## 왜 `typo-*` 래퍼가 필요했나

이 앱은 태블릿이 메인이긴 하지만 모바일도 같이 대응해야 한다. 처음에는 모바일/태블릿 디자인이 각각 따로 나와 있었지만 배포 일정이 꽤 빡빡했고, 그걸 그대로 구현하려고 보면 내가 뽑아야 할 화면이 사실상 두 배로 늘어나는 구조였다. 일정상 그대로 가기는 어려웠다.

디자인을 다시 보니 완전히 다른 두 벌의 화면이라기보다는, 전반적인 컴포넌트 배치나 구조는 비슷하고 폰트 크기나 간격 같은 값만 달라지는 경우가 많았다. 이 정도면 전체를 따로 구현하는 것보다 반응형으로 흡수하는 게 낫겠다고 판단했다.

그래서 레이아웃은 최대한 공통으로 가져가고, 타이포 토큰만 breakpoint별로 이원화해서 화면 폭에 따라 다르게 적용되도록 했다. 결국 같은 의미의 토큰을 모바일용, 태블릿용으로 각각 두고 breakpoint에 따라 갈아끼우는 방식이다.

```ts
// 모바일 (기본)
'title-1-bold': ['22px', { fontWeight: '700', lineHeight: '30px', letterSpacing: '-0.55px' }],

// 태블릿 (740px+)
'title-1-bold-tablet': ['24px', { fontWeight: '700', lineHeight: '32px', letterSpacing: '-0.6px' }],
```

tailwind의 `text-*` 유틸리티에 이 토큰들을 등록하면 `text-title-1-bold`로 쓸 수 있다. 근데 이건 모바일 값만 적용되고, 태블릿 대응을 하려면 매번 이렇게 써야 한다.

```tsx
<Text className="text-title-1-bold md:text-title-1-bold-tablet">제목</Text>
```

타이포가 들어가는 모든 곳에 두 개씩. 하나라도 빠뜨리면 태블릿에서 모바일 크기로 렌더되거나, 모바일에서는 기본 폰트로 출력될 거다. 실수하기도 쉽고, **솔직히 귀찮다.** `className`도 괜히 길어져서 가독성도 안 좋다.

결국 `typo-*`라는 래퍼 클래스를 만들어서, `typo-title-1-bold` 하나만 쓰면 알아서 전환되게 해보기로 했다.

### 1트: CSS 커스텀 클래스 (`@layer components`)

처음에는 CSS 파일을 만들어서 `@apply`로 조합했다.

```css
/* typography.css */
@layer components {
  .typo-title-1-bold {
    @apply text-title-1-bold;
  }
  .typo-body-1-medium {
    @apply text-body-1-medium;
  }
  /* ... 17개 토큰 */
}

@media (min-width: 740px) {
  .typo-title-1-bold {
    @apply text-title-1-bold-tablet;
  }
  .typo-body-1-medium {
    @apply text-body-1-medium-tablet;
  }
  /* ... 태블릿 오버라이드가 있는 토큰들 */
}
```

17개 토큰 $\times$ 모바일/태블릿이니 약 130줄짜리 CSS 파일이 만들어졌다. 일단 얼추 돌아는 가긴 하는데, 바로 문제를 발견해버렸다.

> **IDE에서 `typo-*` 클래스의 autocomplete가 동작하지 않는다!**

`@layer components`로 등록한 클래스는 tailwind Language Server가 인식을 못하더라. `className`을 쓸 때 autocomplete가 안 되면 매번 토큰 이름을 외우거나 CSS 파일을 열어봐야 하고, 오타가 나도 아무런 에러가 없다. 존재하지 않는 클래스를 써도 그냥 스타일이 안 먹을 뿐이니, 디버깅 시간만 늘어난다.

거기다 태블릿에서는 미디어 쿼리가 안 먹고 모바일 토큰으로 렌더되는 문제도 있었다. responsive가 목적인데 responsive가 안 되면 의미가 없는데?

잠깐 생각해보면 Nativewind가 CSS `@layer`의 미디어 쿼리를 네이티브 런타임으로 변환하는 과정에서 발생하지 않을까 싶긴한데, 어차피 autocomplete가 안 되는 순간 이건 아니다 싶어서 바로 버려버려서 이유는 자세하게 알아보지는 않았다.

## 2트: tailwind Plugin (`addUtilities`)

tailwind의 [Plugin API](https://v3.tailwindcss.com/docs/plugins)를 쓰면 `addUtilities`로 커스텀 유틸리티를 등록할 수 있다. 이렇게 등록된 유틸리티는 tailwind Language Server가 정상적으로 인식하고, Nativewind 런타임에서도 안정적으로 동작했다.

```js
// tailwind.config.js
import { fontSize } from './src/theme/tokens';

const typoTokenMap = [
  ['display-1-bold',     'display-1-bold-tablet'],
  ['title-1-bold',       'title-1-bold-tablet'],
  ['title-1-semibold',   'title-1-semibold-tablet'],
  ['title-2-bold',       'title-2-bold-tablet'],
  ['title-2-semibold',   'title-2-semibold-tablet'],
  ['heading-1-bold',     'heading-1-bold-tablet'],
  ['heading-1-semibold', 'heading-1-semibold-tablet'],
  ['heading-2-bold',     'heading-2-bold-tablet'],
  ['heading-2-semibold', 'heading-2-semibold-tablet'],
  ['body-1-medium',      'body-1-medium-tablet'],
  ['body-1-regular',     'body-1-regular-tablet'],
  ['body-2-medium',      null],  // 태블릿과 동일
  ['body-2-regular',     null],
  ['label-semibold',     null],
  ['label-medium',       null],
  ['caption-medium',     null],
  ['caption-regular',    null],
];

function buildTypoUtilities() {
  const utilities = {};

  for (const [mobileKey, tabletKey] of typoTokenMap) {
    const [mobileSize, mobileAttrs] = fontSize[mobileKey];
    const base = { fontSize: mobileSize, ...mobileAttrs };

    if (tabletKey) {
      const [tabletSize, tabletAttrs] = fontSize[tabletKey];
      base['@media (min-width: 740px)'] = {
        fontSize: tabletSize,
        ...tabletAttrs,
      };
    }

    utilities[`.typo-${mobileKey}`] = base;
  }

  return utilities;
}

module.exports = {
  // ...
  plugins: [
    function ({ addUtilities }) {
      addUtilities(buildTypoUtilities());
    },
  ],
};
```

위와 같이, `tokens.ts`에서 정의된 `fontSize` 값을 직접 가져와서 유틸리티를 생성한다. CSS 파일에서 `@apply`로 하나하나 매핑하던 130줄이, config 안의 함수 하나로 줄일 수 있었다. 역시 사람은 배워야... 만약 토큰이 추가되더라도, `typoTokenMap` 배열에 한 줄만 넣으면 된다.

`body-2`, `label`, `caption`처럼 태블릿에서도 값이 동일한 토큰은 tablet 매핑을 `null`로 두면 미디어 쿼리 없이 모바일 값과 동일한 값이 적용된다.

이제 `typo-title-1-bold`를 입력하면 autocomplete도 되고, breakpoint에 따라 값도 정확히 전환된다. 1트때 만든 `typography.css`는 삭제했다.

정리하면, CSS `@layer components`로 커스텀 클래스를 만드는 방법과 Tailwind Plugin의 `addUtilities`로 유틸리티를 등록하는 방법은 결과물만 보면 비슷해 보여도 개발 경험 측면에서 완전히 다르다. 특히 NativeWind 환경에서는 후자가 autocomplete, responsive 처리 둘 다 더 안정적이었다.

# 커스텀 토큰 강제하기

열심히 새 토큰을 쓰라고 만들어놔도 결국 사람은 실수를 하게 된다. 나만 신경쓰면 되지 보다는 다른 팀원도 변경된 기준에 어떻게든 잘 맞춰서 코드를 작성하는 게 중요하다고 생각했다.

기존 코드가 전부 `text-*` 패턴이다 보니, 관성 때문에라도 `typo-title-1-bold`를 써야 하는데 `text-title-1-bold`를 쓸 수 있다. NativeWind의 `text-*`는 기존 `fontSize` 토큰을 그대로 참조하기 때문에 에러는 없이 어떻게든 동작하겠지만, responsive 처리가 빠진다. 또 이거 못보고 잘못 짚어서 디버깅 하려 난리치면 골때릴 게 틀림없다.

이걸 방지하기 위해 두 번째 ESLint 룰을 만들었다.

```js
const TYPOGRAPHY_TOKEN_PREFIXES = [
  'display-1-', 'title-1-', 'title-2-',
  'heading-1-', 'heading-2-',
  'body-1-', 'body-2-',
  'label-', 'caption-',
];

const directTypographyPattern = new RegExp(
  `\\btext-(${TYPOGRAPHY_TOKEN_PREFIXES.map(p => p.replace('-', '\\-')).join('|')})\\S*\\b`,
  'g'
);
```

이 룰은 `className`에서 `text-title-1-bold` 와 같이 새 토큰을 `text-*` 꼴로 사용하는 것을 감지하면, `typo-title-1-bold`를 쓰라는 에러를 띄운다.

```js
rules: {
  'typography/no-deprecated-font-token': 'warn',     // 기존 토큰 → 경고
  'typography/no-direct-typography-token': 'error',  // text-* 직접 사용 → 에러
}
```

기존 토큰은 아직 이관 중이니 `warn`이지만, 새 토큰을 `text-*`로 쓰는 건 명백한 실수니까 처음부터 `error`로 잡았다.

ESLint 에러 메시지도 조금 신경을 썼다. `className` 문자열 전체에 밑줄이 그어지면 어디가 문제인지 찾기 어렵다. 그래서 문제가 되는 토큰의 정확한 위치에만 밑줄이 그어지도록 위치 계산을 넣었다.

```js
function computeLoc(node, offsetInValue, matchLength, valueStartOffset) {
  const startCol = node.loc.start.column + valueStartOffset + offsetInValue;
  return {
    start: { line: node.loc.start.line, column: startCol },
    end: { line: node.loc.start.line, column: startCol + matchLength },
  };
}
```

이걸로 `className="flex text-title-1-bold text-gray-900"`이라고 쓰면 `text-title-1-bold` 부분에만 정확히 에러가 표시된다.

# 하다 보니 느낀 점들

이번 작업을 하면서 느낀 건데, 토큰 개발에 쏟은 시간보다, 팀 내 컨벤션 유지를 위해 작업한 시간이 더 길더라.

사람은 '앞으로 새 토큰은 `typo-*`로만 쓴다.' 처럼 팀 내 컨벤션을 기억하고는 있다. 그런데 기존 코드베이스 전체가 `text-16b` 같은 패턴으로 가득 차 있으면, 관성에 따라 무심코 기존 관습을 따라가는 경우가 생긴다. 특히 주변 코드를 참고해서 비슷하게 작성하다 보면 더 그렇다.

AI 에이전트를 같이 쓰는 환경에서는 이 문제가 더 커진다. 에이전트는 기존 코드를 기반으로 새 코드를 작성하는데, 코드베이스의 대부분이 옛날 토큰을 쓰고 있으면 그게 곧 '이 프로젝트의 패턴'이 된다. `CLAUDE.md`나 컨텍스트에 '`typo-*`를 써라'라고 적어놔도, 컨텍스트가 길어지거나 복잡해지면 결국 기존 코드 패턴을 따라가는 경우가 종종 있더라.

lint rule을 걸어두면 이 문제를 꽤나 구조적으로 막을 수 있다. 사람이 실수하든, 에이전트가 잘못된 패턴을 따라가든, `text-title-1-bold`를 쓰면 에러가 뜬다. 사람이든 에이전트든 lint 에러가 나면 그걸 또 고치려 든다. 컨텍스트에 적어두는 것보다 lint error로 막는 쪽이 훨씬 확실했다.

결국 팀 컨벤션도 다 사람 기억에만 맡기면 안 되는 것 같다는 걸 또 한 번 느끼는 계기가 되었다. 도구로 강제할 수 있는 건 강제해두는 게 맞지 않을까? 토큰 사용 규칙이 그런 종류였고, 이 경우엔 lint rule이면 충분했다.

# 마치며

이번 작업이 엄청 거창한 건 아니었다. 토큰 정의하고, tailwind 플러그인 만들고, ESLint 룰 두 개 작성한 게 전부다. 그래도 이것만으로 새 코드는 새 규칙을 따르게 하고, 기존 코드는 건드리지 않는 점진적인 마이그레이션 구조는 만들 수 있었다.

CSS `@apply`로 시작해서 tailwind Plugin으로 갈아탄 것도, 다 해보면서 정리된 것들이다. 처음부터 정답을 알고 시작한 건 아니었고, 이번에도 이것저것 해보다가 맞는 방향으로 수렴한 쪽에 가깝다.

컨벤션 문서 한 장 쓰는 것보다 lint 룰 하나가 더 확실하더라는 게, 이번에 가장 크게 느낀 점이다. 사람이든 에이전트든, 결국 코드를 쓰는 건 도구 위에서니까. 규칙을 도구에 녹여두면 그게 제일 오래 가는 규칙이 아닐까?
