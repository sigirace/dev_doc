## AnimationController



### Ticker

> 매 프레임에 한 번 빠르게 함수를 실행하는 것



### SingleTickerProviderStateMixin

> 현재 트리가 활성화되어 있는 동안만 Ticker를 제공

- 화면을 보지 않을 때, 자원을 아낄 수 있음



### Why use Ticker?

- 애니메이션 컨트롤러가 ticker에 연결됨
- 애니메이션 컨트롤러는 가능한 빨리 실행되어야함 -> 가능한 부드럽게 애니메이션을 실행하고싶기 때문



📒 **AnimationController**

```dart
AnimationController({
    double? value,
    this.duration,
    this.reverseDuration,
    this.debugLabel,
    this.lowerBound = 0.0,
    this.upperBound = 1.0,
    this.animationBehavior = AnimationBehavior.normal,
    required TickerProvider vsync,
  }) : assert(upperBound >= lowerBound),
       _direction = _AnimationDirection.forward {
    assert(debugMaybeDispatchCreated('animation', 'AnimationController', this));
    _ticker = vsync.createTicker(_tick);
    _internalSetValue(value ?? lowerBound);
  }
```

- `AnimationController`를 초기화 할 때, `vsync:this`
- `_tick` 은 애니메이션을 만드는 함수
- `this`로 전달했기에 해당 화면에서만 ticker를 제공함



