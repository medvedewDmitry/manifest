# Фронтенд разработка

## Зачем нужны эти принципы?

Фронтенд — это то, что видят пользователи. Качественный фронтенд должен быть:

- **Быстрым** — загружается и работает быстро
- **Доступным** — работает для всех пользователей
- **Надежным** — не ломается при ошибках
- **Поддерживаемым** — легко развивать и исправлять

## Основные принципы фронтенд разработки

### 1. Компонентный подход

**Принцип**: разбивай интерфейс на маленькие, переиспользуемые компоненты.

```tsx
// ❌ Плохо — один большой компонент
function UserProfile() {
  return (
    <div>
      <h1>Профиль пользователя</h1>
      <div>
        <img src={user.avatar} alt="Avatar" />
        <h2>{user.name}</h2>
        <p>{user.email}</p>
        <button onClick={handleEdit}>Редактировать</button>
        <button onClick={handleDelete}>Удалить</button>
      </div>
    </div>
  );
}

// ✅ Хорошо — разбито на компоненты
function UserProfile() {
  return (
    <div>
      <PageTitle title="Профиль пользователя" />
      <UserAvatar src={user.avatar} alt={user.name} />
      <UserInfo user={user} />
      <UserActions onEdit={handleEdit} onDelete={handleDelete} />
    </div>
  );
}

function UserAvatar({ src, alt }: { src: string; alt: string }) {
  return <img src={src} alt={alt} className="rounded-full w-16 h-16" />;
}

function UserInfo({ user }: { user: User }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### 2. Иммутабельность данных

**Принцип**: не изменяй данные напрямую, создавай новые объекты.

```javascript
// ❌ Плохо — мутация данных
const user = { name: 'John', age: 30 };
user.age = 31; // Мутация!

// ✅ Хорошо — иммутабельность
const user = { name: 'John', age: 30 };
const updatedUser = { ...user, age: 31 };

// ✅ Хорошо — с массивами
const users = [{ id: 1, name: 'John' }, { id: 2, name: 'Jane' }];
const updatedUsers = users.map(user => 
  user.id === 1 ? { ...user, name: 'Johnny' } : user
);

// ✅ Хорошо — с вложенными объектами
const state = {
  user: { profile: { name: 'John', settings: { theme: 'dark' } } }
};
const newState = {
  ...state,
  user: {
    ...state.user,
    profile: {
      ...state.user.profile,
      settings: {
        ...state.user.profile.settings,
        theme: 'light'
      }
    }
  }
};
```

### 3. Обработка состояний загрузки и ошибок

**Принцип**: всегда показывай пользователю, что происходит.

```tsx
// ✅ Хорошо — обработка всех состояний
function UserList() {
  const { data: users, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers
  });

  if (isLoading) {
    return <LoadingSpinner message="Загружаем пользователей..." />;
  }

  if (error) {
    return <ErrorMessage 
      message="Не удалось загрузить пользователей" 
      onRetry={() => refetch()} 
    />;
  }

  if (!users?.length) {
    return <EmptyState message="Пользователи не найдены" />;
  }

  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}

// Компоненты состояний
function LoadingSpinner({ message }: { message: string }) {
  return (
    <div className="flex items-center justify-center p-8">
      <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600"></div>
      <span className="ml-3">{message}</span>
    </div>
  );
}

function ErrorMessage({ message, onRetry }: { message: string; onRetry: () => void }) {
  return (
    <div className="text-center p-8">
      <p className="text-red-600 mb-4">{message}</p>
      <button 
        onClick={onRetry}
        className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Попробовать снова
      </button>
    </div>
  );
}
```

## JavaScript — современные практики

### 1. Использование современных возможностей

```javascript
// ✅ Используй современный синтаксис
// Деструктуризация
const { name, email, ...rest } = user;

// Spread оператор
const newUser = { ...user, updatedAt: new Date() };

// Опциональная цепочка
const userName = user?.profile?.name ?? 'Неизвестно';

// Оператор нулевого состояния
const theme = userSettings.theme ?? 'light';

// Шаблонные строки (литералы)
const message = `Привет, ${name}! Твой email: ${email}`;

// Стрелочные функции
const getFullName = (firstName, lastName) => `${firstName} ${lastName}`;

