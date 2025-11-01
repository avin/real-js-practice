# Работа с localStorage/sessionStorage

## Задача 9.1: Типобезопасное хранилище 🟡

**Контекст:** Создать обертку над localStorage с типизацией и автоматической сериализацией.

**Задача:**
```typescript
class TypedStorage {
  set<T>(key: string, value: T): void {
    // Сериализовать и сохранить
    // Обработать ошибки (quota exceeded)
  }

  get<T>(key: string, defaultValue?: T): T | undefined {
    // Получить и десериализовать
    // Вернуть defaultValue если нет
    // Обработать ошибки парсинга
  }

  remove(key: string): void {}

  clear(): void {}

  has(key: string): boolean {}
}

// Использование:
const storage = new TypedStorage();
storage.set('user', { id: 1, name: 'John' });
const user = storage.get<User>('user');
```

<details>
<summary>💡 Подсказки</summary>

- Используйте JSON.stringify/parse
- Try/catch для обработки ошибок
- Проверяйте наличие localStorage (может быть недоступен)
- Обрабатывайте QuotaExceededError

</details>


## Задача 9.2: Синхронизация между вкладками 🟡

**Контекст:** Обновлять данные во всех открытых вкладках при изменении в одной.

**Задача:**
```typescript
class SyncedStorage<T> {
  private listeners: Array<(value: T | null) => void> = [];

  constructor(private key: string) {
    // Слушать storage event
    window.addEventListener('storage', this.onStorageChange);
  }

  private onStorageChange = (e: StorageEvent): void {
    // Проверить что изменился наш ключ
    // Уведомить всех listeners
  };

  set(value: T): void {
    localStorage.setItem(this.key, JSON.stringify(value));
    // storage event не срабатывает в текущей вкладке
    // Уведомить listeners вручную
  }

  get(): T | null {
    const item = localStorage.getItem(this.key);
    return item ? JSON.parse(item) : null;
  }

  subscribe(listener: (value: T | null) => void): () => void {
    this.listeners.push(listener);
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }
}

// Использование:
const userStorage = new SyncedStorage<User>('user');
userStorage.subscribe((user) => {
  console.log('User updated in another tab:', user);
});
```

<details>
<summary>💡 Подсказки</summary>

- storage event срабатывает только в других вкладках
- В текущей вкладке уведомляйте listeners вручную
- e.key содержит ключ изменившегося item
- e.newValue содержит новое значение

</details>


## Задача 9.3: Управление размером хранилища 🟡

**Контекст:** Ограничить размер localStorage, удаляя старые записи при превышении лимита.

**Задача:**
```typescript
interface StorageItem<T> {
  value: T;
  timestamp: number;
  size: number; // в байтах
}

class SizeLimitedStorage {
  private maxSize: number; // в байтах
  private currentSize = 0;

  constructor(maxSize: number = 5 * 1024 * 1024) { // 5MB default
    this.maxSize = maxSize;
    this.calculateCurrentSize();
  }

  set<T>(key: string, value: T): void {
    const item: StorageItem<T> = {
      value,
      timestamp: Date.now(),
      size: this.getSize(value)
    };

    // Если превышен лимит - удалить старые записи
    // Сохранить новую запись
  }

  private evictOldest(): void {
    // Удалить самую старую запись
  }

  private getSize(value: any): number {
    // Вычислить размер в байтах
    return new Blob([JSON.stringify(value)]).size;
  }

  private calculateCurrentSize(): void {
    // Посчитать общий размер всех записей
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- Сохраняйте metadata (timestamp, size) вместе с данными
- При добавлении проверяйте: currentSize + newSize > maxSize
- Если превышен - удаляйте самые старые записи
- Сортируйте по timestamp для поиска старых

</details>


## Задача 9.4: Экспирация (TTL) для записей 🟡

**Контекст:** Автоматически удалять устаревшие записи из storage.

**Задача:**
```typescript
interface CachedItem<T> {
  value: T;
  expiresAt: number;
}

