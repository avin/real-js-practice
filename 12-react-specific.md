# React специфичные задачи

## Задача 12.1: Custom hook для fetch данных 🟢

**Контекст:** Переиспользуемый hook для загрузки данных с обработкой состояний.

**Задача:**
```typescript
interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

function useFetch<T>(url: string, options?: RequestInit): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = useCallback(async () => {
    // Загрузить данные
    // Обработать loading, error, data states
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
}

// Использование:
function UserProfile({ userId }: { userId: string }) {
  const { data: user, loading, error, refetch } = useFetch<User>(`/api/users/${userId}`);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return null;

  return <div>{user.name}</div>;
}
```

<details>
<summary>💡 Подсказки</summary>

- Используйте useState для data, loading, error
- useEffect для запуска fetch при монтировании
- useCallback для функции refetch
- Обработайте cleanup (AbortController)
- Обновляйте состояния последовательно

</details>
---

## Задача 12.2: useDebounce hook 🟢

**Контекст:** Hook для debounced значения (см. задачу 10.15, но с деталями).

**Задача:**
```typescript
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Использование в поиске:
function SearchComponent() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (debouncedQuery) {
      searchAPI(debouncedQuery);
    }
  }, [debouncedQuery]);

  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

<details>
<summary>💡 Подсказки</summary>

- useState для debouncedValue
- useEffect с зависимостями [value, delay]
- setTimeout для задержки
- Cleanup функция с clearTimeout

</details>
---

## Задача 12.3: usePrevious hook 🟢

**Контекст:** Получить предыдущее значение prop или state.

**Задача:**
```typescript
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}

// Использование:
function Counter({ count }: { count: number }) {
  const prevCount = usePrevious(count);

  return (
    <div>
      Current: {count}, Previous: {prevCount}
    </div>
  );
}
```

<details>
<summary>💡 Подсказки</summary>

- Используйте useRef для хранения предыдущего значения
- useEffect обновляет ref после рендера
- Вернуть ref.current (значение до обновления)

</details>
---

## Задача 12.4: useLocalStorage hook 🟡

**Контекст:** Синхронизировать state с localStorage.

**Задача:**
```typescript
function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value: T | ((prev: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  // Синхронизация между вкладками
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === key && e.newValue) {
        setStoredValue(JSON.parse(e.newValue));
      }
    };

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, [key]);

  return [storedValue, setValue];
}

// Использование:
function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  return <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>{theme}</button>;
}
```

<details>
<summary>💡 Подсказки</summary>

- Инициализация из localStorage в useState
- При изменении сохранять в localStorage
- Слушать storage event для синхронизации между вкладками
- Обрабатывать ошибки JSON.parse

</details>
---

## Задача 12.5: useIntersectionObserver hook 🟡

**Контекст:** Отслеживать видимость элемента во viewport.

**Задача:**
```typescript
interface UseIntersectionObserverOptions {
  threshold?: number;
  root?: Element | null;
  rootMargin?: string;
}

function useIntersectionObserver(
  ref: RefObject<Element>,
  options: UseIntersectionObserverOptions = {}
): boolean {
  const [isIntersecting, setIsIntersecting] = useState(false);

  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        setIsIntersecting(entry.isIntersecting);
      },
      options
    );

    observer.observe(element);

    return () => {
      observer.disconnect();
    };
  }, [ref, options.threshold, options.root, options.rootMargin]);

  return isIntersecting;
}

// Использование (lazy loading):
function LazyImage({ src }: { src: string }) {
  const ref = useRef<HTMLDivElement>(null);
  const isVisible = useIntersectionObserver(ref);

  return (
    <div ref={ref}>
      {isVisible && <img src={src} alt="" />}
    </div>
  );
}
```

<details>
<summary>💡 Подсказки</summary>

- Создайте IntersectionObserver в useEffect
- Наблюдайте за ref.current
- Обновляйте state при изменении isIntersecting
- Cleanup: observer.disconnect()

</details>
---

## Задача 12.6: useWindowSize hook 🟢

**Контекст:** Отслеживать размер окна браузера.

**Задача:**
```typescript
interface WindowSize {
  width: number;
  height: number;
}

function useWindowSize(): WindowSize {
  const [size, setSize] = useState<WindowSize>({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    // Debounce для производительности
    const debouncedResize = debounce(handleResize, 100);

    window.addEventListener('resize', debouncedResize);
    return () => window.removeEventListener('resize', debouncedResize);
  }, []);

  return size;
}

// Использование:
function ResponsiveComponent() {
  const { width } = useWindowSize();
  return <div>{width < 768 ? 'Mobile' : 'Desktop'}</div>;
}
```

<details>
<summary>💡 Подсказки</summary>

- useState с начальными размерами
- Слушайте resize event
- Используйте debounce для оптимизации
- Cleanup: removeEventListener

</details>
---

## Задача 12.7: Оптимизация Context с разделением 🟡

**Контекст:** Избежать лишних ре-рендеров при использовании Context.

**Задача:**
```typescript
// Плохо: один большой Context
const AppContext = createContext<{
  user: User;
  theme: Theme;
  settings: Settings;
}>(null!);

