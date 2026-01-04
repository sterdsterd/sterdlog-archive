---
title: JS 애니메이션 성능 개선하기
date: 2023-05-03
summary: requestAnimationFrame()을 사용해서 애니메이션 성능을 개선해봅시다.
tags:
  - web
  - animation
  - performance
---

# JS Animation 성능 이슈 해결하기

올해(2023년) 초 [개인 웹사이트](sterd.dev)를 개발하며, Landing 페이지에 개인 로고가 애니메이션을 통해 등장하는 식으로 구현을 했다.

처음 개발을 할 때에는 CSS에서 `conic-gradient`의 각도에 `@keyframes`를 먹여서 시간에 따라 서서히 로고가 펼쳐지는 식으로 구현하려 했지만, 아쉽게도 CSS에서 이 방법은 지원하지 않았다.(`@property`를 사용하면 어찌어찌 구현할 수 있는 것 같긴 했는데, 내가 메인으로 쓰는 브라우저인 Safari에서 이를 지원하지 않기 때문에 사실상 이 방법은 쓰지 않는게 좋다고 판단했다.)

그래서 부득이하게 JS로 애니메이션을 구현하게 되었다.

# `setInterval()`로 Animation 구현

```tsx
const [angle, setAngle] = useState<number>(-0.4);

useEffect(() => {
  const c = setInterval(() => {
    if (1 >= angle) setAngle(angle + 0.0025);
  }, 1);

  return () => {
    clearInterval(c);
  };
}, [angle]);
```

일단 처음 작성한 코드의 일부는 위와 같은데, `useEffect` Hook로 `angle`이 변할 때 마다 `setInterval()`을 실행시키고? 바로 `clearInterval()`해버리는 식으로 코드를 작성한 것 같은데, 지금 보니 왜 이렇게 짰는지 모르겠다..

주요 문제는 다음과 같다.

- 디바이스의 성능에 따라 Animation의 속도가 다르다.
- `angle`의 값이 바뀔 때마다 새로운 `setInterval()`이 실행된다.

그 중, 디바이스 성능에 따라 Animation의 속도가 다르다는 점이 (사용자의 입장에서는) 굉장히 큰 문제였는데, `setInterval()`로 매 시간 단위마다 `angle`의 값에 상수 값을 더해주는 식으로 구현하였지만, `setInterval()`이 정확히 특정 시간이 지난 후 실행됨을 보장하지 못하므로, 디바이스의 성능에 따라 `angle`의 값이 바뀌는 주기가 다르다.

아무튼 작동이 되기는 하니깐 냅뒀는데, 이 방법엔 문제가 조금 많아 리팩토링을 하기로 했다.

# `requestAnimationFrame()`으로 Animation 구현

- [`requestAnimationFrame()` Reference](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)

`requestAnimationFrame()`은 애니메이션과 같은 화면 갱신 작업을 처리하기 위해 사용된다. `setInterval()`과 비슷한 기능을 하지만, `requestAnimationFrame()`은 브라우저의 렌더링 엔진에게 애니메이션을 수행하도록 요청하는 방식으로 작동한다. 이 방식은 브라우저가 최적화된 타이밍으로 애니메이션을 처리할 수 있도록 하며, 브라우저가 현재 시간에 맞추어 애니메이션을 갱신하는 것보다 더 부드러운 애니메이션을 구현할 수 있다.

그에 반해 `setInterval()`은 일정한 시간 간격으로 지정된 작업을 반복적으로 실행하는 함수인데, `setInterval()`은 정확한 간격으로 작업을 실행하지 않을 수 있으며, 브라우저가 작업을 처리하는 동안 지연이 발생할 수 있다. 이러한 지연은 애니메이션에 사용하는 데는 적합하지 않으며, 부드러운 애니메이션을 구현하기 위해 `requestAnimationFrame()`을 사용하기로 했다.

```tsx
const refAngle = useRef<number>(-0.4);
const [angle, setAngle] = useState<number>(-0.4);
var then: Date;
var startTime: Date;
var frameCount: number = 0;

const animate = () => {
  if (refAngle.current > 1) return;
  var now: Date = new Date();

  then = new Date(now.getTime() - (elapsed % FPS));

  var sinceStart = now.getTime() - startTime.getTime();
  var currentFps = Math.round((1000 / (sinceStart / ++frameCount)) * 100) / 100;

  setAngle(refAngle.current + 1.25 / currentFps);

  requestAnimationFrame(animate);
};

useEffect(() => {
  then = new Date();
  startTime = then;
  var animationFrame = requestAnimationFrame(animate);

  return () => {
    cancelAnimationFrame(animationFrame);
  };
}, []);

useEffect(() => {
  refAngle.current = angle;
}, [angle]);
```

