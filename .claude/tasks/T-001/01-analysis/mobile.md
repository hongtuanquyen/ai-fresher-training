# Mobile Analysis

> Output của role Mobile. Input: `ba.md` + design tokens + Figma + API docs.
> Phải map tới AC ID trong `ba.md`.

## 1. Tổng quan kỹ thuật

- Screen mới: ...
- Screen sửa: ...
- Module bị ảnh hưởng: ...

## 2. Component tree (mỗi screen)

### Screen: `<ScreenName>`

```
<ScreenName>
├── <HeaderComponent>     [reuse từ design-system]
├── <FormSection>         [tạo mới]
│   ├── <InputField>      [reuse]
│   └── <SubmitButton>    [reuse]
└── <FooterNote>          [tạo mới]
```

- Reuse: liệt kê component từ `.claude/design-system/components.md`
- Tạo mới: liệt kê + lý do (chưa có trong design-system)

## 3. State management

| State | Scope (local/global) | Lưu ở | Cache strategy |
|---|---|---|---|
| ... | ... | Redux/Context/local | ... |

Tham chiếu: `.claude/mobile/conventions/state-management.md`

## 4. Navigation / routing

- Entry point: ...
- Route name + params: ...
- Back behavior: ...
- Deep link (nếu có): ...

## 5. Interaction states (đầy đủ cho mỗi screen/component)

| Component | Loading | Empty | Error | Disabled | Success |
|---|---|---|---|---|---|
| `<Name>` | ... | ... | ... | ... | ... |

## 6. Platform-specific behavior

| Behavior | iOS | Android |
|---|---|---|
| ... | ... | ... |

## 7. Accessibility (a11y)

- Touch target size: ≥ 44x44 pt
- VoiceOver/TalkBack labels: liệt kê element + label
- Contrast: WCAG AA
- Focus order: ...

## 8. i18n keys mới

| Key | en | ja |
|---|---|---|
| `screen.x.title` | ... | ... |

Tham chiếu: `.claude/mobile/conventions/i18n-policy.md`

## 9. API integration

| Screen/Action | Endpoint | Method | Loading UI | Error UI | Retry |
|---|---|---|---|---|---|
| ... | `/...` | GET | skeleton | toast | 3x exp backoff |

Tham chiếu: `.claude/mobile/architecture/api-integration.md`

## 10. Offline behavior (nếu có)

- ...

## 11. Observability

### Analytics Events

| Event | Params | Khi nào fire |
|---|---|---|
| `screen_viewed` | screenName, fromScreen | mount |
| `<event>` | ... | ... |

### Crash Breadcrumbs

- Navigation: screen name
- API: endpoint + status

### Performance Traces

| Trace | Target |
|---|---|
| `screen_x_render` | < 300ms |

Tham chiếu: `.claude/mobile/conventions/monitoring.md`

## 12. Dependencies mới (nếu có)

> Mọi package mới phải có entry trong `dependencies-proposal.md` và pass §8 governance.

- DEP-01: `<package>@<version>` — xem `dependencies-proposal.md`

## 13. Test Plan

### Test pyramid

- Unit: <số lượng>, cover layer (utils, hooks, store)
- Component: <số lượng>, cover screen/component
- E2E: <số lượng>, journey

### Test cases theo AC

| AC ID | Test type | Test name | Fixture/data |
|---|---|---|---|
| AC-01.1 | unit | `should_xxx_when_yyy` | ... |
| AC-01.2 | component | ... | ... |
| AC-02.1 | e2e | ... | ... |

### Non-functional tests

- Performance: cold start, FPS, memory (so với `.claude/mobile/architecture/performance-budget.md`)
- a11y: axe / accessibility inspector

### Risk-based priority

- P0 (must pass trước merge): ...
- P1 (should pass): ...
- P2 (nice to have): ...

## 14. Mapping AC → Component/Screen

| AC ID | Screen | Component | File dự kiến |
|---|---|---|---|
| AC-01.1 | ... | ... | `App/Containers/.../...` |