class ExpiringStorage {
  set<T>(key: string, value: T, ttl: number): void {
    // ttl в миллисекундах
    // Сохранить с временем истечения
  }

  get<T>(key: string): T | null {
    const item = this.getItem<T>(key);
    if (!item) return null;

    // Проверить не истек ли срок
    if (Date.now() > item.expiresAt) {
      this.remove(key);
      return null;
    }

    return item.value;
  }

  private getItem<T>(key: string): CachedItem<T> | null {
    const raw = localStorage.getItem(key);
    return raw ? JSON.parse(raw) : null;
  }

  cleanup(): void {
    // Удалить все истекшие записи
  }
}

// Использование:
const storage = new ExpiringStorage();
storage.set('token', 'abc123', 3600000); // 1 час
```

<details>
<summary>💡 Подсказки</summary>

- Сохраняйте expiresAt = Date.now() + ttl
- При get проверяйте: Date.now() > expiresAt
- cleanup должен проходить по всем ключам localStorage
- Запускайте cleanup периодически или при get

</details>


## Задача 9.5: Миграции данных 🟡

**Контекст:** При изменении структуры данных мигрировать старые записи.

**Задача:**
```typescript
type Migration = {
  version: number;
  migrate: (data: any) => any;
};

class MigratableStorage {
  private currentVersion: number;
  private migrations: Migration[];

  constructor(migrations: Migration[]) {
    this.migrations = migrations.sort((a, b) => a.version - b.version);
    this.currentVersion = Math.max(...migrations.map(m => m.version));
  }

  set(key: string, value: any): void {
    const item = {
      version: this.currentVersion,
      data: value
    };
    localStorage.setItem(key, JSON.stringify(item));
  }

  get(key: string): any {
    const raw = localStorage.getItem(key);
    if (!raw) return null;

    const item = JSON.parse(raw);
    const version = item.version || 0;

    // Применить все миграции от version до currentVersion
    const data = this.runMigrations(item.data, version);

    // Сохранить мигрированные данные
    this.set(key, data);

    return data;
  }

  private runMigrations(data: any, fromVersion: number): any {
    // Применить миграции последовательно
  }
}

// Использование:
const storage = new MigratableStorage([
  {
    version: 1,
    migrate: (data) => ({ ...data, newField: 'default' })
  },
  {
    version: 2,
    migrate: (data) => ({ ...data, renamed: data.oldField })
  }
]);
```

<details>
<summary>💡 Подсказки</summary>

- Сохраняйте версию с каждой записью
- При get сравнивайте версию записи с текущей
- Применяйте все миграции между версиями последовательно
- После миграции сохраните обновленные данные

</details>


## Задача 9.6: Compressed storage 🔴

**Контекст:** Сжимать большие данные перед сохранением в localStorage.

**Задача:**
```typescript
class CompressedStorage {
  async set(key: string, value: any): Promise<void> {
    const json = JSON.stringify(value);

    // Сжать используя CompressionStream API
    const compressed = await this.compress(json);

    // Конвертировать в base64 для хранения
    const base64 = this.arrayBufferToBase64(compressed);

    localStorage.setItem(key, base64);
  }

  async get<T>(key: string): Promise<T | null> {
    const base64 = localStorage.getItem(key);
    if (!base64) return null;

    // Декодировать из base64
    const compressed = this.base64ToArrayBuffer(base64);

    // Распаковать
    const json = await this.decompress(compressed);

    return JSON.parse(json);
  }

  private async compress(text: string): Promise<ArrayBuffer> {
    // Использовать CompressionStream
  }

  private async decompress(buffer: ArrayBuffer): Promise<string> {
    // Использовать DecompressionStream
  }

