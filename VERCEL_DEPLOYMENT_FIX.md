# 🚀 VERCEL DEPLOYMENT FIX - Исправление проблем деплоя

## 🚨 Проблема
После деплоя в Vercel возникает ошибка аутентификации:
- `AuthRetryableFetchError`
- `HTTP 504 Gateway Timeout`
- `Session: null`
- Используется демо-ключ Supabase

## 🔧 Решение

### 1. Настройка переменных окружения в Vercel

1. **Откройте Vercel Dashboard**:
   - Перейдите в проект `distrprorab360`
   - Откройте вкладку **Settings** → **Environment Variables**

2. **Добавьте переменные окружения**:
   ```
   VITE_SUPABASE_URL = https://prorab360.online
   VITE_SUPABASE_ANON_KEY = ваш_реальный_anon_ключ_из_supabase
   ```

3. **Получите реальный ключ Supabase**:
   - Откройте [Supabase Dashboard](https://supabase.com/dashboard)
   - Выберите проект `prorab360`
   - Перейдите в **Settings** → **API**
   - Скопируйте **anon public** ключ

### 2. Обновление конфигурации Supabase

Текущая проблема в `src/supabaseClient.ts`:
```typescript
// ❌ ПРОБЛЕМА: Используется демо-ключ по умолчанию
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY || 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
```

### 3. Проверка Supabase проекта

Убедитесь, что:
- ✅ Проект `prorab360` активен в Supabase
- ✅ База данных доступна
- ✅ API ключи корректны
- ✅ RLS политики настроены

### 4. Передеплой

После настройки переменных окружения:
1. **Redeploy** проект в Vercel
2. Проверьте логи деплоя
3. Тестируйте аутентификацию

## 🔍 Диагностика

### Проверка переменных окружения
```javascript
// В консоли браузера на продакшене
console.log('VITE_SUPABASE_URL:', import.meta.env.VITE_SUPABASE_URL)
console.log('VITE_SUPABASE_ANON_KEY:', import.meta.env.VITE_SUPABASE_ANON_KEY?.substring(0, 20) + '...')
```

### Проверка Supabase подключения
```javascript
// В консоли браузера
window.supabase = supabase
console.log('Supabase URL:', supabase.supabaseUrl)
console.log('Supabase Key:', supabase.supabaseKey?.substring(0, 20) + '...')
```

## 📋 Чек-лист исправления

- [ ] Добавить `VITE_SUPABASE_URL` в Vercel Environment Variables
- [ ] Добавить `VITE_SUPABASE_ANON_KEY` в Vercel Environment Variables
- [ ] Получить реальный anon ключ из Supabase Dashboard
- [ ] Убедиться, что Supabase проект активен
- [ ] Передеплоить проект в Vercel
- [ ] Протестировать аутентификацию

## 🎯 Ожидаемый результат

После исправления:
- ✅ Аутентификация работает корректно
- ✅ Нет ошибок `AuthRetryableFetchError`
- ✅ Сессия устанавливается правильно
- ✅ Приложение работает в продакшене

## 📞 Поддержка

Если проблемы остаются:
1. Проверьте логи Vercel деплоя
2. Проверьте статус Supabase проекта
3. Убедитесь в корректности API ключей
4. Проверьте CORS настройки в Supabase
