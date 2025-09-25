# 🛡️ SUPABASE SAFETY GUIDE - Инструкция по безопасным изменениям

## ⚠️ КРИТИЧЕСКИ ВАЖНО: Читать перед каждым изменением кода!

### 🚨 Основные правила безопасности:

#### 1. **НИКОГДА не используй `Promise.all` для Supabase запросов**
```typescript
// ❌ ОПАСНО - ломает все при одной ошибке
const [res1, res2] = await Promise.all([
    supabase.from('table1').select('*'),
    supabase.from('table2').select('*')
]);

// ✅ БЕЗОПАСНО - обрабатывает ошибки независимо
const [res1, res2] = await Promise.allSettled([
    supabase.from('table1').select('*'),
    supabase.from('table2').select('*')
]);

// Обработка результатов:
const data1 = res1.status === 'fulfilled' ? res1.value : null;
const data2 = res2.status === 'fulfilled' ? res2.value : null;

if (res1.status === 'rejected') {
    console.error('Ошибка запроса 1:', res1.reason);
}
if (res2.status === 'rejected') {
    console.error('Ошибка запроса 2:', res2.reason);
}
```

#### 2. **ВСЕГДА добавляй задержки между запросами**
```typescript
// ✅ БЕЗОПАСНО - добавляй задержку
await new Promise(resolve => setTimeout(resolve, 50));

const { data, error } = await supabase.from('table').select('*');
```

#### 3. **ВСЕГДА оборачивай Supabase запросы в try-catch**
```typescript
// ✅ БЕЗОПАСНО
try {
    const { data, error } = await supabase.from('table').select('*');
    if (error) {
        console.error('Ошибка:', error);
        return; // Не выбрасывай ошибку
    }
    // Обработка данных
} catch (error) {
    console.error('Критическая ошибка:', error);
    // Показывай пустые данные вместо краха
    setData([]);
}
```

#### 4. **НИКОГДА не добавляй избыточное логирование**
```typescript
// ❌ ОПАСНО - слишком много логов
console.log('[DEBUG] Шаг 1:', data);
console.log('[DEBUG] Шаг 2:', data);
console.log('[DEBUG] Шаг 3:', data);

// ✅ БЕЗОПАСНО - только критичные логи
console.log('Загружаем данные:', projectId);
if (error) console.error('Ошибка:', error);
```

#### 5. **ВСЕГДА соблюдай порядок объявления переменных в React**
```typescript
// ❌ ОПАСНО - использование до объявления
console.log('Профиль:', companyProfileHook.profile); // ОШИБКА!
const companyProfileHook = useCompanyProfile(session);

// ✅ БЕЗОПАСНО - сначала объявление, потом использование
const companyProfileHook = useCompanyProfile(session);
console.log('Профиль:', companyProfileHook.profile);
```

#### 6. **ВСЕГДА размещай логирование ПОСЛЕ объявления всех хуков**
```typescript
// ❌ ОПАСНО - логирование до объявления хуков
console.log('🚀 App: companyProfile:', companyProfileHook.profile);
const companyProfileHook = useCompanyProfile(session);

// ✅ БЕЗОПАСНО - логирование после объявления всех хуков
const companyProfileHook = useCompanyProfile(session);
const otherHook = useOtherHook();
// ... все остальные хуки

// Только теперь логирование
console.log('🚀 App: companyProfile:', companyProfileHook.profile);
```

### 🔧 Шаблоны безопасных изменений:

#### **Безопасное изменение useEffect:**
```typescript
useEffect(() => {
    const loadData = async () => {
        if (!session?.user) return;
        
        try {
            // Задержка для предотвращения спама
            await new Promise(resolve => setTimeout(resolve, 50));
            
            const { data, error } = await supabase
                .from('table')
                .select('*')
                .eq('user_id', session.user.id);
            
            if (error) {
                console.error('Ошибка загрузки:', error);
                return; // Не выбрасывай ошибку
            }
            
            setData(data || []);
        } catch (error) {
            console.error('Критическая ошибка:', error);
            setData([]); // Показывай пустые данные
        }
    };

    loadData();
}, [session?.user]); // Минимальные зависимости
```

