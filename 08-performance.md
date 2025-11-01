# Производительность и оптимизация

## Задача 8.1: Мемоизация функций 🟢

**Контекст:** Кэшировать результаты дорогих вычислений.

**Задача:**
```typescript
function memoize<T extends (...args: any[]) => any>(fn: T): T {
  // Создать мемоизированную версию функции
  // Кэшировать результаты по аргументам
}

// Использование:
const expensiveCalculation = (n: number) => {
  console.log('Calculating...');
  return n * n;
};

const memoized = memoize(expensiveCalculation);
memoized(5); // Вычисляет и кэширует
memoized(5); // Возвращает из кэша
memoized(10); // Вычисляет для нового аргумента
```

<details>
<summary>💡 Подсказки</summary>

- Используйте Map для хранения кэша
- Ключ кэша - сериализованные аргументы (JSON.stringify)
- Верните функцию-обертку с той же сигнатурой
- Для сложных объектов рассмотрите WeakMap

</details>


## Задача 8.2: Debounce и Throttle с очисткой 🟡

**Контекст:** Реализовать debounce/throttle с возможностью отмены.

**Задача:**
```typescript
interface DebouncedFunction<T extends (...args: any[]) => any> {
  (...args: Parameters<T>): void;
  cancel(): void;
  flush(): void;
}

function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): DebouncedFunction<T> {
  // Реализовать debounce
  // cancel() - отменить отложенный вызов
  // flush() - немедленно вызвать отложенную функцию
}
```

<details>
<summary>💡 Подсказки</summary>

- Храните ссылку на таймер
- cancel() вызывает clearTimeout
- flush() отменяет таймер и вызывает функцию немедленно
- Сохраняйте последние аргументы для flush

</details>


## Задача 8.3: Ленивая загрузка изображений 🟡

**Контекст:** Загружать изображения только когда они входят во viewport.

**Задача:**
```typescript
class LazyImageLoader {
  constructor(options?: {
    rootMargin?: string; // '50px' - загружать за 50px до появления
    threshold?: number;
  }) {}

  observe(img: HTMLImageElement): void {
    // Наблюдать за img
    // Когда появляется во viewport - загрузить
    // data-src → src
  }

  disconnect(): void {}
}

// HTML: <img data-src="image.jpg" class="lazy">
```

<details>
<summary>💡 Подсказки</summary>

- Используйте IntersectionObserver
- Когда entry.isIntersecting === true, загружайте изображение
- Установите src из data-src
- После загрузки прекратите наблюдение за этим элементом

</details>


## Задача 8.4: Батчинг DOM операций 🟡

**Контекст:** Объединить множественные DOM обновления в одну операцию.

**Задача:**
```typescript
class DOMBatcher {
  private queue: Array<() => void> = [];
  private scheduled = false;

  schedule(callback: () => void): void {
    // Добавить в очередь
    // Запланировать выполнение в следующем animation frame
  }

  private flush(): void {
    // Выполнить все операции из очереди
  }
}

// Использование:
const batcher = new DOMBatcher();

// Вместо 1000 отдельных операций:
for (let i = 0; i < 1000; i++) {
  batcher.schedule(() => {
    const div = document.createElement('div');
    div.textContent = `Item ${i}`;
    container.appendChild(div);
  });
}
// Все выполнится в одном animation frame
```

<details>
<summary>💡 Подсказки</summary>

- Используйте requestAnimationFrame
- Собирайте операции в массив
- Выполняйте все за один раз в RAF callback
- Флаг scheduled предотвращает множественные RAF

</details>


## Задача 8.5: Web Worker для тяжелых вычислений 🔴

**Контекст:** Вынести тяжелые вычисления в Web Worker чтобы не блокировать UI.

