# Component Structure — FE

> **Role**: Frontend
> **Nguồn tham chiếu**: `.claude/FE-architecture.md` (Next.js 15 App Router + React 19 + TypeScript)

**Phạm vi áp dụng**: tất cả file trong `src/app/`, `src/components/`.
**Mức độ bắt buộc**: **MUST — vi phạm = block PR**.

---

## Rule 1 — Named export cho component thường; default export CHỈ cho file Next.js đặc biệt

File đặc biệt cần default export: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, `template.tsx`. Còn lại PHẢI named export.

Lý do: named export giúp grep tên component dễ và rename an toàn; App Router yêu cầu default ở các file đặc biệt.

### ❌ Sai

```tsx
// src/components/vocabulary/word-card.tsx
export default function WordCard() { /* ... */ }
```

### ✅ Đúng

```tsx
// src/components/vocabulary/word-card.tsx
export function WordCard() { /* ... */ }
```

```tsx
// src/app/folders/page.tsx
export default function FoldersPage() { /* ... */ }
```

---

## Rule 2 — Server Component là mặc định; CHỈ thêm `'use client'` khi thực sự cần

Cần `'use client'` khi: hook React (`useState`, `useEffect`, `useQuery`, Zustand), event handler DOM, browser API (Web Speech, papaparse, `window`).

Lý do: Server Component giảm bundle JS gửi client → load nhanh hơn.

### ❌ Sai

```tsx
// src/app/folders/page.tsx
'use client';                              // ❌ chỉ render list, không cần client
import { FolderList } from '@/components/folders/folder-list';

export default function FoldersPage() {
  return <FolderList />;
}
```

### ✅ Đúng

```tsx
// src/app/folders/page.tsx — Server Component
import { FolderList } from '@/components/folders/folder-list';

export default function FoldersPage() {
  return <FolderList />;
}
```

```tsx
// src/components/folders/folder-list.tsx — cần useQuery → client
'use client';
import { useFolders } from '@/hooks/use-folders';

export function FolderList() {
  const { data, isLoading } = useFolders();
  if (isLoading) return <p>Đang tải...</p>;
  return <ul>{data?.map((f) => <li key={f.id}>{f.name}</li>)}</ul>;
}
```

---

## Rule 3 — Thứ tự khai báo trong file: `'use client'` → imports → types → constants → component → sub-components

Lý do: scan từ trên xuống là biết ngay context, dependency, shape props.

### ❌ Sai

```tsx
export function WordCard(props: any) { /* ... */ }
import { Button } from '@/components/ui/button';
const MAX_LEN = 50;
type WordCardProps = { word: string };
'use client';
```

### ✅ Đúng

```tsx
'use client';

import { Button } from '@/components/ui/button';
import { speak } from '@/lib/speech';

type WordCardProps = {
  word: string;
  ipa?: string;
  onFlip: () => void;
};

const MAX_WORD_LEN = 50;

export function WordCard({ word, ipa, onFlip }: WordCardProps) {
  return (
    <article>
      <header>{word.slice(0, MAX_WORD_LEN)}</header>
      {ipa && <small>{ipa}</small>}
      <Button onClick={() => speak(word)}>🔊</Button>
      <Button onClick={onFlip}>Flip</Button>
    </article>
  );
}
```

---

## Rule 4 — Props PHẢI có type explicit; CẤM `any`; inline type chỉ khi ≤ 2 field

Lý do: Props là contract; `any` phá toàn bộ type-safety. Inline type dài làm signature khó đọc.

### ❌ Sai

```tsx
export function QuizRunner(props: any) {
  return <div>{props.questions.length}</div>;
}

export function FolderItem({ id, name, description, wordCount, createdAt }: {
  id: string; name: string; description?: string; wordCount: number; createdAt: string;
}) { /* ... */ }
```

### ✅ Đúng

```tsx
import type { Folder, Vocabulary } from '@/types/api';

type QuizRunnerProps = {
  questions: Vocabulary[];
  onSubmit: (correct: number) => void;
};

export function QuizRunner({ questions, onSubmit }: QuizRunnerProps) { /* ... */ }

type FolderItemProps = {
  folder: Folder & { wordCount: number };
};

export function FolderItem({ folder }: FolderItemProps) { /* ... */ }
```

---

## Rule 5 — Component > 150 dòng PHẢI tách sub-component hoặc extract hook

