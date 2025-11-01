# State Management

## Задача 5.1: Простой реактивный store 🟡

**Контекст:** Создать простое хранилище состояния с подписками на изменения.

**Задача:**
```typescript
class Store<T> {
  private state: T;
  private listeners: Array<(state: T) => void> = [];

  constructor(initialState: T) {
    this.state = initialState;
  }

  getState(): T {
    // Вернуть текущее состояние
  }

  setState(newState: Partial<T>): void {
    // Обновить состояние (merge)
    // Уведомить всех подписчиков
  }

  subscribe(listener: (state: T) => void): () => void {
    // Подписаться на изменения
    // Вернуть функцию отписки
  }
}

// Использование:
const store = new Store({ count: 0, user: null });
const unsubscribe = store.subscribe((state) => {
  console.log('State changed:', state);
});
store.setState({ count: 1 }); // вызовет subscriber
```

<details>
<summary>💡 Подсказки</summary>

- Храните массив listener функций
- При setState вызывайте все listeners
- subscribe возвращает функцию для удаления listener из массива
- Используйте spread для merge: { ...this.state, ...newState }

</details>


## Задача 5.2: Computed values (вычисляемые значения) 🟡

**Контекст:** Некоторые значения должны автоматически пересчитываться при изменении state.

**Задача:**
```typescript
class StoreWithComputed<T> {
  private state: T;
  private computed: Map<string, () => any>;
  private computedCache: Map<string, any>;

  setState(newState: Partial<T>): void {
    // Обновить state
    // Очистить кэш computed values
  }

  addComputed(name: string, selector: (state: T) => any): void {
    // Добавить computed value
  }

  get(name: string): any {
    // Получить computed value (с кэшированием)
  }
}

// Использование:
const store = new StoreWithComputed({ items: [], filter: 'all' });
store.addComputed('filteredItems', (state) => {
  return state.filter === 'all'
    ? state.items
    : state.items.filter(item => item.category === state.filter);
});
console.log(store.get('filteredItems'));
```

<details>
<summary>💡 Подсказки</summary>

- Храните функции-селекторы в Map
- Кэшируйте результат вычисления
- При изменении state очищайте кэш
- При первом обращении вычисляйте и кэшируйте

</details>


## Задача 5.3: Immutable обновления вложенных объектов 🟡

**Контекст:** Обновлять глубоко вложенные поля без мутации.

**Задача:**
```typescript
// State:
const state = {
  user: {
    id: 1,
    profile: {
      name: 'John',
      address: {
        city: 'New York',
        street: '5th Avenue'
      }
    }
  }
};

// Функция: updateNested(state, path, value)
// updateNested(state, 'user.profile.address.city', 'Boston')
// Должна вернуть новый объект с измененным значением без мутации оригинала
```

<details>
<summary>💡 Подсказки</summary>

- Разделите path по точке
- Рекурсивно создавайте новые объекты на каждом уровне
- Используйте spread operator для копирования
- Альтернатива: библиотека immer, но реализуйте сами для понимания

</details>


## Задача 5.4: Undo/Redo функциональность 🟡

**Контекст:** Добавить возможность отмены и повтора действий.

**Задача:**
```typescript
class UndoableStore<T> {
  private present: T;
  private past: T[] = [];
  private future: T[] = [];

  setState(newState: T): void {
    // Добавить текущее состояние в past
    // Очистить future
    // Установить новое состояние
  }

  undo(): void {
    // Откатить к предыдущему состоянию
  }

  redo(): void {
    // Вернуть отмененное состояние
  }

  canUndo(): boolean {}
  canRedo(): boolean {}
}
```

<details>
<summary>💡 Подсказки</summary>

- past - стек предыдущих состояний
- future - стек отмененных состояний
- При новом setState очищайте future
- При undo перемещайте present → future, last past → present
- При redo перемещайте present → past, last future → present
- Ограничьте размер past (например, 50 последних состояний)

</details>


## Задача 5.5: Middleware для логирования 🟢

**Контекст:** Логировать все изменения состояния для debugging.

**Задача:**
```typescript
type Middleware<T> = (
  state: T,
  newState: Partial<T>,
  next: (state: Partial<T>) => void
) => void;

class StoreWithMiddleware<T> {
  private middlewares: Middleware<T>[] = [];

  use(middleware: Middleware<T>): void {
    // Добавить middleware
  }

  setState(newState: Partial<T>): void {
    // Прогнать через все middlewares
    // Затем применить изменение
  }
}

// Использование:
const store = new StoreWithMiddleware({ count: 0 });
store.use((state, newState, next) => {
  console.log('Before:', state);
  next(newState);
  console.log('After:', { ...state, ...newState });
});
```

<details>
<summary>💡 Подсказки</summary>

- Middleware - это цепочка функций
- Каждый middleware вызывает next() для продолжения
- Последний next() - это реальное обновление state
- Middleware могут изменить newState перед передачей дальше

</details>


## Задача 5.6: Синхронизация с localStorage 🟢

**Контекст:** Автоматически сохранять state в localStorage и восстанавливать при загрузке.

**Задача:**
```typescript
class PersistentStore<T> {
  private storageKey: string;

  constructor(initialState: T, storageKey: string) {
    // Загрузить из localStorage если есть
    // Иначе использовать initialState
  }

  setState(newState: Partial<T>): void {
    // Обновить state
    // Сохранить в localStorage
  }

  clear(): void {
    // Очистить localStorage
  }
}
```

<details>
<summary>💡 Подсказки</summary>