**Задача:**
```typescript
// worker.ts
self.onmessage = (e: MessageEvent) => {
  const { type, data } = e.data;

  if (type === 'SORT_LARGE_ARRAY') {
    const sorted = data.sort((a: number, b: number) => a - b);
    self.postMessage({ type: 'SORT_COMPLETE', data: sorted });
  }
};

// main.ts
class WorkerPool {
  private worker: Worker;

  constructor(workerScript: string) {
    this.worker = new Worker(workerScript);
  }

  async sortLargeArray(data: number[]): Promise<number[]> {
    return new Promise((resolve) => {
      this.worker.onmessage = (e) => {
        if (e.data.type === 'SORT_COMPLETE') {
          resolve(e.data.data);
        }
      };
      this.worker.postMessage({ type: 'SORT_LARGE_ARRAY', data });
    });
  }

  terminate(): void {
    this.worker.terminate();
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- Worker работает в отдельном потоке
- Общение через postMessage/onmessage
- Данные копируются (или transferable objects)
- Обрабатывайте разные типы сообщений

</details>


## Задача 8.6: Виртуализация длинного списка 🔴

**Контекст:** Оптимизировать рендеринг списка из 100,000 элементов.

**Задача:**
```typescript
class VirtualList {
  private container: HTMLElement;
  private items: any[];
  private itemHeight: number;
  private visibleCount: number;
  private startIndex = 0;

  constructor(
    container: HTMLElement,
    items: any[],
    itemHeight: number,
    renderItem: (item: any) => string
  ) {
    // Инициализировать
    // Создать wrapper с полной высотой
    // Рендерить только видимые элементы
  }

  private onScroll = (): void => {
    // Пересчитать startIndex
    // Обновить видимые элементы
  };