Lý do: file dài che đi luồng dữ liệu; tách giúp test và đọc.

### ❌ Sai

```tsx
// src/components/quiz/quiz-runner.tsx — 300 dòng làm mọi thứ
'use client';
export function QuizRunner({ questions }: QuizRunnerProps) {
  // 50 dòng state
  // 80 dòng logic shuffle, score, timer
  // 120 dòng JSX
  // 50 dòng modal kết quả
}
```

### ✅ Đúng

```tsx
// src/components/quiz/quiz-runner.tsx
'use client';
import { useQuizSession } from '@/hooks/use-quiz-session';
import { QuestionView } from './question-view';
import { QuizResultDialog } from './quiz-result-dialog';

export function QuizRunner({ questions }: QuizRunnerProps) {
  const { current, answer, isFinished, score } = useQuizSession(questions);
  return (
    <>
      {!isFinished && <QuestionView question={current} onAnswer={answer} />}
      <QuizResultDialog open={isFinished} score={score} total={questions.length} />
    </>
  );
}
```

---

## Rule 6 — Logic stateful PHẢI tách custom hook nếu reuse > 1 nơi HOẶC > 30 dòng

Lý do: tách logic khỏi presentation; dễ test và reuse.

### ❌ Sai

```tsx
'use client';
export function FlashcardDeck({ words }: { words: Vocabulary[] }) {
  const [index, setIndex] = useState(0);
  const [flipped, setFlipped] = useState(false);
  // 40 dòng next/prev/shuffle/keyboard handlers
  return <FlashcardItem {...} />;
}
```

### ✅ Đúng

```ts
// src/hooks/use-flashcard-deck.ts
import { useState } from 'react';
import type { Vocabulary } from '@/types/api';

export function useFlashcardDeck(words: Vocabulary[]) {
  const [index, setIndex] = useState(0);
  const [isFlipped, setIsFlipped] = useState(false);
  const next = () => { setIsFlipped(false); setIndex((i) => (i + 1) % words.length); };
  return { current: words[index], isFlipped, flip: () => setIsFlipped((f) => !f), next };
}
```

```tsx
// src/components/flashcard/flashcard-deck.tsx
'use client';
import { useFlashcardDeck } from '@/hooks/use-flashcard-deck';

export function FlashcardDeck({ words }: { words: Vocabulary[] }) {
  const { current, isFlipped, flip, next } = useFlashcardDeck(words);
  return <FlashcardItem word={current} isFlipped={isFlipped} onFlip={flip} onNext={next} />;
}
```

---

## Rule 7 — Co-locate test cạnh component

Lý do: di chuyển component → di chuyển artefact liên quan; không hunt file ở `__tests__/` xa.

### ❌ Sai

```
src/components/vocabulary/word-card.tsx
__tests__/components/vocabulary/word-card.test.tsx
```

### ✅ Đúng

```
src/components/vocabulary/
  word-card.tsx
  word-card.test.tsx
```

---

## Rule 8 — Đẩy `'use client'` xuống component lá; CẤM ở root layout/container thuần render con

Lý do: `'use client'` ở root làm cả cây con bị bundle gửi client.

### ❌ Sai

```tsx
// src/app/(dashboard)/layout.tsx
'use client';                              // ❌ Cả layout client-only
import { Sidebar } from '@/components/layout/sidebar';

export default function Layout({ children }: { children: React.ReactNode }) {
  return <div><Sidebar />{children}</div>;
}
```

### ✅ Đúng

```tsx
// src/app/(dashboard)/layout.tsx — Server Component
import { Sidebar } from '@/components/layout/sidebar';

export default function Layout({ children }: { children: React.ReactNode }) {
  return <div><Sidebar />{children}</div>;
}
```

```tsx
// src/components/layout/sidebar.tsx — chỉ phần cần interactivity là client
'use client';
export function Sidebar() { /* useState cho menu mobile */ }
```

---

## Checklist tự kiểm

- [ ] Named export cho component thường; default chỉ ở file Next.js đặc biệt.
- [ ] `'use client'` chỉ thêm khi cần, đẩy xuống lá.
- [ ] Thứ tự: directive → imports → types → constants → component → sub.
- [ ] Props có type rõ, không `any`, inline ≤ 2 field.
- [ ] Component > 150 dòng đã tách sub hoặc extract hook.
- [ ] Logic stateful reuse → custom hook trong `src/hooks/`.
- [ ] File test cạnh component.
