# Performance Report — <task-id>

> Chỉ tạo nếu task đụng hot path (startup, navigation transition, animation, scroll, list, image-heavy).
> Đối chiếu budget trong `.claude/mobile/architecture/performance-budget.md`.

## Test environment

- Device iOS: <model> / iOS <version>
- Device Android: <model> / Android <version>
- Build variant: dev | staging | release

## Cold start

| Metric | Budget | iOS measured | Android measured | Pass? |
|---|---|---|---|---|
| Time to interactive | < 2s | <Nms> | <Nms> | ✅/❌ |

## FPS (animation/scroll)

| Screen / Action | Budget | iOS | Android | Pass? |
|---|---|---|---|---|
| `<Screen>` scroll | ≥ 55 fps | <N> | <N> | ✅/❌ |

## Bundle size

| Platform | Before | After | Delta | Budget | Pass? |
|---|---|---|---|---|---|
| iOS | ... | ... | +X KB | ... | ✅/❌ |
| Android | ... | ... | +X KB | ... | ✅/❌ |

## Memory

| Scenario | Peak | Budget | Pass? |
|---|---|---|---|
| `<scenario>` | <N> MB | <N> MB | ✅/❌ |

## Screen render traces

| Trace | Target | Measured | Pass? |
|---|---|---|---|
| `screen_x_render` | < 300ms | <Nms> | ✅/❌ |

## Kết luận

- [ ] Tất cả metric trong budget
- [ ] Có metric vượt budget — đã được human approve override? (Y/N) — lý do: ...

## Tools used

- iOS: Instruments / Xcode profiler
- Android: Android Studio Profiler / Perfetto
- Bundle: `react-native-bundle-visualizer` / Android `bundletool`
