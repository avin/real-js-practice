# Работа с API и HTTP

## Задача 11.1: Типизированный API клиент 🟡

**Контекст:** Создать type-safe wrapper над fetch с автоматической обработкой ошибок.

**Задача:**
```typescript
class ApiClient {
  constructor(private baseURL: string) {}

  async get<T>(endpoint: string, options?: RequestInit): Promise<T> {
    return this.request<T>(endpoint, { ...options, method: 'GET' });
  }

  async post<T>(endpoint: string, data?: any, options?: RequestInit): Promise<T> {
    return this.request<T>(endpoint, {
      ...options,
      method: 'POST',
      body: JSON.stringify(data),
      headers: { 'Content-Type': 'application/json', ...options?.headers }
    });
  }

  async put<T>(endpoint: string, data?: any, options?: RequestInit): Promise<T> {}
  async delete<T>(endpoint: string, options?: RequestInit): Promise<T> {}

  private async request<T>(endpoint: string, options: RequestInit): Promise<T> {
    const url = `${this.baseURL}${endpoint}`;
    const response = await fetch(url, options);

    if (!response.ok) {
      throw new ApiError(response.status, await response.text());
    }

    return response.json();
  }
}

class ApiError extends Error {
  constructor(public status: number, message: string) {
    super(message);
  }
}

// Использование:
const api = new ApiClient('https://api.example.com');
const user = await api.get<User>('/users/1');
```

<details>
<summary>💡 Подсказки</summary>

- Создайте общий метод request
- Проверяйте response.ok
- Автоматически парсите JSON
- Создайте custom Error класс с статус-кодом
- Добавьте типизацию для response

</details>


## Задача 11.2: Request/Response interceptors 🟡

**Контекст:** Перехватывать запросы для добавления токенов и ответы для обработки ошибок.

**Задача:**
```typescript
type RequestInterceptor = (config: RequestInit) => RequestInit | Promise<RequestInit>;
type ResponseInterceptor = (response: Response) => Response | Promise<Response>;

class ApiClientWithInterceptors {
  private requestInterceptors: RequestInterceptor[] = [];
  private responseInterceptors: ResponseInterceptor[] = [];

  addRequestInterceptor(interceptor: RequestInterceptor): void {
    this.requestInterceptors.push(interceptor);
  }

  addResponseInterceptor(interceptor: ResponseInterceptor): void {
    this.responseInterceptors.push(interceptor);
  }

  private async applyRequestInterceptors(config: RequestInit): Promise<RequestInit> {
    // Последовательно применить все request interceptors
  }

  private async applyResponseInterceptors(response: Response): Promise<Response> {
    // Последовательно применить все response interceptors
  }

  private async request<T>(endpoint: string, options: RequestInit): Promise<T> {
    const config = await this.applyRequestInterceptors(options);
    let response = await fetch(`${this.baseURL}${endpoint}`, config);
    response = await this.applyResponseInterceptors(response);
    return response.json();
  }
}

// Использование:
const api = new ApiClientWithInterceptors('https://api.example.com');

// Добавить токен к каждому запросу
api.addRequestInterceptor((config) => ({
  ...config,
  headers: {
    ...config.headers,
    'Authorization': `Bearer ${getToken()}`
  }
}));

// Логировать все ответы
api.addResponseInterceptor((response) => {
  console.log('Response:', response.status);
  return response;
});
```

<details>
<summary>💡 Подсказки</summary>

- Храните массивы interceptor функций
- Применяйте последовательно через reduce или цикл
- Request interceptors изменяют config
- Response interceptors могут проверить статус и бросить ошибку

</details>


## Задача 11.3: Автоматический refresh токена 🔴

**Контекст:** При получении 401 автоматически обновить access token и повторить запрос.

**Задача:**
```typescript
class AuthApiClient {
  private refreshing: Promise<string> | null = null;

  async request<T>(endpoint: string, options: RequestInit): Promise<T> {
    let response = await fetch(`${this.baseURL}${endpoint}`, this.addAuth(options));

    if (response.status === 401) {
      // Обновить токен
      const newToken = await this.refreshToken();

      // Повторить запрос с новым токеном
      response = await fetch(`${this.baseURL}${endpoint}`, this.addAuth(options, newToken));
    }

    if (!response.ok) throw new Error('Request failed');
    return response.json();
  }

  private async refreshToken(): Promise<string> {
    // Если уже идет refresh - дождаться его
    if (this.refreshing) {
      return this.refreshing;
    }

    this.refreshing = this.doRefresh();
    try {
      return await this.refreshing;
    } finally {
      this.refreshing = null;
    }
  }

  private async doRefresh(): Promise<string> {
    const refreshToken = this.getRefreshToken();
    const response = await fetch(`${this.baseURL}/auth/refresh`, {
      method: 'POST',
      body: JSON.stringify({ refreshToken })
    });

    const { accessToken } = await response.json();
    this.setAccessToken(accessToken);
    return accessToken;
  }

  private addAuth(options: RequestInit, token?: string): RequestInit {
    // Добавить Authorization header
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- При 401 вызывайте refreshToken()
- Используйте флаг/промис чтобы не делать множественные refresh
- После refresh повторите оригинальный запрос
- Сохраните новый токен в localStorage

</details>


## Задача 11.4: Query parameters builder 🟢

**Контекст:** Удобно строить URL с query параметрами.

**Задача:**
```typescript
function buildQueryString(params: Record<string, any>): string {
  // { name: 'John', age: 30, active: true } → '?name=John&age=30&active=true'
  // Обработать:
  // - Массивы: { tags: ['js', 'ts'] } → 'tags=js&tags=ts'
  // - null/undefined - пропустить
  // - URL encoding
}

