# TypeScript типы

## Задача 7.1: Типизация API ответов 🟢

**Контекст:** Строго типизировать ответы от API.

**Задача:**
```typescript
// Создать типы для API ответов с возможными состояниями

type ApiResponse<T> =
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

// Использование:
async function fetchUser(id: string): Promise<ApiResponse<User>> {
  // TypeScript должен заставить обработать все случаи
}

function UserProfile({ response }: { response: ApiResponse<User> }) {
  if (response.status === 'loading') {
    return <Loader />;
  }
  if (response.status === 'error') {
    return <Error message={response.error} />;
  }
  // TypeScript знает что здесь response.data существует
  return <div>{response.data.name}</div>;
}
```

<details>
<summary>💡 Подсказки</summary>

- Используйте discriminated unions
- Status field - discriminator
- TypeScript автоматически narrowing типа в if блоках

</details>
---

## Задача 7.2: Generic функция для работы с массивами 🟡

**Контекст:** Создать type-safe функцию group by.

**Задача:**
```typescript
function groupBy<T, K extends keyof T>(
  array: T[],
  key: K
): Record<string, T[]> {
  // Группировать массив по ключу
  // TypeScript должен проверять что key существует в T
}

// Использование:
const users = [
  { id: 1, name: 'John', role: 'admin' },
  { id: 2, name: 'Jane', role: 'user' },
];

const byRole = groupBy(users, 'role'); // ✓
const byInvalid = groupBy(users, 'invalid'); // ✗ TS error
```

<details>
<summary>💡 Подсказки</summary>

- Используйте generic параметры
- K extends keyof T гарантирует что key существует
- Тип возвращаемого значения: Record<string, T[]>

</details>
---

## Задача 7.3: Utility type для форм 🟡

**Контекст:** Создать типы для значений формы и ошибок валидации.

**Задача:**
```typescript
// Из типа модели создать тип для формы

type User = {
  id: number;
  name: string;
  email: string;
  age: number;
};

// Создать utility type:
type FormState<T> = {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
};

// Использование:
const userForm: FormState<User> = {
  values: { id: 1, name: '', email: '', age: 0 },
  errors: { email: 'Invalid email' },
  touched: { email: true }
};
```

<details>
<summary>💡 Подсказки</summary>

- Используйте Partial для опциональных полей
- Record<keyof T, V> для объекта со всеми ключами T
- Комбинируйте встроенные utility types

</details>
---

## Задача 7.4: Строгая типизация event handlers 🟡

**Контекст:** Типизировать обработчики событий для разных элементов.

**Задача:**
```typescript
// Создать type-safe обработчики событий

type EventHandler<E extends HTMLElement, T extends Event> = (
  event: T & { currentTarget: E }
) => void;

// Использование:
const handleClick: EventHandler<HTMLButtonElement, MouseEvent> = (e) => {
  e.currentTarget.disabled = true; // ✓ TypeScript знает что это button
  e.currentTarget.value; // ✗ Error: button не имеет value
};

const handleInput: EventHandler<HTMLInputElement, InputEvent> = (e) => {
  console.log(e.currentTarget.value); // ✓
};
```

<details>
<summary>💡 Подсказки</summary>

- Используйте type intersection (&)
- Переопределите currentTarget для конкретного типа
- Для React: используйте встроенные типы событий

</details>
---

## Задача 7.5: Вывод типов из конфигурации 🟡

**Контекст:** Автоматически выводить типы из объекта конфигурации.

**Задача:**
```typescript
// Создать type-safe конфигурацию роутов

const routes = {
  home: '/',
  user: '/users/:id',
  post: '/posts/:postId',
  settings: '/settings'
} as const;

// Создать функцию для генерации URL с type-safe параметрами

type RouteParams<T extends string> =
  T extends `${infer _Start}:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof RouteParams<`/${Rest}`>]: string }
    : T extends `${infer _Start}:${infer Param}`
    ? { [K in Param]: string }
    : never;

function generateUrl<K extends keyof typeof routes>(
  route: K,
  ...params: RouteParams<typeof routes[K]> extends never
    ? []
    : [RouteParams<typeof routes[K]>]
): string {
  // Генерировать URL с подстановкой параметров
}