// Методы массивов
const activeUsers = users
  .filter(user => user.isActive)
  .map(user => ({ id: user.id, name: user.name }))
  .sort((a, b) => a.name.localeCompare(b.name));

// Метод reduce для агрегации данных
const userStats = users.reduce((stats, user) => {
  stats.total++;
  if (user.isActive) stats.active++;
  if (user.role === 'admin') stats.admins++;
  return stats;
}, { total: 0, active: 0, admins: 0 });

// Тернарный оператор для условной логики (вложенные тернарники НЕ используем!)
const userStatus = user.isActive ? 'Активен' : 'Неактивен';
```

### 2. Асинхронное программирование

```javascript
// ✅ Используй async/await вместо промисов
async function fetchUserData(userId) {
  try {
    const [user, posts, settings] = await Promise.all([
      fetchUser(userId),
      fetchUserPosts(userId),
      fetchUserSettings(userId)
    ]);

    return { user, posts, settings };
  } catch (error) {
    console.error('Failed to fetch user data:', error);
    throw new Error('USER_DATA_FETCH_FAILED');
  }
}

// ✅ Обработка ошибок с retry
async function fetchWithRetry(url, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      return await response.json();
    } catch (error) {
      if (i === maxRetries - 1) {
        throw error;
      }
      
      // Экспоненциальная задержка
      await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
    }
  }
}
```

### 3. Валидация данных

```javascript
// ✅ Создай утилиты для валидации (можно в формате обхекта, можно отдельными экспортными утилитами)
const validators = {
  email: (email) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  },
  
  password: (password) => {
    return password.length >= 8 && 
           /[A-Z]/.test(password) && 
           /[a-z]/.test(password) && 
           /\d/.test(password);
  },
  
  required: (value) => {
    return value !== null && value !== undefined && value !== '';
  }
};

// ✅ Использование валидаторов
function validateUserData(data) {
  const errors = {};
  
  if (!validators.required(data.name)) {
    errors.name = 'Имя обязательно';
  }
  
  if (!validators.email(data.email)) {
    errors.email = 'Некорректный email';
  }
  
  if (!validators.password(data.password)) {
    errors.password = 'Пароль должен содержать минимум 8 символов, заглавную и строчную букву, цифру';
  }
  
  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
}
```

## TypeScript — типобезопасность

### 1. Строгая типизация

```typescript
// ✅ Определяй типы для всего
interface IUser {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  createdAt: Date;
  isActive: boolean;
}

interface IApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

// ✅ Используй дженерики, если есть необходимость в динамических типах
async function apiRequest<T>(url: string): Promise<ApiResponse<T>> {
  const response = await fetch(url);
  
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  
  return response.json();
}

// ✅ Типизированные хуки
function useUser(userId: string) {
  return useQuery<ApiResponse<User>>({
    queryKey: ['user', userId],
    queryFn: () => apiRequest<User>(`/api/users/${userId}`)
  });
}
```

### 2. Сужение типов

```typescript
// ✅ Сужение unknown типов вместо any (не используем any, если не знаем, то через проверки сужаем)
function parseUserData(data: unknown): User {
  if (
    typeof data === 'object' && 
    data !== null &&
    'id' in data &&
    'name' in data &&
    'email' in data
  ) {
    const userData = data as any;
    
    if (
      typeof userData.id === 'string' &&
      typeof userData.name === 'string' &&
      typeof userData.email === 'string'
    ) {
      return {
        id: userData.id,
        name: userData.name,
        email: userData.email,
        avatar: userData.avatar,
        createdAt: new Date(userData.createdAt),
        isActive: Boolean(userData.isActive)
      };
    }
  }
  
  throw new Error('Invalid user data format');
}

// ✅ Type guards
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj &&
    'email' in obj
  );
}
```

### 3. Утилитарные типы

```typescript
// ✅ Используй утилитарные типы
type UserFormData = Pick<User, 'name' | 'email'>;
type UserUpdateData = Partial<User>;
type UserId = User['id'];

// ✅ Условные типы
type ApiEndpoint<T> = T extends 'user' ? '/api/users' : '/api/posts';

// ✅ Mapped types
type ApiEndpoints = {
  [K in 'users' | 'posts' | 'comments']: `/api/${K}`;
};