  private render(): void {
    // Рендерить элементы от startIndex до startIndex + visibleCount
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- Общая высота контейнера = items.length * itemHeight
- startIndex = Math.floor(scrollTop / itemHeight)
- Рендерите startIndex до startIndex + visibleCount + buffer
- Используйте transform: translateY для позиционирования
- См. задачу 4.11 для деталей

</details>


## Задача 8.7: Оптимизация ре-рендеров React (custom hook) 🟡

**Контекст:** Создать hook для предотвращения лишних рендеров.

**Задача:**
```typescript
// Shallow comparison для предотвращения ре-рендеров

function useShallowMemo<T>(value: T): T {
  const ref = useRef<T>(value);

  if (!shallowEqual(ref.current, value)) {
    ref.current = value;
  }

  return ref.current;
}

function shallowEqual(obj1: any, obj2: any): boolean {
  // Сравнить объекты поверхностно
  // Если все ключи и значения равны - true
}

// Использование:
function Component({ filters }) {
  const memoizedFilters = useShallowMemo(filters);

  useEffect(() => {
    fetchData(memoizedFilters);
  }, [memoizedFilters]); // не будет вызываться если filters поверхностно равен
}
```

<details>
<summary>💡 Подсказки</summary>

- Сравните ключи объектов
- Сравните значения по ключам (===)
- Для массивов сравните длину и элементы
- Используйте Object.keys и Object.is

</details>


## Задача 8.8: RequestIdleCallback для фоновых задач 🟡

**Контекст:** Выполнять некритичные задачи когда браузер свободен.

**Задача:**
```typescript
class IdleTaskQueue {
  private tasks: Array<() => void> = [];

  addTask(task: () => void): void {
    this.tasks.push(task);
    this.scheduleWork();
  }

  private scheduleWork(): void {
    // Использовать requestIdleCallback
    // Выполнять задачи пока есть свободное время
  }

  private workLoop(deadline: IdleDeadline): void {
    // Выполнять задачи пока:
    // 1. Есть задачи
    // 2. Есть свободное время (deadline.timeRemaining() > 0)
  }
}

// Использование:
const queue = new IdleTaskQueue();
queue.addTask(() => console.log('Background task 1'));
queue.addTask(() => console.log('Background task 2'));
```

<details>
<summary>💡 Подсказки</summary>

- requestIdleCallback дает время когда браузер свободен
- deadline.timeRemaining() - оставшееся время в мс
- Если timeRemaining() > 0, выполнить еще одну задачу
- Fallback для браузеров без поддержки: setTimeout

</details>


## Задача 8.9: Оптимизация поиска в большом списке 🟡

**Контекст:** Быстрый поиск в списке из 100,000 элементов.

**Задача:**
```typescript
class OptimizedSearch<T> {
  private items: T[];
  private index: Map<string, Set<number>>; // слово → индексы элементов

  constructor(items: T[], getSearchableText: (item: T) => string) {
    this.items = items;
    this.index = this.buildIndex(items, getSearchableText);
  }

  private buildIndex(
    items: T[],
    getSearchableText: (item: T) => string
  ): Map<string, Set<number>> {
    // Построить инвертированный индекс
    // Каждое слово → набор индексов где оно встречается
  }

  search(query: string): T[] {
    // Использовать индекс для быстрого поиска
    // O(1) вместо O(n)
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- Разбейте текст на слова (split, toLowerCase)
- Для каждого слова сохраните индексы элементов
- При поиске найдите индексы по слову в индексе
- Используйте Set для быстрой проверки вхождения

</details>


## Задача 8.10: Code Splitting на уровне функций 🟡

**Контекст:** Загружать код функции только когда она нужна.

**Задача:**
```typescript
function lazyFunction<T extends (...args: any[]) => any>(
  importFn: () => Promise<{ default: T }>
): T {
  let cached: T | null = null;

  return (async (...args: Parameters<T>) => {
    if (!cached) {
      const module = await importFn();
      cached = module.default;
    }
    return cached(...args);
  }) as T;
}

// Использование:
const heavyCalculation = lazyFunction(() =>
  import('./heavy-calculation').then(m => ({ default: m.calculate }))
);

// При первом вызове загрузит модуль
await heavyCalculation(data);
```

<details>
<summary>💡 Подсказки</summary>

- Используйте dynamic import()
- Кэшируйте загруженную функцию
- Оберните в async функцию
- При первом вызове загрузите и кэшируйте

</details>


## Задача 8.11: Оптимизация рендеринга canvas 🔴

**Контекст:** Рисовать только измененные области canvas.

**Задача:**
```typescript
class OptimizedCanvas {
  private ctx: CanvasRenderingContext2D;
  private dirtyRegions: Array<{x: number; y: number; w: number; h: number}> = [];

  constructor(canvas: HTMLCanvasElement) {
    this.ctx = canvas.getContext('2d')!;
  }

  markDirty(x: number, y: number, w: number, h: number): void {
    this.dirtyRegions.push({ x, y, w, h });
  }

  render(drawFn: (ctx: CanvasRenderingContext2D, region: any) => void): void {
    // Рендерить только dirty regions
    // Очистить только измененные области
    // Вызвать drawFn для каждой области
  }

  clear(): void {
    this.dirtyRegions = [];
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- Используйте ctx.save() и ctx.restore()
- ctx.clip() для ограничения области рисования
- Очищайте только dirty region: clearRect(x, y, w, h)
- После рендера очистите массив dirty regions

</details>


## Задача 8.12: Prefetching данных 🟡

**Контекст:** Предзагружать данные для страниц куда пользователь вероятно перейдет.

**Задача:**
```typescript
class DataPrefetcher<T> {
  private cache = new Map<string, Promise<T>>();

  prefetch(key: string, fetcher: () => Promise<T>): void {
    // Начать загрузку данных
    // Сохранить промис в кэш
  }

  async get(key: string, fetcher: () => Promise<T>): Promise<T> {
    // Если в кэше - вернуть
    // Иначе - начать загрузку и вернуть промис
  }

  has(key: string): boolean {
    return this.cache.has(key);
  }
}

// Использование:
const prefetcher = new DataPrefetcher<User>();

// При наведении на ссылку
link.addEventListener('mouseenter', () => {
  prefetcher.prefetch('/api/users/123', () => fetchUser('123'));
});

// При клике данные уже загружены
link.addEventListener('click', async () => {
  const user = await prefetcher.get('/api/users/123', () => fetchUser('123'));
});
```

<details>
<summary>💡 Подсказки</summary>

- Храните Promise в кэше, не данные
- prefetch просто запускает fetcher и сохраняет Promise
- get проверяет кэш или запускает fetcher
- Добавьте очистку старого кэша (TTL)

</details>