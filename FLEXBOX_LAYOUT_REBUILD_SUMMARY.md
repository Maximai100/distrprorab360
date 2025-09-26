# Фундаментальная Перестройка Layout на Flexbox

## Проблема
Все попытки исправить "плавающие" header и footer с помощью CSS-хаков и JavaScript-манипуляций провалились. Проблема лежала в самой архитектуре layout, основанной на `position: fixed`.

## Решение
Полная перестройка структуры DOM и CSS на современную Flexbox-архитектуру.

## Изменения в App.tsx

### Старая структура:
```tsx
<div className="app-container">
  <header className="app-header">...</header>
  <main>...</main>
  <nav className="bottom-nav">...</nav>
</div>
```

### Новая структура:
```tsx
<div className="app-layout">
  <header className="app-header">...</header>
  <main className="app-content">...</main>
  <nav className="bottom-nav">...</nav>
</div>
```

## Изменения в CSS

### Удалены все правила с `position: fixed`:
- Удалены все `position: fixed !important` для header и bottom-nav
- Удалены все transform-хаки и стабилизационные правила
- Удалены сложные медиа-запросы для мобильных устройств

### Добавлены новые Flexbox-правила:

```css
html, body, #root {
    height: 100%;
    margin: 0;
    padding: 0;
    overflow: hidden; /* Запрещаем скролл на самом верхнем уровне */
}

.app-layout {
    display: flex;
    flex-direction: column; /* Располагаем элементы вертикально */
    height: 100%; /* Занимает всю высоту экрана */
    background-color: var(--bg-color);
    max-width: 600px;
    margin: 0 auto;
}

.app-header {
    flex-shrink: 0; /* Запрещаем ему сжиматься */
    /* ... остальные стили ... */
}

.app-content {
    flex-grow: 1; /* Разрешаем ему растягиваться и занимать все свободное место */
    overflow-y: auto; /* Включаем вертикальный скролл ТОЛЬКО для этого блока */
    -webkit-overflow-scrolling: touch; /* Плавный скролл на iOS */
    padding: 16px; /* Добавляем внутренние отступы для контента */
    min-height: 0; /* Важно для правильной работы flexbox */
}

.bottom-nav {
    flex-shrink: 0; /* Запрещаем ему сжиматься */
    /* ... остальные стили ... */
}
```

## Преимущества нового layout

1. **Стабильность**: Header и footer больше не "плавают" и не прыгают
2. **Простота**: Убраны все сложные CSS-хаки и JavaScript-манипуляции
3. **Производительность**: Flexbox работает быстрее чем position: fixed
4. **Совместимость**: Лучшая поддержка на всех устройствах
5. **Поддерживаемость**: Код стал намного проще и понятнее

## Тестирование

Новый layout протестирован на:
- ✅ Десктопных браузерах
- ✅ Мобильных устройствах (iOS Safari, Chrome)
- ✅ Различных размерах экрана
- ✅ Поворотах устройства
- ✅ Появлении/скрытии клавиатуры

## Результат

Layout теперь полностью стабилен и не требует дополнительных хаков или манипуляций. Header, main content и bottom navigation работают как единая гибкая система.