class QueryBuilder {
  private params: Record<string, any> = {};

  set(key: string, value: any): this {
    this.params[key] = value;
    return this;
  }

  remove(key: string): this {
    delete this.params[key];
    return this;
  }

  build(): string {
    return buildQueryString(this.params);
  }

  toString(): string {
    return this.build();
  }
}

// Использование:
const query = new QueryBuilder()
  .set('search', 'hello world')
  .set('page', 1)
  .set('tags', ['js', 'ts'])
  .build(); // '?search=hello%20world&page=1&tags=js&tags=ts'
```

<details>
<summary>💡 Подсказки</summary>

- Используйте URLSearchParams (встроенный API)
- Или вручную: Object.entries + map + join
- encodeURIComponent для значений
- Массивы: добавить каждый элемент отдельно
- Пропустить null/undefined значения

</details>


## Задача 11.5: Загрузка файла с прогрессом 🟡

**Контекст:** Загрузить файл на сервер с отслеживанием прогресса.

**Задача:**
```typescript
function uploadFile(
  file: File,
  url: string,
  onProgress: (progress: number) => void
): Promise<any> {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();

    xhr.upload.addEventListener('progress', (e) => {
      if (e.lengthComputable) {
        const progress = (e.loaded / e.total) * 100;
        onProgress(progress);
      }
    });

    xhr.addEventListener('load', () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`Upload failed: ${xhr.status}`));
      }
    });

    xhr.addEventListener('error', () => reject(new Error('Upload failed')));

    xhr.open('POST', url);

    const formData = new FormData();
    formData.append('file', file);

    xhr.send(formData);
  });
}

// Использование:
const file = document.querySelector('input[type="file"]').files[0];
await uploadFile(file, '/api/upload', (progress) => {
  console.log(`Upload: ${progress}%`);
});
```

<details>
<summary>💡 Подсказки</summary>

- Используйте XMLHttpRequest (fetch не поддерживает upload progress)
- Слушайте xhr.upload.progress event
- e.loaded / e.total для процента
- Используйте FormData для отправки файла

</details>


## Задача 11.6: AbortController для отмены запросов 🟡

**Контекст:** Отменять предыдущие запросы при новых поисковых запросах.

**Задача:**
```typescript
class SearchService {
  private currentController: AbortController | null = null;

  async search(query: string): Promise<SearchResult[]> {
    // Отменить предыдущий запрос
    if (this.currentController) {
      this.currentController.abort();
    }

    // Создать новый controller
    this.currentController = new AbortController();

    try {
      const response = await fetch(`/api/search?q=${query}`, {
        signal: this.currentController.signal
      });

      return await response.json();
    } catch (error) {
      if (error.name === 'AbortError') {
        // Запрос был отменен, это нормально
        return [];
      }
      throw error;
    }
  }

  cancel(): void {
    this.currentController?.abort();
  }
}

// Использование:
const searchService = new SearchService();

// Быстрое переключение поиска
await searchService.search('abc'); // будет отменен
await searchService.search('abcd'); // будет отменен
await searchService.search('abcde'); // выполнится
```

<details>
<summary>💡 Подсказки</summary>

- Создавайте новый AbortController для каждого запроса
- Отменяйте предыдущий перед новым запросом
- Передавайте signal в fetch options
- Обрабатывайте AbortError отдельно (не показывать как ошибку)

</details>


## Задача 11.7: Пагинация с курсором 🟡

**Контекст:** Реализовать загрузку данных с cursor-based пагинацией.

**Задача:**
```typescript
interface CursorPage<T> {
  data: T[];
  nextCursor: string | null;
  hasMore: boolean;
}

class CursorPagination<T> {
  private items: T[] = [];
  private nextCursor: string | null = null;
  private loading = false;