// ✅ Template literal types
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
type ApiUrl = `${HttpMethod} /api/${string}`;
```

## React — современные практики

### 1. Функциональные компоненты и хуки

```tsx
// ✅ Используй функциональные компоненты
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading, error } = useUser(userId);
  const [isEditing, setIsEditing] = useState(false);
  
  const handleSave = useCallback(async (userData: UserFormData) => {
    try {
      await updateUser(userId, userData);
      setIsEditing(false);
    } catch (error) {
      console.error('Failed to update user:', error);
    }
  }, [userId]);

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <NotFound />;

  return (
    <div>
      {isEditing ? (
        <UserEditForm 
          user={user} 
          onSave={handleSave}
          onCancel={() => setIsEditing(false)}
        />
      ) : (
        <UserDisplay 
          user={user}
          onEdit={() => setIsEditing(true)}
        />
      )}
    </div>
  );
}
```

### 2. Кастомные хуки

```tsx
// ✅ Создавай кастомные хуки для переиспользования логики
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error('Error reading from localStorage:', error);
      return initialValue;
    }
  });

  const setValue = useCallback((value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('Error setting localStorage:', error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue] as const;
}

// ✅ Хук для API запросов
function useApiRequest<T>(url: string, options?: RequestInit) {
  const [data, setData] = useState<T | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let isMounted = true;

    const fetchData = async () => {
      try {
        setIsLoading(true);
        setError(null);
        
        const response = await fetch(url, options);
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }
        
        const result = await response.json();
        
        if (isMounted) {
          setData(result);
        }
      } catch (err) {
        if (isMounted) {
          setError(err instanceof Error ? err : new Error('Unknown error'));
        }
      } finally {
        if (isMounted) {
          setIsLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      isMounted = false;
    };
  }, [url, JSON.stringify(options)]);

  return { data, isLoading, error };
}
```

### 3. Оптимизация производительности

#### Когда использовать мемоизацию

**Принцип**: мемоизируй только то, что действительно дорого вычислять или часто перерендеривается.

```tsx
// ✅ useMemo — для дорогих вычислений
function UserStats({ users }: { users: User[] }) {
  // Мемоизируем только если вычисления дорогие
  const stats = useMemo(() => {
    return {
      total: users.length,
      active: users.filter(u => u.isActive).length,
      inactive: users.filter(u => !u.isActive).length,
      // Дорогое вычисление — группировка по ролям
      byRole: users.reduce((acc, user) => {
        acc[user.role] = (acc[user.role] || 0) + 1;
        return acc;
      }, {} as Record<string, number>)
    };
  }, [users]); // Зависимость — массив пользователей

  return (
    <div>
      <p>Всего: {stats.total}</p>
      <p>Активных: {stats.active}</p>
      <p>Неактивных: {stats.inactive}</p>
    </div>
  );
}

// ❌ НЕ мемоизируй простые вычисления
function SimpleComponent({ count }: { count: number }) {
  // Это НЕ нужно мемоизировать — слишком просто
  const doubled = useMemo(() => count * 2, [count]);
  
  // Лучше так:
  const doubled = count * 2;
  
  return <div>{doubled}</div>;
}

