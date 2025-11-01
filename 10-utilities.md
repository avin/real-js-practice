# Утилиты и хелперы

## Задача 10.1: Deep clone объекта 🟡

**Контекст:** Создать полную копию объекта со всеми вложенными уровнями.

**Задача:**
```typescript
function deepClone<T>(obj: T): T {
  // Клонировать объект рекурсивно
  // Обработать:
  // - Примитивы
  // - Массивы
  // - Объекты
  // - Date
  // - null/undefined
  // - Циклические ссылки (опционально)
}

// Использование:
const original = {
  name: 'John',
  address: { city: 'NY', street: '5th' },
  hobbies: ['reading', 'coding']
};
const cloned = deepClone(original);
cloned.address.city = 'LA'; // не должно изменить original
```

<details>
<summary>💡 Подсказки</summary>

- Проверяйте тип: typeof, Array.isArray, instanceof Date
- Рекурсивно клонируйте вложенные объекты/массивы
- Для циклических ссылок используйте WeakMap для отслеживания
- Альтернатива (простая): JSON.parse(JSON.stringify(obj))
- Альтернатива (современная): structuredClone(obj)

</details>


## Задача 10.2: Deep merge объектов 🟡

**Контекст:** Объединить два объекта с глубоким слиянием вложенных свойств.

**Задача:**
```typescript
function deepMerge<T extends object>(target: T, ...sources: Partial<T>[]): T {
  // Слить источники в target
  // Вложенные объекты тоже должны сливаться
}

// Использование:
const obj1 = { a: 1, b: { c: 2, d: 3 } };
const obj2 = { b: { d: 4, e: 5 }, f: 6 };
const merged = deepMerge(obj1, obj2);
// { a: 1, b: { c: 2, d: 4, e: 5 }, f: 6 }
```

<details>
<summary>💡 Подсказки</summary>

- Проверяйте что оба значения - объекты
- Если оба объекты - рекурсивно мержить
- Иначе - значение из source перезаписывает target
- Обрабатывайте массивы (заменять или мержить?)

</details>


## Задача 10.3: Работа с cookies 🟢

**Контекст:** Утилиты для чтения, записи и удаления cookies.

**Задача:**
```typescript
const cookies = {
  set(name: string, value: string, days?: number, options?: {
    path?: string;
    domain?: string;
    secure?: boolean;
    sameSite?: 'Strict' | 'Lax' | 'None';
  }): void {
    // Установить cookie
  },

  get(name: string): string | null {
    // Получить значение cookie
  },

  remove(name: string, options?: { path?: string; domain?: string }): void {
    // Удалить cookie
  },

  getAll(): Record<string, string> {
    // Получить все cookies как объект
  }
};

// Использование:
cookies.set('token', 'abc123', 7, { secure: true, sameSite: 'Strict' });
const token = cookies.get('token');
```

<details>
<summary>💡 Подсказки</summary>

- document.cookie для чтения/записи
- set: собрать строку с expires, path, domain и т.д.
- get: парсить document.cookie (split по ';')
- remove: установить expires в прошлое
- decodeURIComponent/encodeURIComponent для значений

</details>


## Задача 10.4: Генерация уникальных ID 🟢

**Контекст:** Генерировать уникальные идентификаторы для элементов.

**Задача:**
```typescript
function generateId(prefix: string = 'id'): string {
  // Генерировать уникальный ID
  // Например: 'id-1234567890-abc'
}

function uuid(): string {
  // Генерировать UUID v4
  // Формат: xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx
}

function nanoid(size: number = 21): string {
  // Генерировать короткий уникальный ID
  // Безопасный для URL
}
```

<details>
<summary>💡 Подсказки</summary>

- generateId: prefix + Date.now() + Math.random()
- uuid: используйте crypto.randomUUID() (современные браузеры)
- Или генерируйте вручную по спецификации UUID v4
- nanoid: crypto.getRandomValues() + безопасный алфавит

</details>


## Задача 10.5: Проверка типов данных 🟢

**Контекст:** Надежные функции для проверки типов (лучше чем typeof).

**Задача:**
```typescript
const is = {
  array: (value: unknown): value is Array<any> => {},
  object: (value: unknown): value is object => {},
  string: (value: unknown): value is string => {},
  number: (value: unknown): value is number => {},
  boolean: (value: unknown): value is boolean => {},
  function: (value: unknown): value is Function => {},
  null: (value: unknown): value is null => {},
  undefined: (value: unknown): value is undefined => {},
  date: (value: unknown): value is Date => {},
  regexp: (value: unknown): value is RegExp => {},
  error: (value: unknown): value is Error => {},
  plainObject: (value: unknown): value is Record<string, any> => {
    // Проверить что это plain object (не Array, Date и т.д.)
  }
};

// Использование:
if (is.array(value)) {
  value.map(...); // TypeScript знает что это array
}
```