#### **Безопасное изменение функции загрузки:**
```typescript
const loadData = useCallback(async (id: string, retryCount = 0) => {
    if (!id) return;
    
    setLoading(true);
    setError(null);
    
    try {
        // Задержка с учетом retry
        await new Promise(resolve => setTimeout(resolve, 100 * (retryCount + 1)));
        
        const { data, error } = await supabase
            .from('table')
            .select('*')
            .eq('id', id);
        
        if (error) throw error;
        
        setData(data);
    } catch (error) {
        // Retry логика для сетевых ошибок
        if (retryCount < 2 && error instanceof Error && 
            (error.message.includes('NetworkError') || error.message.includes('fetch'))) {
            setTimeout(() => {
                loadData(id, retryCount + 1);
            }, 1000 * (retryCount + 1));
            return;
        }
        
        // Обработка ошибок
        if (error instanceof Error) {
            if (error.message.includes('NetworkError')) {
                setError('Ошибка сети. Проверьте подключение.');
            } else {
                setError(`Ошибка: ${error.message}`);
            }
        } else {
            setError('Неизвестная ошибка');
        }
    } finally {
        setLoading(false);
    }
}, []);
```

#### **Безопасная структура React компонента:**
```typescript
const MyComponent = () => {
    // 1. СНАЧАЛА все хуки (useState, useEffect, useCallback, etc.)
    const [data, setData] = useState([]);
    const [loading, setLoading] = useState(false);
    
    const appState = useAppState();
    const estimatesHook = useEstimates(session);
    const companyProfileHook = useCompanyProfile(session);
    // ... все остальные хуки
    
    // 2. ПОТОМ refs
    const inputRef = useRef<HTMLInputElement>(null);
    
    // 3. ПОТОМ логирование (только после объявления всех хуков!)
    console.log('🚀 Component: data:', data);
    console.log('🚀 Component: profile:', companyProfileHook.profile);
    
    // 4. ПОТОМ функции и обработчики
    const handleSubmit = useCallback(() => {
        // логика
    }, []);
    
    // 5. ПОТОМ useEffect
    useEffect(() => {
        // логика
    }, []);
    
    // 6. В КОНЦЕ рендер
    return <div>...</div>;
};
```

### 🚫 Что НИКОГДА не делать:

1. **Не изменять массивы зависимостей useEffect без необходимости**
2. **Не добавлять логирование в критические пути выполнения**
3. **Не использовать Promise.all для Supabase запросов**
4. **Не выбрасывать ошибки без обработки**
5. **Не делать множественные изменения одновременно**
6. **Не изменять порядок выполнения хуков**
7. **Не использовать переменные до их объявления**
8. **Не размещать логирование до объявления всех хуков**
9. **Не нарушать порядок объявления переменных в React компонентах**

### ✅ Чек-лист перед каждым изменением:

- [ ] Я использую `Promise.allSettled` вместо `Promise.all`?
- [ ] Я добавил задержки между запросами?
- [ ] Я обернул все Supabase запросы в try-catch?
- [ ] Я не добавляю избыточное логирование?
- [ ] Я не изменяю критические зависимости useEffect?
- [ ] Я тестирую изменения поэтапно?
- [ ] Я могу откатить изменения если что-то сломается?
- [ ] Я объявил все переменные перед их использованием?
- [ ] Я разместил логирование после объявления всех хуков?
- [ ] Я соблюдаю правильный порядок объявления переменных?

### 🆘 Экстренные меры при поломке:

1. **Немедленно откатить последние изменения**
2. **Проверить консоль на ошибки NetworkError**
3. **Убедиться что все Promise.all заменены на Promise.allSettled**
4. **Добавить задержки в проблемные места**
5. **Убрать избыточное логирование**
6. **Проверить порядок объявления переменных**
7. **Убедиться что логирование размещено после объявления хуков**
8. **Проверить что все переменные объявлены перед использованием**

### 📝 История проблем и решений:

**Проблема:** `NetworkError when attempting to fetch resource`
**Причина:** Promise.all ломал все запросы при одной ошибке
**Решение:** Замена на Promise.allSettled + retry логика + задержки

**Проблема:** Слишком частые запросы к Supabase
**Причина:** Отсутствие задержек между запросами
**Решение:** Добавление setTimeout перед каждым запросом

**Проблема:** Приложение ломается при ошибках загрузки
**Причина:** Необработанные исключения
**Решение:** Graceful error handling с показом пустых данных

**Проблема:** `can't access lexical declaration 'companyProfileHook' before initialization`
**Причина:** Использование переменной до её объявления в React компоненте
**Решение:** Перемещение логирования после объявления всех хуков

---

## 🎯 ЗАПОМНИ: Лучше сделать меньше изменений, но безопасно, чем много изменений, которые все сломают!