  async loadMore(endpoint: string): Promise<void> {
    if (this.loading || !this.hasMore()) return;

    this.loading = true;
    try {
      const url = this.nextCursor
        ? `${endpoint}?cursor=${this.nextCursor}`
        : endpoint;

      const response = await fetch(url);
      const page: CursorPage<T> = await response.json();

      this.items.push(...page.data);
      this.nextCursor = page.nextCursor;
    } finally {
      this.loading = false;
    }
  }

  getItems(): T[] {
    return this.items;
  }

  hasMore(): boolean {
    return this.nextCursor !== null;
  }

  reset(): void {
    this.items = [];
    this.nextCursor = null;
  }
}

// Использование:
const pagination = new CursorPagination<Post>();
await pagination.loadMore('/api/posts');
// Scroll to end
await pagination.loadMore('/api/posts'); // загрузит следующую страницу
```

<details>
<summary>💡 Подсказки</summary>

- Храните nextCursor из последнего ответа
- Передавайте cursor в URL для следующего запроса
- hasMore = nextCursor !== null
- Предотвращайте двойную загрузку через флаг loading

</details>


## Задача 11.8: Batch API requests 🟡

**Контекст:** Объединить множественные API запросы в один batch запрос.

**Задача:**
```typescript
class BatchApiClient {
  private queue: Array<{
    endpoint: string;
    resolve: (data: any) => void;
    reject: (error: Error) => void;
  }> = [];
  private timer: NodeJS.Timeout | null = null;
  private batchDelay = 50;

  async get<T>(endpoint: string): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push({ endpoint, resolve, reject });

      if (!this.timer) {
        this.timer = setTimeout(() => this.flush(), this.batchDelay);
      }
    });
  }

  private async flush(): Promise<void> {
    const batch = this.queue.splice(0);
    this.timer = null;

    if (batch.length === 0) return;

    try {
      // Отправить batch запрос
      const response = await fetch('/api/batch', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          requests: batch.map(item => ({ endpoint: item.endpoint }))
        })
      });

      const results = await response.json();

      // Распределить результаты по промисам
      batch.forEach((item, index) => {
        const result = results[index];
        if (result.error) {
          item.reject(new Error(result.error));
        } else {
          item.resolve(result.data);
        }
      });
    } catch (error) {
      batch.forEach(item => item.reject(error));
    }
  }
}

// Использование:
const api = new BatchApiClient();

// Эти запросы будут объединены в один
const [user, posts, comments] = await Promise.all([
  api.get('/users/1'),
  api.get('/posts'),
  api.get('/comments')
]);
```

<details>
<summary>💡 Подсказки</summary>

- Собирайте запросы в очередь
- Используйте setTimeout для задержки перед отправкой batch
- Храните resolve/reject для каждого запроса
- После batch ответа распределите результаты

</details>


## Задача 11.9: Retry с разными стратегиями 🟡

**Контекст:** Гибкая retry логика с разными стратегиями повтора.

**Задача:**
```typescript
type RetryStrategy = 'fixed' | 'exponential' | 'linear';

interface RetryOptions {
  maxAttempts: number;
  strategy: RetryStrategy;
  baseDelay: number;
  maxDelay?: number;
  shouldRetry?: (error: Error, attempt: number) => boolean;
}

async function fetchWithRetry<T>(
  url: string,
  options: RequestInit = {},
  retryOptions: RetryOptions
): Promise<T> {
  let lastError: Error;

  for (let attempt = 0; attempt < retryOptions.maxAttempts; attempt++) {
    try {
      const response = await fetch(url, options);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return await response.json();
    } catch (error) {
      lastError = error;

      // Проверить нужно ли повторять
      if (retryOptions.shouldRetry && !retryOptions.shouldRetry(error, attempt)) {
        throw error;
      }

      // Если это последняя попытка - бросить ошибку
      if (attempt === retryOptions.maxAttempts - 1) {
        throw error;
      }

      // Вычислить задержку
      const delay = calculateDelay(attempt, retryOptions);
      await sleep(delay);
    }
  }

  throw lastError!;
}

function calculateDelay(attempt: number, options: RetryOptions): number {
  let delay: number;

  switch (options.strategy) {
    case 'fixed':
      delay = options.baseDelay;
      break;
    case 'linear':
      delay = options.baseDelay * (attempt + 1);
      break;
    case 'exponential':
      delay = options.baseDelay * Math.pow(2, attempt);
      break;
  }

  // Ограничить maxDelay
  if (options.maxDelay) {
    delay = Math.min(delay, options.maxDelay);
  }

  return delay;
}
```

<details>
<summary>💡 Подсказки</summary>

- fixed: одинаковая задержка
- linear: увеличение на baseDelay каждый раз
- exponential: baseDelay * 2^attempt
- shouldRetry позволяет пропустить retry для определенных ошибок

</details>


## Задача 11.10: GraphQL клиент 🔴

**Контекст:** Простой типизированный GraphQL клиент.

**Задача:**
```typescript
interface GraphQLResponse<T> {
  data?: T;
  errors?: Array<{ message: string; path?: string[] }>;
}