<details>
<summary>💡 Подсказки</summary>

- Array.isArray() для массивов
- Object.prototype.toString.call(value) для точного типа
- '[object Array]', '[object Date]' и т.д.
- plainObject: проверить что prototype === Object.prototype
- Используйте type predicates (value is Type)

</details>


## Задача 10.6: Retry с экспоненциальной задержкой 🟡

**Контекст:** Повторять операцию с увеличивающейся задержкой при неудаче.

**Задача:**
```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  options: {
    maxRetries?: number;
    initialDelay?: number;
    maxDelay?: number;
    factor?: number; // множитель для экспоненциального роста
    onRetry?: (error: Error, attempt: number) => void;
  } = {}
): Promise<T> {
  // Попытки: 0, 100ms, 200ms, 400ms, 800ms, 1600ms...
  // Ограничить maxDelay
}

// Использование:
const data = await retryWithBackoff(
  () => fetch('/api/data').then(r => r.json()),
  {
    maxRetries: 5,
    initialDelay: 100,
    maxDelay: 5000,
    factor: 2,
    onRetry: (err, attempt) => console.log(`Retry ${attempt}: ${err.message}`)
  }
);
```

<details>
<summary>💡 Подсказки</summary>

- delay = initialDelay * Math.pow(factor, attempt)
- Ограничить: Math.min(delay, maxDelay)
- await sleep(delay) между попытками
- Опционально добавить jitter (случайное отклонение)

</details>


## Задача 10.7: Function composition 🟡

**Контекст:** Композиция функций справа налево (как в функциональном программировании).

**Задача:**
```typescript
function compose<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
  // compose(f, g, h)(x) = f(g(h(x)))
}

function pipe<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
  // pipe(f, g, h)(x) = h(g(f(x)))
}

// Использование:
const add = (x: number) => x + 1;
const multiply = (x: number) => x * 2;
const square = (x: number) => x * x;

const composed = compose(square, multiply, add);
composed(5); // square(multiply(add(5))) = square(multiply(6)) = square(12) = 144

const piped = pipe(add, multiply, square);
piped(5); // square(multiply(add(5))) = 144
```

<details>
<summary>💡 Подсказки</summary>

- compose: reduceRight для применения функций справа налево
- pipe: reduce для применения слева направо
- Начальное значение - аргумент функции

</details>


## Задача 10.8: Промисификация callback функций 🟡

**Контекст:** Превратить функцию с callback в функцию возвращающую Promise.

**Задача:**
```typescript
function promisify<T>(
  fn: (...args: any[]) => void
): (...args: any[]) => Promise<T> {
  return (...args) => {
    return new Promise((resolve, reject) => {
      fn(...args, (err: Error | null, result: T) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  };
}

// Использование:
function readFile(path: string, callback: (err: Error | null, data: string) => void) {
  // Legacy callback-based function
}

const readFileAsync = promisify<string>(readFile);
const data = await readFileAsync('file.txt');
```

<details>
<summary>💡 Подсказки</summary>

- Верните функцию которая возвращает Promise
- Добавьте callback к аргументам
- В callback проверьте первый аргумент (error)
- Если ошибка - reject, иначе resolve

</details>


## Задача 10.9: Частичное применение (partial) 🟢

**Контекст:** Создать функцию с предзаполненными аргументами.

**Задача:**
```typescript
function partial<T extends any[], U extends any[], R>(
  fn: (...args: [...T, ...U]) => R,
  ...partialArgs: T
): (...args: U) => R {
  return (...restArgs) => fn(...partialArgs, ...restArgs);
}

// Использование:
function greet(greeting: string, name: string) {
  return `${greeting}, ${name}!`;
}

const sayHello = partial(greet, 'Hello');
sayHello('John'); // 'Hello, John!'
sayHello('Jane'); // 'Hello, Jane!'
```

<details>
<summary>💡 Подсказки</summary>

- Верните новую функцию
- Объедините partialArgs и args при вызове
- Используйте spread operator

</details>


## Задача 10.10: Безопасный доступ к вложенным свойствам 🟡

**Контекст:** Получить значение по пути без ошибок если путь не существует.

