# SCSS Architecture Rules

Цели:

- избежать свалки стилей
- сохранить масштабируемость
- четко разделять ответственность

Если сомневаешься, куда писать стиль — смотри сюда.

---

## Entry point

Единственная точка входа — `src/scss/main.scss`.
Только он собирает все слои через `@use`.
Никаких `@use "./base"` или `@use "bootstrap/scss/bootstrap"` внутри компонентов.

Пример:

```scss
@use './abstracts';
@use './base';
@use './layout';
@use './components';
```

---

## Module system

Используем только `@use` и `@forward`.
`@import` запрещен.

Правило индексов:

- каждая папка имеет `_index.scss`
- внутри только `@forward`
- `main.scss` подключает только папки

---

## Папки и ответственность

### abstracts/

Только вспомогательная логика. Никаких селекторов.
Разрешено: variables, mixins, functions, tokens.

### base/

Глобальные стили, которые применяются ко всему проекту:
reset, body, html, типографика.
Тут же живет Bootstrap bridge.

### layout/

Каркас и структура страницы:
header, footer, navigation, sections.
Без "декора" и стилей компонентов.

### components/

Переиспользуемые UI-компоненты:
button, card, form, mobile-menu и т.д.
Компоненты не знают о layout.

---

## Bootstrap integration

### Bridge

Bootstrap подключается только через bridge:
`src/scss/base/_bootstrap.scss`

Bridge читает переменные из `abstracts/_variables.scss` и передает их в Bootstrap:

```scss
@use '../abstracts/variables' as vars;
@use 'bootstrap/scss/bootstrap' with (
	$grid-breakpoints: vars.$grid-breakpoints,
	$container-max-widths: vars.$container-max-widths
);
```

Это гарантирует:

- единый источник правды для ваших карт
- отсутствие дублирования
- правильный порядок инициализации

### Tokens (легкий доступ)

Если в компоненте нужны цвета/токены Bootstrap, используем CSS variables:
`abstracts/_bs-tokens.scss`

Пример в компоненте:

```scss
@use '../abstracts/bs-tokens' as bs;

.button {
	color: bs.$bs-primary;
	border-color: bs.$bs-border-color;
}
```

Bootstrap SCSS в компоненты не подключаем.

---

## Rules (коротко)

❌ Компоненты не импортируют `base` и `bootstrap`.
❌ Layout не красит компоненты.
❌ `sections.scss` не превращается в помойку.

✅ Меню = layout.
✅ Кнопка = component.
✅ Fixed / sticky UI — отдельный слой (layout или component, по смыслу).

---

## Naming

- файл = одна ответственность
- имя по смыслу, не по библиотеке

❌ `slideout.scss`
✅ `navigation.scss` или `offcanvas.scss`

---

## Где писать

- мобильное меню: `layout/_navigation.scss`
- кнопка бургера: `components/_burger.scss`