// Использование:
generateUrl('home'); // ✓
generateUrl('user', { id: '123' }); // ✓
generateUrl('user'); // ✗ Error: missing id
generateUrl('user', { wrong: '123' }); // ✗ Error: wrong param
```

<details>
<summary>💡 Подсказки</summary>

- Используйте template literal types
- Recursive types для парсинга параметров
- Conditional types для проверки наличия параметров

</details>
---

## Задача 7.6: Типизация Redux-подобного store 🔴

**Контекст:** Создать type-safe store с actions и reducers.

**Задача:**
```typescript
// Actions
type Action =
  | { type: 'SET_USER'; payload: User }
  | { type: 'INCREMENT_COUNTER' }
  | { type: 'SET_FILTER'; payload: string };

// State
type State = {
  user: User | null;
  counter: number;
  filter: string;
};

// Создать типизированный reducer
type Reducer<S, A> = (state: S, action: A) => S;

// ActionCreator должен выводить правильный тип action
type ActionCreator<T extends Action['type']> =
  Extract<Action, { type: T }> extends { payload: infer P }
    ? (payload: P) => Extract<Action, { type: T }>
    : () => Extract<Action, { type: T }>;

// Использование:
const setUser: ActionCreator<'SET_USER'> = (payload) => ({
  type: 'SET_USER',
  payload
});

const increment: ActionCreator<'INCREMENT_COUNTER'> = () => ({
  type: 'INCREMENT_COUNTER'
});
```

<details>
<summary>💡 Подсказки</summary>

- Используйте discriminated unions для actions
- Extract для извлечения конкретного action по type
- Conditional types для проверки наличия payload
- infer для извлечения типа payload

</details>
---

## Задача 7.7: Типы для Builder Pattern 🟡

**Контекст:** Создать type-safe builder для конструирования объектов.

**Задача:**
```typescript
class QueryBuilder<T> {
  private filters: Partial<T> = {};
  private sortField?: keyof T;
  private limitValue?: number;

  where<K extends keyof T>(field: K, value: T[K]): this {
    this.filters[field] = value;
    return this;
  }

  sort(field: keyof T): this {
    this.sortField = field;
    return this;
  }

  limit(n: number): this {
    this.limitValue = n;
    return this;
  }

  build() {
    return {
      filters: this.filters,
      sort: this.sortField,
      limit: this.limitValue
    };
  }
}

// Использование:
type User = { name: string; age: number; email: string };

const query = new QueryBuilder<User>()
  .where('name', 'John') // ✓
  .where('age', 25) // ✓
  .where('invalid', 'value') // ✗ Error
  .sort('email') // ✓
  .limit(10)
  .build();
```

<details>
<summary>💡 Подсказки</summary>

- Используйте generics для типа объекта
- where принимает field: K и value: T[K] (связанные типы)
- Возвращайте this для chaining
- keyof T для ограничения возможных полей

</details>
---

## Задача 7.8: Mapped Types для создания производных типов 🟡

**Контекст:** Создать типы на основе существующих с модификациями.

**Задача:**
```typescript
type User = {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
};

// Создать тип где все поля опциональны и nullable
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

// Создать тип только с полями определенного типа
type PropertiesOfType<T, V> = {
  [K in keyof T as T[K] extends V ? K : never]: T[K];
};

// Использование:
type NullableUser = Nullable<User>;
// { id: number | null; name: string | null; ... }

type StringProperties = PropertiesOfType<User, string>;
// { name: string; email: string; }
```

<details>
<summary>💡 Подсказки</summary>

- Используйте mapped types: [K in keyof T]
- Key remapping с as для фильтрации
- Conditional types для проверки типа значения

</details>
---

## Задача 7.9: Recursive types для вложенных структур 🔴

**Контекст:** Типизировать глубоко вложенные объекты.

**Задача:**
```typescript
// Создать тип для deep partial (все поля опциональны рекурсивно)

type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object
    ? DeepPartial<T[K]>
    : T[K];
};