- Сериализуйте state в JSON перед сохранением
- При загрузке проверяйте наличие данных в localStorage
- Обрабатывайте ошибки JSON.parse (невалидные данные)
- Добавьте версионирование для миграций

</details>


## Задача 5.7: Actions и reducers паттерн 🟡

**Контекст:** Организовать изменения state через actions (как в Redux).

**Задача:**
```typescript
interface Action {
  type: string;
  payload?: any;
}

type Reducer<T> = (state: T, action: Action) => T;

class StoreWithReducer<T> {
  private state: T;
  private reducer: Reducer<T>;

  constructor(reducer: Reducer<T>, initialState: T) {
    this.reducer = reducer;
    this.state = initialState;
  }

  dispatch(action: Action): void {
    // Передать state и action в reducer
    // Обновить state результатом
    // Уведомить subscribers
  }

  getState(): T {
    return this.state;
  }
}

// Использование:
const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'SET_USER':
      return { ...state, user: action.payload };
    default:
      return state;
  }
};
const store = new StoreWithReducer(reducer, { count: 0, user: null });
store.dispatch({ type: 'INCREMENT' });
```

<details>
<summary>💡 Подсказки</summary>

- Reducer должен быть pure function
- Всегда возвращайте новый объект, не мутируйте state
- Добавьте подписки для реактивности

</details>


## Задача 5.8: Selectors с мемоизацией 🟡

**Контекст:** Избежать лишних пересчетов computed values.

**Задача:**
```typescript
function createSelector<T, R>(
  selector: (state: T) => R,
  equalityFn?: (a: R, b: R) => boolean
): (state: T) => R {
  // Создать мемоизированный selector
  // Пересчитывать только если результат изменился
}

// Использование:
const selectFilteredItems = createSelector(
  (state) => state.items.filter(item => item.active),
  (a, b) => a.length === b.length && a.every((item, i) => item === b[i])
);

// При одинаковом state вернуть закэшированный результат
```

<details>
<summary>💡 Подсказки</summary>

- Храните предыдущий state и результат
- При новом вызове сравнивайте с предыдущим state
- Если equalityFn говорит что равны - вернуть кэш
- По умолчанию используйте shallow equality

</details>


## Задача 5.9: Async actions 🟡

**Контекст:** Обрабатывать асинхронные операции в store.

**Задача:**
```typescript
class AsyncStore<T> {
  async dispatch(action: Action | AsyncAction): Promise<void> {
    // Если action - функция (thunk), вызвать ее с dispatch и getState
    // Иначе обработать как обычный action
  }
}

type AsyncAction = (
  dispatch: (action: Action) => void,
  getState: () => any
) => Promise<void>;

// Использование:
const fetchUser = (id: string): AsyncAction => async (dispatch, getState) => {
  dispatch({ type: 'FETCH_USER_START' });
  try {
    const user = await api.getUser(id);
    dispatch({ type: 'FETCH_USER_SUCCESS', payload: user });
  } catch (error) {
    dispatch({ type: 'FETCH_USER_ERROR', payload: error });
  }
};

store.dispatch(fetchUser('123'));
```

<details>
<summary>💡 Подсказки</summary>

- Проверяйте тип action: если функция - вызвать, иначе reducer
- Передавайте dispatch и getState в async action
- Обрабатывайте loading, success, error states

</details>


## Задача 5.10: Модульность (несколько reducers) 🟡

**Контекст:** Разделить state на модули, каждый со своим reducer.

**Задача:**
```typescript
function combineReducers<T>(reducers: {
  [K in keyof T]: Reducer<T[K]>
}): Reducer<T> {
  // Объединить несколько reducers в один
  // Каждый reducer управляет своей частью state
}

// Использование:
const rootReducer = combineReducers({
  user: userReducer,
  posts: postsReducer,
  comments: commentsReducer
});

// State будет: { user: {...}, posts: {...}, comments: {...} }
```

<details>
<summary>💡 Подсказки</summary>

- Верните функцию-reducer
- Пройдитесь по всем reducers
- Вызовите каждый с соответствующей частью state
- Соберите результаты в один объект

</details>


## Задача 5.11: Слежение за изменениями конкретных полей 🟢

**Контекст:** Подписаться на изменения только определенного поля.

**Задача:**
```typescript
class SelectiveStore<T> {
  subscribe<K extends keyof T>(
    key: K,
    listener: (value: T[K]) => void
  ): () => void {
    // Вызывать listener только когда state[key] изменился
  }
}

// Использование:
store.subscribe('user', (user) => {
  console.log('User changed:', user);
});
store.setState({ count: 5 }); // не вызовет listener
store.setState({ user: newUser }); // вызовет listener
```

<details>
<summary>💡 Подсказки</summary>

- Храните предыдущее значение для сравнения
- При setState сравнивайте старое и новое значение
- Вызывайте listener только если изменилось
- Используйте shallow equality или deep equality

</details>


## Задача 5.12: DevTools интеграция 🔴

**Контекст:** Интегрировать store с Redux DevTools для debugging.

**Задача:**
```typescript
class DevToolsStore<T> {
  constructor(reducer: Reducer<T>, initialState: T) {
    // Подключиться к Redux DevTools Extension
    // Отправлять все actions в DevTools
    // Поддерживать time-travel debugging
  }
}

// Должно работать с расширением Redux DevTools в браузере
```

<details>
<summary>💡 Подсказки</summary>

- Проверьте наличие window.__REDUX_DEVTOOLS_EXTENSION__
- Используйте devTools.connect()
- Отправляйте каждый action: devTools.send(action, state)
- Слушайте сообщения от DevTools для time-travel

</details>