// Хорошо: разделить на несколько контекстов
const UserContext = createContext<User>(null!);
const ThemeContext = createContext<Theme>(null!);
const SettingsContext = createContext<Settings>(null!);

// Еще лучше: Context + selector
interface AppState {
  user: User;
  theme: Theme;
  settings: Settings;
}

const AppStateContext = createContext<AppState>(null!);
const AppDispatchContext = createContext<Dispatch<Action>>(null!);

function useAppSelector<T>(selector: (state: AppState) => T): T {
  const state = useContext(AppStateContext);
  const selectedValue = selector(state);

  // Мемоизация для предотвращения ре-рендеров
  return useMemo(() => selectedValue, [selectedValue]);
}

// Использование:
function UserProfile() {
  // Компонент ре-рендерится только при изменении user, не theme/settings
  const user = useAppSelector(state => state.user);
  return <div>{user.name}</div>;
}
```

<details>
<summary>💡 Подсказки</summary>

- Разделяйте большие контексты на маленькие
- Используйте отдельный Context для dispatch
- Создайте selector hook для выборки части состояния
- useMemo для мемоизации результата selector

</details>
---

## Задача 12.8: useAsync hook для асинхронных операций 🟡

**Контекст:** Универсальный hook для выполнения async функций.

**Задача:**
```typescript
interface UseAsyncResult<T> {
  execute: (...args: any[]) => Promise<void>;
  data: T | null;
  loading: boolean;
  error: Error | null;
}

function useAsync<T>(
  asyncFunction: (...args: any[]) => Promise<T>,
  immediate: boolean = false
): UseAsyncResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(immediate);
  const [error, setError] = useState<Error | null>(null);

  const execute = useCallback(async (...args: any[]) => {
    setLoading(true);
    setError(null);

    try {
      const result = await asyncFunction(...args);
      setData(result);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  }, [asyncFunction]);

  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return { execute, data, loading, error };
}

// Использование:
function UserProfile({ userId }: { userId: string }) {
  const fetchUser = async (id: string) => {
    const res = await fetch(`/api/users/${id}`);
    return res.json();
  };

  const { execute, data: user, loading, error } = useAsync(fetchUser);

  useEffect(() => {
    execute(userId);
  }, [userId, execute]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error!</div>;
  return <div>{user?.name}</div>;
}
```

<details>
<summary>💡 Подсказки</summary>

- useState для data, loading, error
- useCallback для execute функции
- Обрабатывайте try/catch/finally
- immediate параметр для автоматического вызова

</details>
---

## Задача 12.9: Form hook с валидацией 🟡

**Контекст:** Универсальный hook для управления формами.

**Задача:**
```typescript
interface UseFormOptions<T> {
  initialValues: T;
  validate?: (values: T) => Partial<Record<keyof T, string>>;
  onSubmit: (values: T) => void | Promise<void>;
}

function useForm<T extends Record<string, any>>({
  initialValues,
  validate,
  onSubmit
}: UseFormOptions<T>) {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (name: keyof T) => (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    setValues(prev => ({ ...prev, [name]: e.target.value }));

    // Валидировать поле при изменении
    if (validate) {
      const fieldErrors = validate({ ...values, [name]: e.target.value });
      setErrors(prev => ({ ...prev, [name]: fieldErrors[name] }));
    }
  };

  const handleBlur = (name: keyof T) => () => {
    setTouched(prev => ({ ...prev, [name]: true }));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);

      if (Object.keys(validationErrors).length > 0) {
        return;
      }
    }

    setIsSubmitting(true);
    try {
      await onSubmit(values);
    } finally {
      setIsSubmitting(false);
    }
  };

  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  };

  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    reset
  };
}

// Использование:
function LoginForm() {
  const form = useForm({
    initialValues: { email: '', password: '' },
    validate: (values) => {
      const errors: any = {};
      if (!values.email) errors.email = 'Required';
      if (!values.password) errors.password = 'Required';
      return errors;
    },
    onSubmit: async (values) => {
      await loginAPI(values);
    }
  });

  return (
    <form onSubmit={form.handleSubmit}>
      <input
        value={form.values.email}
        onChange={form.handleChange('email')}
        onBlur={form.handleBlur('email')}
      />
      {form.touched.email && form.errors.email && <span>{form.errors.email}</span>}
    </form>
  );
}
```

<details>
<summary>💡 Подсказки</summary>

- Храните values, errors, touched, isSubmitting
- handleChange обновляет значение и валидирует
- handleBlur отмечает поле как touched
- handleSubmit проверяет всю форму и вызывает onSubmit
- Показывайте ошибки только для touched полей

</details>
---

## Задача 12.10: Виртуализация списка (React) 🔴

**Контекст:** Компонент для виртуализированного списка с большим количеством элементов.

**Задача:**
```typescript
interface VirtualListProps<T> {
  items: T[];
  itemHeight: number;
  height: number; // высота контейнера
  renderItem: (item: T, index: number) => React.ReactNode;
  overscan?: number; // сколько элементов рендерить за границами видимой области
}