위 소스는 `requestAnimationFrame()`을 사용하여 애니메이션을 구현한 것이다.
현재 애니메이션의 FPS를 구하고, 그 FPS값에 맞게 `angle`의 값이 변하는 정도를 조절해 프레임 드랍이 일어나더라도 애니메이션의 속도를 일정하게 유지하도록 헀다.

처음에 `requestAnimationFrame()`을 사용하여 애니메이션을 구현할 때, `useRef()` 훅을 사용하지 않고, 바로 `setAngle(angle + 1.25 / currentFps)`과 같은 식으로 State의 값을 변경했는데, 이렇게 작성하면 현재 상태와 이전 상태를 구분할 수 없기 때문에, `angle`의 값이 정상적으로 바뀌지 않았고, 이를 해결하기 위해 `useRef()` 훅을 사용하였다.

# 결과

이렇게 구현한 결과, 처음 로딩할 때처럼 리소스가 많이 필요해 `setInterval()`이 정상적인 인터벌을 두고 실행을 보장할 수 없어 애니메이션이 느려지던 문제와, 매번 새로운 `setInterval()`이 생성되는 문제를 해결할 수 있었다.

# 모바일 환경에서 Animation 프레임 드랍 해결하기

이렇게 수정한 Animation을 deploy해서 모바일 환경에서 구동해보았는데, 데스크탑 환경에서 보던 부드러움이 나오지 않는다. 보기 거북할 정도로 프레임 드랍이 일어나, 문제가 뭘까 생각해보고 해결해보려 한다.

# `new Date()` 쓰지 마세요?

위 문제를 해결하기 위해 Stack Overflow를 뒤져보던 도중, 다음과 같은 글을 발견했다.
[Calculate FPS in Canvas using requestAnimationFrame](https://stackoverflow.com/questions/8279729/calculate-fps-in-canvas-using-requestanimationframe)

기존 코드에서 현재 FPS를 계산하기 위해서 `new Date()`를 사용하고 있는데, `Date` API는 인터벌을 측정하는 데에 적합하지 않다는 것이다.

> The Date-API uses the operating system's internal clock, which is constantly updated and synchronized with NTP time servers. This means, that the speed / frezquency of this clock is sometimes faster and sometimes slower than the actual time - and therefore not useable for measuring durations and framerates.
>
> If someone changes the system time (either manually or due to DST), you could at least see the problem if a single frame suddenly needed an hour. Or a negative time. But if the system clock ticks 20% faster to synchronize with world-time, it is practically impossible to detect.
>
> Also, the Date-API is very imprecise - often much less than 1ms. This makes it especially useless for framerate measurements, where one 60Hz frame needs ~17ms.

위와 같은 이유 때문에, `performance.now()`를 사용하는 것이 좋다고 한다.

- [Performance: `now()` method Reference](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now)

그래서, `new Date()`로 시간을 측정하던 부분을 `performance.now()`로 모두 리팩토링하였고, 작성된 소스코드는 다음과 같다.

```tsx
const refAngle = useRef<number>(-0.4);
const [angle, setAngle] = useState<number>(-0.4);
var then: number;
var startTime: number;
var frameCount: number = 0;

const animate = () => {
  if (refAngle.current > 1) return;

  var now: number = performance.now();

  var sinceStart = now - startTime;

  var currentFps = Math.round((1000 / (sinceStart / ++frameCount)) * 100) / 100;

  refAngle.current += 1.25 / currentFps;
  setAngle(refAngle.current);

  requestAnimationFrame(animate);
};

useEffect(() => {
  then = performance.now();
  startTime = then;

  var animationFrame = requestAnimationFrame(animate);

  return () => {
    cancelAnimationFrame(animationFrame);
  };
}, []);
```

그런데, 위 글에 딸린 Comment들을 읽던 도중 다음과 같은 글을 발견했다.

> Don't use the `Date`, but don't even use `performance.now`. `requestAnimationFrame` passes an highResTimestamp to its callback

`requestAnimationFrame()`이 콜백으로 주는 타임스탬프를 이용하는 것이 더 낫다는 것인데, 이 방법으로도 구현해보고 성능을 테스트 해볼 예정이다.

# 결과

위와 같이 `new Date()`를 `performance.now()`로 변경하는 것 만으로도 모바일 환경에서 프레임 드랍이 일어나던 문제가 어느정도 해결되었다. 그럼에도 불구하고, 60fps가 나오지는 않아서 눈으로 보기에는 여전히 부드럽지는 않은데, 이 부분은 또 어디가 문제인지 다시 확인해볼 생각이다.

관련 문제를 찾아보던 도중, [iOS 자체에서 `requestAnimationFrame()`을 30fps로 Throttle을 건다는 글](https://popmotion.io/blog/20180104-when-ios-throttles-requestanimationframe/)을 발견했다. 이와 관련된 문제인지 다시 한 번 살펴봐야겠다.