  private arrayBufferToBase64(buffer: ArrayBuffer): string {
    // Конвертация для хранения в string
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- Используйте CompressionStream('gzip')
- Создайте ReadableStream из строки
- Пропустите через compression stream
- Соберите chunks и объедините в ArrayBuffer
- Конвертируйте в base64 для хранения как строка

</details>


## Задача 9.7: Namespace для модулей 🟢

**Контекст:** Изолировать данные разных модулей приложения.

**Задача:**
```typescript
class NamespacedStorage {
  constructor(private namespace: string) {}

  private getKey(key: string): string {
    return `${this.namespace}:${key}`;
  }

  set<T>(key: string, value: T): void {
    localStorage.setItem(this.getKey(key), JSON.stringify(value));
  }

  get<T>(key: string): T | null {
    const item = localStorage.getItem(this.getKey(key));
    return item ? JSON.parse(item) : null;
  }

  keys(): string[] {
    // Вернуть все ключи в этом namespace
  }

  clear(): void {
    // Удалить все записи в этом namespace
  }
}

// Использование:
const userStorage = new NamespacedStorage('user');
const appStorage = new NamespacedStorage('app');

userStorage.set('token', 'abc'); // сохранит как 'user:token'
appStorage.set('token', 'xyz'); // сохранит как 'app:token'
```

<details>
<summary>💡 Подсказки</summary>

- Добавляйте префикс к каждому ключу
- keys() должен фильтровать ключи по префиксу
- Используйте Object.keys(localStorage)
- Удаляйте префикс при возврате ключей

</details>


## Задача 9.8: Observable storage 🟡

**Контекст:** Создать реактивное хранилище с подписками на изменения.

**Задача:**
```typescript
class ObservableStorage {
  private listeners = new Map<string, Set<(value: any) => void>>();

  set<T>(key: string, value: T): void {
    localStorage.setItem(key, JSON.stringify(value));
    this.notify(key, value);
  }

  get<T>(key: string): T | null {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : null;
  }

  subscribe<T>(key: string, listener: (value: T | null) => void): () => void {
    if (!this.listeners.has(key)) {
      this.listeners.set(key, new Set());
    }
    this.listeners.get(key)!.add(listener);

    // Немедленно вызвать с текущим значением
    listener(this.get(key));

    // Вернуть функцию отписки
    return () => {
      this.listeners.get(key)?.delete(listener);
    };
  }

  private notify(key: string, value: any): void {
    // Уведомить всех подписчиков этого ключа
  }
}

// Использование:
const storage = new ObservableStorage();
storage.subscribe('user', (user) => {
  console.log('User changed:', user);
});
storage.set('user', { name: 'John' }); // вызовет callback
```

<details>
<summary>💡 Подсказки</summary>

- Храните Map<key, Set<listener>>
- При set уведомляйте всех слушателей этого ключа
- subscribe вызывает listener сразу с текущим значением
- Верните функцию для удаления listener

</details>


## Задача 9.9: Backup и restore 🟢

**Контекст:** Экспортировать все данные localStorage и импортировать обратно.

**Задача:**
```typescript
class StorageBackup {
  export(keys?: string[]): string {
    // Экспортировать все или указанные ключи в JSON
    const data: Record<string, any> = {};

    const keysToExport = keys || Object.keys(localStorage);

    for (const key of keysToExport) {
      const value = localStorage.getItem(key);
      if (value !== null) {
        data[key] = value;
      }
    }

    return JSON.stringify(data, null, 2);
  }

  import(json: string, overwrite: boolean = false): void {
    // Импортировать данные
    // Если overwrite = false, не перезаписывать существующие
  }

  download(filename: string = 'storage-backup.json'): void {
    // Скачать backup как файл
  }

  restore(file: File): Promise<void> {
    // Восстановить из файла
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- export собирает данные в объект и stringify
- import парсит и сохраняет в localStorage
- download создает Blob и использует URL.createObjectURL
- restore читает файл через FileReader

</details>


## Задача 9.10: Encrypted storage 🔴

**Контекст:** Шифровать чувствительные данные перед сохранением.

**Задача:**
```typescript
class EncryptedStorage {
  constructor(private encryptionKey: string) {}

  async set(key: string, value: any): Promise<void> {
    const json = JSON.stringify(value);
    const encrypted = await this.encrypt(json);
    localStorage.setItem(key, encrypted);
  }

  async get<T>(key: string): Promise<T | null> {
    const encrypted = localStorage.getItem(key);
    if (!encrypted) return null;

    const json = await this.decrypt(encrypted);
    return JSON.parse(json);
  }

  private async encrypt(text: string): Promise<string> {
    // Использовать Web Crypto API
    // AES-GCM шифрование
  }

  private async decrypt(encrypted: string): Promise<string> {
    // Расшифровать
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- Используйте crypto.subtle.encrypt/decrypt
- Алгоритм: AES-GCM
- Генерируйте IV (initialization vector) для каждого шифрования
- Сохраняйте IV вместе с зашифрованными данными
- Конвертируйте в base64 для хранения как строка

</details>


## Задача 9.11: Автосохранение с debounce 🟢

**Контекст:** Автоматически сохранять изменения в localStorage с задержкой.

**Задача:**
```typescript
class AutoSaveStorage<T> {
  private data: T;
  private saveTimeout: NodeJS.Timeout | null = null;
  private saveDelay: number;

  constructor(
    private key: string,
    initialData: T,
    saveDelay: number = 1000
  ) {
    this.data = this.load() || initialData;
    this.saveDelay = saveDelay;
  }

  update(updater: (data: T) => T): void {
    this.data = updater(this.data);
    this.scheduleSave();
  }

  private scheduleSave(): void {
    // Отменить предыдущий таймер
    // Запланировать новый save через saveDelay
  }

  private save(): void {
    localStorage.setItem(this.key, JSON.stringify(this.data));
  }

  private load(): T | null {
    const item = localStorage.getItem(this.key);
    return item ? JSON.parse(item) : null;
  }

  getData(): T {
    return this.data;
  }

  flush(): void {
    // Немедленно сохранить
  }
}

// Использование:
const formData = new AutoSaveStorage('form', { name: '', email: '' });
formData.update(data => ({ ...data, name: 'John' }));
// Сохранится через 1 секунду автоматически
```

<details>
<summary>💡 Подсказки</summary>

- Используйте setTimeout для задержки
- Отменяйте предыдущий таймер при новом update
- flush() отменяет таймер и сохраняет немедленно
- При размонтировании компонента вызывайте flush()

</details>


## Задача 9.12: Версионирование и откат 🟡

**Контекст:** Сохранять историю изменений и возможность отката.

**Задача:**
```typescript
interface VersionedData<T> {
  current: T;
  history: Array<{ timestamp: number; data: T }>;
  maxHistory: number;
}

class VersionedStorage<T> {
  constructor(
    private key: string,
    private maxHistory: number = 10
  ) {}

  set(value: T): void {
    const versioned = this.load();

    // Добавить текущее значение в историю
    versioned.history.push({
      timestamp: Date.now(),
      data: versioned.current
    });

    // Ограничить размер истории
    if (versioned.history.length > this.maxHistory) {
      versioned.history.shift();
    }

    versioned.current = value;
    this.save(versioned);
  }

  get(): T | null {
    return this.load().current;
  }

  getHistory(): Array<{ timestamp: number; data: T }> {
    return this.load().history;
  }

  rollback(index: number): void {
    // Откатиться к версии из истории
  }

  private load(): VersionedData<T> {
    // Загрузить или вернуть пустую структуру
  }

  private save(data: VersionedData<T>): void {
    localStorage.setItem(this.key, JSON.stringify(data));
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- Сохраняйте текущее значение в history перед обновлением
- Ограничивайте массив history (shift старые)
- rollback берет значение из history и делает его current
- Добавьте timestamp для каждой версии

</details>