function VirtualList<T>({
  items,
  itemHeight,
  height,
  renderItem,
  overscan = 3
}: VirtualListProps<T>) {
  const [scrollTop, setScrollTop] = useState(0);

  const startIndex = Math.max(0, Math.floor(scrollTop / itemHeight) - overscan);
  const endIndex = Math.min(
    items.length - 1,
    Math.ceil((scrollTop + height) / itemHeight) + overscan
  );

  const visibleItems = items.slice(startIndex, endIndex + 1);

  const totalHeight = items.length * itemHeight;
  const offsetY = startIndex * itemHeight;

  const handleScroll = (e: React.UIEvent<HTMLDivElement>) => {
    setScrollTop(e.currentTarget.scrollTop);
  };

  return (
    <div
      style={{ height, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((item, i) => (
            <div key={startIndex + i} style={{ height: itemHeight }}>
              {renderItem(item, startIndex + i)}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// Использование:
function App() {
  const items = Array.from({ length: 10000 }, (_, i) => ({ id: i, name: `Item ${i}` }));

  return (
    <VirtualList
      items={items}
      itemHeight={50}
      height={400}
      renderItem={(item) => <div>{item.name}</div>}
    />
  );
}
```

<details>
<summary>💡 Подсказки</summary>

- Вычислите startIndex и endIndex на основе scrollTop
- Рендерите только видимые элементы + overscan
- Используйте transform: translateY для позиционирования
- Общая высота контейнера = items.length * itemHeight

</details>
---

## Задача 12.11: useMemoCompare для глубокого сравнения 🟡

**Контекст:** useMemo с кастомной функцией сравнения.

**Задача:**
```typescript
function useMemoCompare<T>(
  value: T,
  compare: (prev: T | undefined, next: T) => boolean
): T {
  const ref = useRef<T>();

  if (!ref.current || !compare(ref.current, value)) {
    ref.current = value;
  }

  return ref.current;
}

// Использование (shallow equality):
function useShallowMemo<T extends object>(obj: T): T {
  return useMemoCompare(obj, (prev, next) => {
    if (!prev) return false;

    const keys1 = Object.keys(prev);
    const keys2 = Object.keys(next);

    if (keys1.length !== keys2.length) return false;

    return keys1.every(key => prev[key as keyof T] === next[key as keyof T]);
  });
}

// Использование:
function Component({ filters }) {
  const memoizedFilters = useShallowMemo(filters);

  useEffect(() => {
    fetchData(memoizedFilters);
  }, [memoizedFilters]); // не пересоздается если поверхностно равен
}
```

<details>
<summary>💡 Подсказки</summary>

- useRef для хранения предыдущего значения
- Сравните с кастомной функцией
- Обновите ref только если не равны
- Верните ref.current

</details>
---

## Задача 12.12: Compound Components паттерн 🔴

**Контекст:** Создать компонент с подкомпонентами которые работают вместе.

**Задача:**
```typescript
// Создать Tabs компонент:
// <Tabs defaultValue="tab1">
//   <TabsList>
//     <TabsTrigger value="tab1">Tab 1</TabsTrigger>
//     <TabsTrigger value="tab2">Tab 2</TabsTrigger>
//   </TabsList>
//   <TabsContent value="tab1">Content 1</TabsContent>
//   <TabsContent value="tab2">Content 2</TabsContent>
// </Tabs>

interface TabsContextValue {
  activeTab: string;
  setActiveTab: (value: string) => void;
}

const TabsContext = createContext<TabsContextValue | undefined>(undefined);

function useTabs() {
  const context = useContext(TabsContext);
  if (!context) {
    throw new Error('Tabs components must be used within Tabs');
  }
  return context;
}

interface TabsProps {
  defaultValue: string;
  children: React.ReactNode;
}

function Tabs({ defaultValue, children }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultValue);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
}

function TabsList({ children }: { children: React.ReactNode }) {
  return <div role="tablist">{children}</div>;
}

function TabsTrigger({ value, children }: { value: string; children: React.ReactNode }) {
  const { activeTab, setActiveTab } = useTabs();

  return (
    <button
      role="tab"
      aria-selected={activeTab === value}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

function TabsContent({ value, children }: { value: string; children: React.ReactNode }) {
  const { activeTab } = useTabs();

  if (activeTab !== value) return null;

  return <div role="tabpanel">{children}</div>;
}

// Экспорт как одно целое
Tabs.List = TabsList;
Tabs.Trigger = TabsTrigger;
Tabs.Content = TabsContent;
```

<details>
<summary>💡 Подсказки</summary>

- Используйте Context для связи между компонентами
- Главный компонент управляет состоянием
- Подкомпоненты используют Context через hook
- Прикрепите подкомпоненты к главному: Tabs.List = TabsList
- Валидируйте что компоненты используются внутри родителя

</details>