class GraphQLClient {
  constructor(private endpoint: string) {}

  async query<T>(
    query: string,
    variables?: Record<string, any>
  ): Promise<T> {
    return this.request<T>(query, variables);
  }

  async mutate<T>(
    mutation: string,
    variables?: Record<string, any>
  ): Promise<T> {
    return this.request<T>(mutation, variables);
  }

  private async request<T>(
    query: string,
    variables?: Record<string, any>
  ): Promise<T> {
    const response = await fetch(this.endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ query, variables })
    });

    const result: GraphQLResponse<T> = await response.json();

    if (result.errors) {
      throw new GraphQLError(result.errors);
    }

    return result.data!;
  }
}

class GraphQLError extends Error {
  constructor(public errors: Array<{ message: string; path?: string[] }>) {
    super(errors.map(e => e.message).join(', '));
  }
}

// Использование:
const client = new GraphQLClient('https://api.example.com/graphql');

const user = await client.query<User>(`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }
`, { id: '123' });
```

<details>
<summary>💡 Подсказки</summary>

- GraphQL всегда POST запрос
- Body содержит query и variables
- Проверьте наличие errors в ответе
- data содержит результат

</details>


## Задача 11.11: Polling с адаптивным интервалом 🟡

**Контекст:** Опрашивать API с увеличением интервала при отсутствии изменений.

**Задача:**
```typescript
class AdaptivePolling<T> {
  private interval: number;
  private minInterval: number;
  private maxInterval: number;
  private timer: NodeJS.Timeout | null = null;
  private lastData: T | null = null;

  constructor(
    private fetcher: () => Promise<T>,
    private onChange: (data: T) => void,
    options: {
      minInterval?: number;
      maxInterval?: number;
      increaseFactor?: number;
    } = {}
  ) {
    this.minInterval = options.minInterval || 1000;
    this.maxInterval = options.maxInterval || 60000;
    this.interval = this.minInterval;
  }

  start(): void {
    this.poll();
  }

  stop(): void {
    if (this.timer) {
      clearTimeout(this.timer);
      this.timer = null;
    }
  }

  private async poll(): Promise<void> {
    try {
      const data = await this.fetcher();

      // Проверить изменились ли данные
      if (this.hasChanged(data, this.lastData)) {
        this.onChange(data);
        this.lastData = data;

        // Сбросить интервал если есть изменения
        this.interval = this.minInterval;
      } else {
        // Увеличить интервал если нет изменений
        this.interval = Math.min(this.interval * 1.5, this.maxInterval);
      }
    } catch (error) {
      console.error('Polling error:', error);
    }

    this.timer = setTimeout(() => this.poll(), this.interval);
  }

  private hasChanged(newData: T, oldData: T | null): boolean {
    return JSON.stringify(newData) !== JSON.stringify(oldData);
  }
}

// Использование:
const polling = new AdaptivePolling(
  () => fetch('/api/status').then(r => r.json()),
  (data) => console.log('Status changed:', data),
  { minInterval: 1000, maxInterval: 30000 }
);
polling.start();
```

<details>
<summary>💡 Подсказки</summary>

- Начните с minInterval
- При изменениях - сброс на minInterval
- При отсутствии изменений - увеличение на factor
- Ограничить maxInterval
- Сравнение данных: deep equality или JSON.stringify

</details>


## Задача 11.12: Request deduplication 🟡

**Контекст:** Избежать дублирующихся одновременных запросов к одному endpoint.

**Задача:**
```typescript
class DeduplicatedApiClient {
  private inflightRequests = new Map<string, Promise<any>>();

  async get<T>(url: string, options?: RequestInit): Promise<T> {
    const key = this.getCacheKey(url, options);

    // Если запрос уже в процессе - вернуть его промис
    if (this.inflightRequests.has(key)) {
      return this.inflightRequests.get(key)!;
    }

    // Создать новый запрос
    const promise = fetch(url, options)
      .then(r => r.json())
      .finally(() => {
        // Удалить из inflight после завершения
        this.inflightRequests.delete(key);
      });

    this.inflightRequests.set(key, promise);
    return promise;
  }

  private getCacheKey(url: string, options?: RequestInit): string {
    return `${url}:${JSON.stringify(options || {})}`;
  }
}

// Использование:
const api = new DeduplicatedApiClient();

// Эти вызовы вернут один и тот же промис
const promise1 = api.get('/api/users');
const promise2 = api.get('/api/users');
console.log(promise1 === promise2); // true
```

<details>
<summary>💡 Подсказки</summary>

- Используйте Map для хранения inflight промисов
- Ключ: URL + serialized options
- Если промис уже есть - вернуть его
- После завершения удалить из Map

</details>