**Задача:**
```typescript
function get<T = any>(
  obj: any,
  path: string | string[],
  defaultValue?: T
): T | undefined {
  // get(obj, 'a.b.c') или get(obj, ['a', 'b', 'c'])
  // Если путь не существует - вернуть defaultValue
}

function set(obj: any, path: string | string[], value: any): void {
  // set(obj, 'a.b.c', value)
  // Создать промежуточные объекты если не существуют
}

// Использование:
const obj = { a: { b: { c: 42 } } };
get(obj, 'a.b.c'); // 42
get(obj, 'a.x.y', 'default'); // 'default'

set(obj, 'a.b.d', 100); // obj.a.b.d = 100
set(obj, 'x.y.z', 200); // создаст { x: { y: { z: 200 } } }
```

<details>
<summary>💡 Подсказки</summary>

- Разделите path по '.' если строка
- Используйте reduce для обхода пути
- Проверяйте на каждом шаге что значение не null/undefined
- set: создавайте объекты на промежуточных уровнях

</details>


## Задача 10.11: Функция sleep 🟢

**Контекст:** Асинхронная задержка.

**Задача:**
```typescript
function sleep(ms: number): Promise<void> {
  // Промис который резолвится через ms миллисекунд
}

// Использование:
console.log('Start');
await sleep(1000);
console.log('After 1 second');
```

<details>
<summary>💡 Подсказки</summary>

- Верните new Promise
- Используйте setTimeout для задержки
- resolve() в callback setTimeout

</details>


## Задача 10.12: Chunk array 🟢

**Контекст:** Разделить массив на части (chunks) определенного размера.

**Задача:**
```typescript
function chunk<T>(array: T[], size: number): T[][] {
  // [1, 2, 3, 4, 5], 2 → [[1, 2], [3, 4], [5]]
}

// Использование:
chunk([1, 2, 3, 4, 5, 6, 7], 3); // [[1, 2, 3], [4, 5, 6], [7]]
```

<details>
<summary>💡 Подсказки</summary>

- Используйте цикл с шагом size
- slice(i, i + size) для каждого chunk
- Или reduce для накопления chunks

</details>


## Задача 10.13: Flatten массива 🟢

**Контекст:** Сделать вложенный массив плоским.

**Задача:**
```typescript
function flatten<T>(array: any[], depth: number = 1): T[] {
  // [1, [2, [3, 4]], 5] → [1, 2, [3, 4], 5] (depth = 1)
  // [1, [2, [3, 4]], 5] → [1, 2, 3, 4, 5] (depth = Infinity)
}

// Использование:
flatten([1, [2, [3, [4]]]], 2); // [1, 2, 3, [4]]
```

<details>
<summary>💡 Подсказки</summary>

- Используйте рекурсию
- Проверяйте Array.isArray(item) && depth > 0
- Уменьшайте depth при каждой рекурсии
- Альтернатива: array.flat(depth) (встроенный метод)

</details>


## Задача 10.14: Pick и omit для объектов 🟢

**Контекст:** Выбрать или исключить поля из объекта.

**Задача:**
```typescript
function pick<T extends object, K extends keyof T>(
  obj: T,
  keys: K[]
): Pick<T, K> {
  // Вернуть новый объект только с указанными ключами
}

function omit<T extends object, K extends keyof T>(
  obj: T,
  keys: K[]
): Omit<T, K> {
  // Вернуть новый объект без указанных ключей
}

// Использование:
const user = { id: 1, name: 'John', email: 'john@example.com', age: 30 };
pick(user, ['id', 'name']); // { id: 1, name: 'John' }
omit(user, ['email', 'age']); // { id: 1, name: 'John' }
```

<details>
<summary>💡 Подсказки</summary>

- pick: создать объект только с keys
- reduce или Object.entries + filter
- omit: создать объект без keys
- Или pick с инвертированными ключами

</details>


## Задача 10.15: Debounce для React (custom hook) 🟡

**Контекст:** Hook для debounced значения.

**Задача:**
```typescript
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    // Установить таймер для обновления debouncedValue
    // Очистить таймер при изменении value или размонтировании

    return () => {
      // cleanup
    };
  }, [value, delay]);

  return debouncedValue;
}

// Использование:
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);

  useEffect(() => {
    if (debouncedSearchTerm) {
      // Выполнить поиск
    }
  }, [debouncedSearchTerm]);
}
```

<details>
<summary>💡 Подсказки</summary>

- Используйте useState для debouncedValue
- useEffect зависит от value и delay
- setTimeout для обновления после delay
- Верните cleanup функцию для clearTimeout

</details>