// ✅ useMemo для фильтрации и сортировки
function UserList({ users, searchTerm, sortBy }: UserListProps) {
  const filteredAndSortedUsers = useMemo(() => {
    let result = users;
    
    // Фильтрация
    if (searchTerm) {
      result = result.filter(user => 
        user.name.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }
    
    // Сортировка
    if (sortBy) {
      result = [...result].sort((a, b) => {
        if (sortBy === 'name') return a.name.localeCompare(b.name);
        if (sortBy === 'email') return a.email.localeCompare(b.email);
        return 0;
      });
    }
    
    return result;
  }, [users, searchTerm, sortBy]);

  return (
    <div>
      {filteredAndSortedUsers.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

#### Когда использовать useCallback

**Принцип**: используй useCallback для функций, которые передаются как пропсы в мемоизированные компоненты.

```tsx
// ✅ useCallback для обработчиков событий
function UserList({ users }: { users: User[] }) {
  const [selectedUser, setSelectedUser] = useState<User | null>(null);
  
  // Мемоизируем функцию, которая передается в мемоизированный компонент
  const handleUserSelect = useCallback((user: User) => {
    setSelectedUser(user);
  }, []); // Пустой массив зависимостей — функция не зависит от внешних переменных
  
  const handleUserEdit = useCallback((user: User) => {
    // Логика редактирования
    console.log('Editing user:', user);
  }, []); // Пустой массив — функция стабильна
  
  return (
    <div>
      {users.map(user => (
        <UserCard 
          key={user.id} 
          user={user}
          onSelect={handleUserSelect} // Передаем мемоизированную функцию
          onEdit={handleUserEdit}
        />
      ))}
    </div>
  );
}

// ✅ useCallback с зависимостями
function UserForm({ user, onSave }: { user: User; onSave: (user: User) => void }) {
  const [formData, setFormData] = useState(user);
  
  // Функция зависит от formData, поэтому включаем в зависимости
  const handleSubmit = useCallback((e: React.FormEvent) => {
    e.preventDefault();
    onSave(formData);
  }, [formData, onSave]); // Зависимости: formData и onSave
  
  return (
    <form onSubmit={handleSubmit}>
      {/* поля формы */}
    </form>
  );
}

// ❌ НЕ используй useCallback без необходимости
function SimpleButton({ onClick, children }: { onClick: () => void; children: React.ReactNode }) {
  // Это НЕ нужно — функция не передается в мемоизированные компоненты
  const handleClick = useCallback(() => {
    onClick();
  }, [onClick]);
  
  // Лучше так:
  return <button onClick={onClick}>{children}</button>;
}
```

#### Когда использовать React.memo

**Принцип**: мемоизируй компоненты, которые часто перерендериваются с теми же пропсами.

```tsx
// ✅ React.memo для компонентов, которые часто перерендериваются
const UserCard = memo(function UserCard({ 
  user, 
  onEdit, 
  onDelete 
}: { 
  user: User; 
  onEdit: (user: User) => void;
  onDelete: (userId: string) => void;
}) {
  return (
    <div className="border rounded p-4">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <div className="flex gap-2">
        <button onClick={() => onEdit(user)}>Редактировать</button>
        <button onClick={() => onDelete(user.id)}>Удалить</button>
      </div>
    </div>
  );
});

// ✅ Кастомная функция сравнения для React.memo
const ExpensiveChart = memo(function ExpensiveChart({ 
  data, 
  config 
}: { 
  data: ChartData[]; 
  config: ChartConfig; 
}) {
  // Дорогой рендеринг графика
  return <div>Chart with {data.length} points</div>;
}, (prevProps, nextProps) => {
  // Кастомное сравнение — компонент перерендерится только если данные действительно изменились
  return (
    prevProps.data.length === nextProps.data.length &&
    prevProps.config.theme === nextProps.config.theme
  );
});

// ❌ НЕ мемоизируй простые компоненты
const SimpleText = memo(function SimpleText({ text }: { text: string }) {
  // Это НЕ нужно — компонент слишком простой
  return <span>{text}</span>;
});

// Лучше так:
function SimpleText({ text }: { text: string }) {
  return <span>{text}</span>;
}
```

#### Ленивая загрузка компонентов

```tsx
// ✅ Ленивая загрузка для тяжелых компонентов
const HeavyChart = lazy(() => import('./HeavyChart'));
const UserAnalytics = lazy(() => import('./UserAnalytics'));
const AdminPanel = lazy(() => import('./AdminPanel'));

function Dashboard({ userRole }: { userRole: string }) {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Всегда загружаем основной контент */}
      <UserStats />
      
      {/* Лениво загружаем тяжелые компоненты */}
      <Suspense fallback={<div>Загружаем график...</div>}>
        <HeavyChart />
      </Suspense>
      
      {/* Условная ленивая загрузка */}
      {userRole === 'admin' && (
        <Suspense fallback={<div>Загружаем панель администратора...</div>}>
          <AdminPanel />
        </Suspense>
      )}
    </div>
  );
}

// ✅ Ленивая загрузка маршрутов
const UserProfile = lazy(() => import('./pages/UserProfile'));
const UserSettings = lazy(() => import('./pages/UserSettings'));

function App() {
  return (
    <Router>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/profile" element={<UserProfile />} />
          <Route path="/settings" element={<UserSettings />} />
        </Routes>
      </Suspense>
    </Router>
  );
}
```

#### Практические рекомендации

```tsx
// ✅ Комбинирование техник оптимизации
function OptimizedUserList({ users, searchTerm }: { users: User[]; searchTerm: string }) {
  // 1. Мемоизируем фильтрацию
  const filteredUsers = useMemo(() => {
    if (!searchTerm) return users;
    return users.filter(user => 
      user.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [users, searchTerm]);
  
  // 2. Мемоизируем обработчики
  const handleUserSelect = useCallback((user: User) => {
    console.log('Selected:', user);
  }, []);
  
  const handleUserEdit = useCallback((user: User) => {
    console.log('Edit:', user);
  }, []);
  
  // 3. Мемоизированный компонент списка
  return (
    <div>
      {filteredUsers.map(user => (
        <UserCard 
          key={user.id}
          user={user}
          onSelect={handleUserSelect}
          onEdit={handleUserEdit}
        />
      ))}
    </div>
  );
}

// ✅ Избегай преждевременной оптимизации
function UserProfile({ user }: { user: User }) {
  // НЕ оптимизируй, пока не измеришь производительность
  // Сначала напиши простой код, потом профилируй и оптимизируй
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <UserPosts userId={user.id} />
    </div>
  );
}
```

#### Инструменты для профилирования

```tsx
// ✅ React DevTools Profiler
import { Profiler } from 'react';

function App() {
  const handleProfiler = (id: string, phase: string, actualDuration: number) => {
    console.log(`${id} took ${actualDuration}ms to ${phase}`);
  };
  
  return (
    <Profiler id="App" onRender={handleProfiler}>
      <UserList users={users} />
    </Profiler>
  );
}

// ✅ useDebugValue для отладки
function useExpensiveCalculation(data: number[]) {
  const result = useMemo(() => {
    return data.reduce((sum, item) => sum + item, 0);
  }, [data]);
  
  // Показывает значение в React DevTools
  useDebugValue(result, (value) => `Sum: ${value}`);
  
  return result;
}
```

## Vue — современные практики

### 1. Composition API

```vue
<!-- ✅ Используй Composition API -->
<template>
  <div>
    <h1>{{ user.name }}</h1>
    <p>{{ user.email }}</p>
    <button @click="toggleEdit">
      {{ isEditing ? 'Отменить' : 'Редактировать' }}
    </button>
    
    <UserEditForm 
      v-if="isEditing"
      :user="user"
      @save="handleSave"
      @cancel="toggleEdit"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useUser } from '@/composables/useUser';
import UserEditForm from './UserEditForm.vue';

interface Props {
  userId: string;
}

const props = defineProps<Props>();
const emit = defineEmits<{
  update: [user: User];
}>();

const { user, isLoading, error, updateUser } = useUser(props.userId);
const isEditing = ref(false);

const toggleEdit = () => {
  isEditing.value = !isEditing.value;
};

const handleSave = async (userData: UserFormData) => {
  try {
    await updateUser(userData);
    isEditing.value = false;
    emit('update', user.value);
  } catch (error) {
    console.error('Failed to update user:', error);
  }
};
</script>
```

### 2. Кастомные композиционные функции

```typescript
// ✅ Создавай композиционные функции
export function useLocalStorage<T>(key: string, defaultValue: T) {
  const stored = ref<T>(defaultValue);
  
  const setValue = (value: T) => {
    stored.value = value;
    localStorage.setItem(key, JSON.stringify(value));
  };
  
  const getValue = () => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch {
      return defaultValue;
    }
  };
  
  onMounted(() => {
    stored.value = getValue();
  });
  
  return {
    value: readonly(stored),
    setValue
  };
}

// ✅ Хук для API
export function useApiRequest<T>(url: string) {
  const data = ref<T | null>(null);
  const isLoading = ref(true);
  const error = ref<Error | null>(null);
  
  const execute = async () => {
    try {
      isLoading.value = true;
      error.value = null;
      
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      data.value = await response.json();
    } catch (err) {
      error.value = err instanceof Error ? err : new Error('Unknown error');
    } finally {
      isLoading.value = false;
    }
  };
  
  onMounted(execute);
  
  return {
    data: readonly(data),
    isLoading: readonly(isLoading),
    error: readonly(error),
    execute
  };
}
```

### 3. Типизация в Vue

```vue
<template>
  <div>
    <UserList :users="users" @select="handleUserSelect" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import type { User } from '@/types';

interface Props {
  initialUsers?: User[];
}

const props = withDefaults(defineProps<Props>(), {
  initialUsers: () => []
});

const emit = defineEmits<{
  userSelect: [user: User];
}>();

const users = ref<User[]>(props.initialUsers);

const handleUserSelect = (user: User) => {
  emit('userSelect', user);
};
</script>
```

## Качество кода

### 1. Линтинг и форматирование

```json
// .eslintrc.json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended"
  ],
  "rules": {
    "no-console": "warn",
    "prefer-const": "error",
    "no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn",
    "react/prop-types": "off"
  }
}
```

### 2. Тестирование
Пока в существующих проектах не используйтеся, но к этому стремимся.

Пример тестирования: <ссылка на проект с тестированием>
```typescript
// ✅ Unit тесты компонентов
import { render, screen, fireEvent } from '@testing-library/react';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com'
  };

  it('renders user information', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('calls onEdit when edit button is clicked', () => {
    const onEdit = jest.fn();
    render(<UserCard user={mockUser} onEdit={onEdit} />);
    
    fireEvent.click(screen.getByText('Редактировать'));
    
    expect(onEdit).toHaveBeenCalledWith(mockUser);
  });
});

// ✅ Тестирование хуков
import { renderHook, act } from '@testing-library/react';
import { useLocalStorage } from './useLocalStorage';

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it('returns initial value when localStorage is empty', () => {
    const { result } = renderHook(() => useLocalStorage('test', 'default'));
    
    expect(result.current[0]).toBe('default');
  });

  it('updates localStorage when value changes', () => {
    const { result } = renderHook(() => useLocalStorage('test', 'default'));
    
    act(() => {
      result.current[1]('new value');
    });
    
    expect(result.current[0]).toBe('new value');
    expect(localStorage.getItem('test')).toBe('"new value"');
  });
});
```

## Производительность

### 1. Оптимизация загрузки

```typescript
// ✅ Ленивая загрузка маршрутов
const routes = [
  {
    path: '/users',
    component: () => import('./pages/Users.vue')
  },
  {
    path: '/users/:id',
    component: () => import('./pages/UserDetail.vue')
  }
];

// ✅ Ленивая загрузка компонентов
const HeavyChart = lazy(() => import('./components/HeavyChart.vue'));

// ✅ Предзагрузка критических ресурсов
<link rel="preload" href="/critical.css" as="style">
<link rel="preload" href="/main.js" as="script">
```

### 2. Оптимизация изображений

```vue
<template>
  <!-- ✅ Используй современные форматы -->
  <picture>
    <source srcset="image.webp" type="image/webp">
    <img src="image.jpg" alt="Description" loading="lazy">
  </picture>
  
  <!-- ✅ Ленивая загрузка -->
  <img 
    src="placeholder.jpg" 
    data-src="actual-image.jpg" 
    loading="lazy"
    @load="handleImageLoad"
  >
</template>
```

## Доступность (A11y)

### 1. Семантическая разметка

```tsx
// ✅ Используй семантические теги
function UserProfile({ user }: { user: User }) {
  return (
    <main>
      <header>
        <h1>Профиль пользователя</h1>
      </header>
      
      <section aria-labelledby="user-info">
        <h2 id="user-info">Информация о пользователе</h2>
        <dl>
          <dt>Имя:</dt>
          <dd>{user.name}</dd>
          <dt>Email:</dt>
          <dd>{user.email}</dd>
        </dl>
      </section>
      
      <nav aria-label="Действия пользователя">
        <button onClick={handleEdit}>Редактировать</button>
        <button onClick={handleDelete}>Удалить</button>
      </nav>
    </main>
  );
}
```

### 2. ARIA атрибуты

```tsx
// ✅ Добавляй ARIA атрибуты
function Modal({ isOpen, onClose, children }: ModalProps) {
  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      className={isOpen ? 'modal open' : 'modal'}
    >
      <div className="modal-content">
        <header>
          <h2 id="modal-title">Заголовок модального окна</h2>
          <button
            aria-label="Закрыть модальное окно"
            onClick={onClose}
          >
            ×
          </button>
        </header>
        {children}
      </div>
    </div>
  );
}
```

## Навигация

- **[Общие принципы](../general/README.md)** — процессы разработки, документация
- **[Бэкенд](../backend/README.md)** — PHP, Laravel
- **[Базы данных](../database/README.md)** — MongoDB, MySQL