// Использование:
type Config = {
  server: {
    host: string;
    port: number;
    ssl: {
      enabled: boolean;
      cert: string;
    };
  };
  database: {
    host: string;
    port: number;
  };
};

type PartialConfig = DeepPartial<Config>;
// Все поля опциональны на всех уровнях вложенности
```

<details>
<summary>💡 Подсказки</summary>

- Используйте рекурсию в типах
- Проверяйте extends object для определения вложенных объектов
- Добавьте ? для опциональности
- Обрабатывайте массивы отдельно если нужно

</details>
---

## Задача 7.10: Типизация функции compose 🔴

**Контекст:** Создать type-safe композицию функций.

**Задача:**
```typescript
// compose(f, g, h)(x) = f(g(h(x)))

function compose<A, B>(f: (a: A) => B): (a: A) => B;
function compose<A, B, C>(
  f: (b: B) => C,
  g: (a: A) => B
): (a: A) => C;
function compose<A, B, C, D>(
  f: (c: C) => D,
  g: (b: B) => C,
  h: (a: A) => B
): (a: A) => D;
// ... и так далее

function compose(...fns: Function[]): Function {
  return (x: any) => fns.reduceRight((acc, fn) => fn(acc), x);
}

// Использование:
const add = (x: number) => x + 1;
const multiply = (x: number) => x * 2;
const toString = (x: number) => x.toString();

const composed = compose(toString, multiply, add);
const result = composed(5); // type: string, value: "12"
```

<details>
<summary>💡 Подсказки</summary>

- Используйте function overloading
- Каждая перегрузка для разного количества функций
- Типы должны "проходить" через цепочку функций
- Можно ограничиться 3-4 перегрузками

</details>
---

## Задача 7.11: Типы для валидации схемы 🔴

**Контекст:** Создать type-safe валидатор схемы (как Zod).

**Задача:**
```typescript
// Базовые валидаторы
const string = () => ({ type: 'string' as const });
const number = () => ({ type: 'number' as const });
const boolean = () => ({ type: 'boolean' as const });

// Объектный валидатор
const object = <T extends Record<string, any>>(schema: T) => ({
  type: 'object' as const,
  schema
});

// Вывести TypeScript тип из схемы
type Infer<T> =
  T extends { type: 'string' } ? string :
  T extends { type: 'number' } ? number :
  T extends { type: 'boolean' } ? boolean :
  T extends { type: 'object'; schema: infer S }
    ? { [K in keyof S]: Infer<S[K]> }
    : never;

// Использование:
const userSchema = object({
  name: string(),
  age: number(),
  verified: boolean()
});

type User = Infer<typeof userSchema>;
// { name: string; age: number; verified: boolean; }
```

<details>
<summary>💡 Подсказки</summary>

- Используйте const assertions для literal types
- Conditional types для разных типов валидаторов
- Recursive types для вложенных объектов
- infer для извлечения типа схемы

</details>
---

## Задача 7.12: Типобезопасный путь к свойству 🟡

**Контекст:** Создать функцию get с автодополнением пути.

**Задача:**
```typescript
type PathImpl<T, K extends keyof T> =
  K extends string
  ? T[K] extends Record<string, any>
    ? K | `${K}.${PathImpl<T[K], keyof T[K]>}`
    : K
  : never;

type Path<T> = PathImpl<T, keyof T> | keyof T;

type PathValue<T, P extends Path<T>> =
  P extends `${infer K}.${infer Rest}`
    ? K extends keyof T
      ? Rest extends Path<T[K]>
        ? PathValue<T[K], Rest>
        : never
      : never
    : P extends keyof T
    ? T[P]
    : never;

function get<T, P extends Path<T>>(obj: T, path: P): PathValue<T, P> {
  // Реализация
}

// Использование:
type User = {
  profile: {
    address: {
      city: string;
    };
  };
};

const user: User = { profile: { address: { city: 'NY' } } };
const city = get(user, 'profile.address.city'); // type: string
const invalid = get(user, 'profile.invalid'); // ✗ Error
```

<details>
<summary>💡 Подсказки</summary>

- Template literal types для пути
- Recursive types для обхода вложенности
- Conditional types для парсинга пути

</details>