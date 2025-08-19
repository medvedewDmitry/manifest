## Фронтенд (JS/TS, React, Vue)

### Цели
- Стабильный UX, производительность, доступность и предсказуемая архитектура.

### Технологии и поддержка версий
- **ЯП**: JavaScript, TypeScript (по умолчанию — TypeScript, `strict: true`).
- **Фреймворки**: React 18+ (поддержка React 16+ с классовыми компонентами), Vue 3 (поддержка Vue 2 с Options API).

### Архитектура и структура
- Проект: `src/` с разделением по фичам или слоям (зафиксировать выбранный подход в README проекта).
- Компоненты: небольшие, переиспользуемые; библиотека UI‑компонентов отдельно.
- Состояние: локальное → контекст → глобальное (Redux Toolkit/Zustand/Pinia) по необходимости.
- Побочные эффекты не в рендере; разделяйте контейнерные и презентационные компоненты.

### JavaScript — стиль и примеры
- Модули: ESM, именованные экспорты.
- Иммутабельность, явная обработка ошибок, `async/await`.

Пример: параллельные запросы и обработка ошибок
```js
export async function loadUserAndPosts(apiClient, userId) {
  try {
    const [user, posts] = await Promise.all([
      apiClient.get(`/users/${userId}`),
      apiClient.get(`/users/${userId}/posts`),
    ]);
    return { ...user, posts };
  } catch (error) {
    console.error('loadUserAndPosts failed', { userId, error });
    throw new Error('FAILED_TO_LOAD_USER');
  }
}
```

Иммутабельное обновление
```js
const next = { ...prev, items: prev.items.map(i => i.id === id ? { ...i, done: true } : i) };
```

### TypeScript — типобезопасность и примеры
- Запрещаем `any`; используем `unknown` + сужение типов.
- Явные типы на границах модулей/публичных API.

Пример: сужение `unknown`
```ts
type User = { id: string; name: string };

export function parseUser(payload: unknown): User {
  if (
    typeof payload === 'object' && payload !== null &&
    'id' in payload && 'name' in payload &&
    typeof (payload as any).id === 'string' &&
    typeof (payload as any).name === 'string'
  ) {
    return payload as User;
  }
  throw new Error('INVALID_USER');
}
```

Пример: дженерики для API
```ts
export async function getJson<T>(url: string): Promise<T> {
  const res = await fetch(url);
  if (!res.ok) throw new Error('HTTP_' + res.status);
  return res.json() as Promise<T>;
}
```

### React — практики и примеры
- Функциональные компоненты и хуки — по умолчанию; мемоизация точечно.
- Динамический импорт тяжёлых компонентов (`React.lazy`, `Suspense`).

Функциональный компонент с `useEffect` и мемоизацией
```tsx
import { useEffect, useMemo, useState } from 'react';

export function TotalPrice({ items }: { items: { price: number }[] }) {
  const [ready, setReady] = useState(false);
  useEffect(() => setReady(true), []);
  const total = useMemo(() => items.reduce((s, i) => s + i.price, 0), [items]);
  return <span>{ready ? total : '…'}</span>;
}
```

Ленивая загрузка
```tsx
import { lazy, Suspense } from 'react';
const Heavy = lazy(() => import('./Heavy'));
export const Screen = () => (
  <Suspense fallback={<div>Loading…</div>}>
    <Heavy />
  </Suspense>
);
```

React Query пример
```tsx
import { useQuery } from '@tanstack/react-query';

export function User({ id }: { id: string }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', id],
    queryFn: () => fetch(`/api/users/${id}`).then(r => r.json()),
  });
  if (isLoading) return <div>…</div>;
  if (error) return <div>error</div>;
  return <div>{data.name}</div>;
}
```

Поддержка старых версий — классовый компонент (React 16+)
```tsx
import React from 'react';
export class Counter extends React.Component<{}, { n: number }> {
  state = { n: 0 };
  render() { return <button onClick={() => this.setState({ n: this.state.n + 1 })}>{this.state.n}</button>; }
}
```

### Vue — практики и примеры
- Vue 3 + Composition API по умолчанию; логика в композиционных функциях.
- Роутинг — Vue Router; глобальное состояние — Pinia по необходимости.

Composition API (`<script setup>`) и ленивый импорт
```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
const items = ref<number[]>([]);
const total = computed(() => items.value.reduce((s, n) => s + n, 0));
onMounted(async () => { items.value = await (await import('./load')).loadItems(); });
</script>

<template>
  <div>{{ total }}</div>
  <button @click="items.push(1)">+1</button>
  <Heavy v-if="items.length > 0" />
</template>
```

Options API (Vue 2) поддержка
```js
export default {
  data() { return { n: 0 }; },
  computed: { doubled() { return this.n * 2; } },
  methods: { inc() { this.n += 1; } },
};
```

Pinia пример
```ts
import { defineStore } from 'pinia';
export const useCart = defineStore('cart', {
  state: () => ({ items: [] as { id: string; price: number }[] }),
  getters: { total: (s) => s.items.reduce((sum, i) => sum + i.price, 0) },
});
```

### Качество и тесты
- ESLint + Prettier; типобезопасность строгая.
- Тесты: unit (Jest/Vitest), компоненты (Testing Library), e2e (Playwright/Cypress).

Пример unit‑теста
```ts
import { loadUserAndPosts } from './api';
test('loads user and posts', async () => {
  const api = { get: jest.fn()
    .mockResolvedValueOnce({ id: '1', name: 'U' })
    .mockResolvedValueOnce([{ id: 'p1' }]) };
  const res = await loadUserAndPosts(api as any, '1');
  expect(res.posts).toHaveLength(1);
});
```

### Процессы
- Ветки: `feature/<ID-задачи-Jira>-<кратко>` → `develop` → `master`; MR небольшие.
- Коммиты: русские описания, указывать ID задачи, пример: `[JIRA-1234] feat(ui): добавить компонент Button`.
- Документация на публичные компоненты: Storybook/MDX‑страницы.


