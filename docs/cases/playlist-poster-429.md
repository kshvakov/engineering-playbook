# Кейс: 429 Too Many Requests на странице плейлиста

Инженер скинул в чат Developers ссылку на страницу плейлиста со скриншотом: десятки запросов на постеры возвращают 429. За одну сессию AI прочитал сообщение из мессенджера через MCP, проанализировал код двух проектов, нашёл корневую причину (отсутствие виртуализации списка), создал задачу в Kaiten, реализовал фикс и запушил ветку -- от сообщения до готовой ветки без переключения из IDE.

Дата: 26 марта 2026.

## Ситуация

Сообщение в канале Developers (Пачка):

> На странице плейлиста, когда много видео, то вот в такой лимит упираемся.
> https://kinescope.io/pl/m99UeXDqsHBMGsVVxjD5on

Скриншот из DevTools: 20+ одновременных GET-запросов на `poster/sm.jpg` -- все возвращают **429 (Too Many Requests)**. Источник запросов -- `kinescope-page-play.js:25`.

CTO попросил AI разобраться: «я думаю проблема не в лимитах, а в том, что в playlist нет lazy load».

## Что сделал AI

### Чтение контекста из Пачки

AI вызвал MCP-tool `fetch_pachka_channel` (Jarvis) с `chat_id: 32137018` и получил последние сообщения канала Developers, включая сообщение Владислава Дружинина со ссылкой на плейлист и скриншотом.

### Анализ скриншота

На скриншоте -- массив GET-запросов вида `https://kinescope.io/{video_id}/poster/sm.jpg`, все с ответом 429. Инициатор -- `kinescope-page-play.js:25`. Все запросы уходят одновременно при открытии страницы.

### Исследование плеера (player) -- ложный след

Первая гипотеза: проблема в коде плеера. AI исследовал репозиторий `player` и обнаружил, что **в плеере всё сделано правильно**:

- Список плейлиста в меню плеера использует **react-virtuoso** (`Virtuoso`) -- элементы вне viewport не монтируются в DOM.
- Компонент `TrackPoster` по умолчанию использует `posterLoading = 'lazy'` -- нативная ленивая загрузка изображений.
- `increaseViewportBy={50}` задаёт буфер для предзагрузки.

```tsx
// packages/player/src/components/ControlBar/PlaylistMenu/PlaylistMenuList.tsx
const VirtualList = styled(Virtuoso<MenuItem<PlaylistItems[number]>, Context>)({
  minHeight: 1,
  '& > [data-viewport-type]': {
    position: 'relative !important',
  },
});
```

### Локализация проблемы

Файл `kinescope-page-play.js` со скриншота -- не плеер, а билд отдельного проекта. AI нашёл его через vite.config.ts:

```typescript
// play/preview-frontend/vite.config.ts
rollupOptions: {
  output: {
    entryFileNames: 'kinescope-page-play.js',
  },
},
```

Проект `preview-frontend` -- страница предпросмотра плейлиста (`kinescope.io/pl/...`), отдельный от плеера.

### Анализ кода preview-frontend

В компоненте `Playlist` все элементы рендерятся через `.map()`:

```tsx
// preview-frontend/src/components/playlist/index.tsx (до фикса)
const body = useMemo(() => {
  return playlist.map((item, index) => {
    return (
      <PlaylistItem
        key={item.id}
        activeID={activeID}
        index={index + 1}
        item={item}
        onCopy={handleCopy}
      />
    );
  });
}, [activeID]);
```

Каждый `PlaylistItem` содержит компонент `Poster` с `<img>` или `<picture>`. Формально `loading="lazy"` стоит на всех изображениях:

```tsx
// preview-frontend/src/components/playlist/components/poster/index.tsx
export default function Poster({poster, src}: Props) {
  if (typeof poster === 'string') {
    return <img src={poster} alt={''} loading="lazy" />;
  }
  // ...
  return (
    <picture>
      {poster.src?.srcset?.map((src, index) => (
        <source key={index} type={src.type} srcSet={src.src} media={src.media} />
      ))}
      {poster.src?.src && <img src={poster.src.src} alt={''} loading="lazy" />}
    </picture>
  );
}
```

### Почему `loading="lazy"` не помогает

`loading="lazy"` работает относительно **viewport документа**, а не скроллируемого контейнера. Все `<img>` находятся внутри `DesktopBody` с `overflow: auto` и `max-height`, но сам контейнер целиком расположен в viewport страницы. Браузер считает все изображения «видимыми» и загружает их одновременно.

Для `<picture>` с `<source srcSet>` ситуация ещё хуже: браузер может начать preload `<source>` до того, как `loading="lazy"` на вложенном `<img>` вступит в силу.

При 20+ видео в плейлисте это приводит к 20+ одновременным запросам на постеры -- CDN отвечает 429.

### Создание задачи в Kaiten

AI вызвал MCP-tool `create_card` (Jarvis):

- Доска: Development > Frontend > In progress
- Заголовок: «[Frontend] Виртуализация списка плейлиста на странице preview (429 Too Many Requests)»
- Описание: проблема, корневая причина, решение
- Назначено: CTO (инициатор)

Результат: карточка **#62524946**.

### Создание ветки

```bash
cd play/preview-frontend
git fetch origin main && git checkout main && git pull origin main
git checkout -b fix/playlist-virtualization-kaiten-62524946
```

Суффикс `kaiten-62524946` привязывает ветку к карточке.

### Реализация фикса

Три файла:

**1. `package.json`** -- добавлена зависимость:

```json
"react-virtuoso": "^4.18.3"
```

**2. `src/components/playlist/index.tsx`** -- замена `.map()` на `Virtuoso`:

```tsx
import {Virtuoso, VirtuosoHandle} from 'react-virtuoso';

const renderItem = useCallback(
  (index: number, item: PlaylistTypes) => (
    <PlaylistItem
      activeID={activeID}
      index={index + 1}
      item={item}
      onCopy={handleCopy}
    />
  ),
  [activeID],
);

const initialIndex = useMemo(() => {
  const idx = playlist.findIndex(play => play.id === activeID);
  return idx > -1 ? idx : 0;
}, [playlist, activeID]);

const body =
  !activeID || activeID === '' ? null : (
    <Virtuoso
      ref={virtuosoRef}
      data={playlist}
      itemContent={renderItem}
      increaseViewportBy={200}
      initialTopMostItemIndex={{index: initialIndex, align: 'center'}}
      style={{height: '100%'}}
    />
  );
```

`DesktopBody` переведён с `display: block` + `overflow: auto` на `display: flex` + `flex-direction: column` -- Virtuoso сам управляет скроллом.

Бонус: `initialTopMostItemIndex` автоматически скроллит к текущему треку при открытии -- раньше этого не было.

**3. `src/components/drawer/index.tsx`** -- `DrawerContent` дополнен для корректной работы Virtuoso в мобильном drawer:

```css
display: flex;
flex-direction: column;
min-height: 0;
```

`min-height: 0` критически важен для flex-контейнеров, чтобы child с `height: 100%` не выходил за пределы.

### Коммит и push

```bash
git add package.json src/components/drawer/index.tsx src/components/playlist/index.tsx
git commit -m "Виртуализация списка плейлиста через react-virtuoso

Заменяет playlist.map() на Virtuoso для ленивого рендеринга элементов.
Элементы вне видимой области не монтируются в DOM, что устраняет
массовую одновременную загрузку постеров и ошибки 429 Too Many Requests
при большом количестве видео в плейлисте.

KAITEN-62524946"

git push -u origin fix/playlist-virtualization-kaiten-62524946
```

## Таймлайн

От ссылки в Cursor до запушенной ветки с фиксом -- **22 минуты**.

| Время | Шаг | Что произошло |
|-------|-----|---------------|
| 14:01 | 1 | Сообщение Владислава в Пачке со скриншотом 429 |
| 14:25 | 2 | CTO вставил ссылку на сообщение в Cursor: «проблема не в лимитах, а в том, что нет lazy load» |
| 14:26 | 3 | AI прочитал канал Developers через MCP (`fetch_pachka_channel`), проанализировал скриншот |
| 14:28 | 4 | Исследование плеера (player) -- обнаружено, что там уже есть Virtuoso + `loading="lazy"` |
| 14:30 | 5 | Локализация: `kinescope-page-play.js` -- билд preview-frontend, не плеера |
| 14:32 | 6 | Анализ preview-frontend: `.map()` без виртуализации, `loading="lazy"` не спасает в scrollable container |
| 14:34 | 7 | Объяснение корневой причины и трёх вариантов решения; CTO выбрал виртуализацию |
| 14:37 | 8 | Создание задачи #62524946 в Kaiten через MCP (`create_card`) |
| 14:38 | 9 | Создание ветки `fix/playlist-virtualization-kaiten-62524946` от main |
| 14:45 | 10 | Реализация: react-virtuoso + flex layout для desktop и mobile (3 файла) |
| 14:47 | 11 | Коммит и push ветки в origin |

## Результат

Инженер получил за одну сессию:

1. **Диагностику** -- корневая причина не в лимитах CDN, а в рендеринге всех элементов плейлиста одновременно.
2. **Кросс-проектный анализ** -- подтверждение, что в плеере проблемы нет (Virtuoso + lazy), и точная локализация в preview-frontend.
3. **Задачу в Kaiten** (#62524946) с описанием проблемы и решения.
4. **Готовую ветку** с фиксом, привязанную к задаче, с ссылкой на MR.

## Чему учит этот кейс

**Проблема не всегда там, где кажется.** Сообщение про «лимит» (429 Too Many Requests) наводит на мысль о rate limiting CDN. Корневая причина -- отсутствие виртуализации: браузер отправляет десятки запросов одновременно, потому что все `<img>` смонтированы в DOM. Фикс на стороне CDN (увеличение лимитов) замаскировал бы проблему, но не устранил.

**`loading="lazy"` не работает в scrollable containers.** Нативный `loading="lazy"` привязан к viewport документа. Если все изображения находятся внутри контейнера с `overflow: auto`, но сам контейнер в viewport -- браузер загрузит всё сразу. Виртуализация (react-virtuoso, @tanstack/react-virtual) -- надёжный способ контролировать, что монтируется в DOM.

**Кросс-проектный анализ через кодовую базу.** AI проверил реализацию в плеере, обнаружил паттерн (Virtuoso + lazy loading), и перенёс его в preview-frontend. Без доступа к обоим проектам пришлось бы изобретать решение заново.

**MCP замыкает цикл в одной сессии.** Пачка (контекст проблемы) -> анализ кода -> Kaiten (задача) -> Git (ветка + коммит + push) -- всё через MCP-интеграции, без переключения между браузером, трекером и терминалом.

**Паттерн уже существовал в кодовой базе.** Плеер использовал react-virtuoso для того же списка плейлиста. Решение -- не изобретение нового подхода, а перенос проверенного паттерна из одного проекта в другой. AI нашёл этот паттерн при анализе плеера и предложил его как решение.

## Использованные инструменты

| Инструмент | Что делал | Этап |
|------------|-----------|------|
| MCP: `fetch_pachka_channel` (Jarvis) | Прочитал сообщение из канала Developers | Контекст |
| MCP: `discover_workspace` (Jarvis) | Получил структуру досок Kaiten | Создание задачи |
| MCP: `create_card` (Jarvis) | Создал карточку #62524946 | Создание задачи |
| Скилл: executing-plans | Пакетное выполнение шагов плана | Реализация |
| Git + SSH | Ветка, коммит, push в preview-frontend | Реализация |
| Анализ кода (player) | Обнаружение паттерна Virtuoso + lazy | Диагностика |
| Анализ кода (preview-frontend) | Локализация `.map()` без виртуализации | Диагностика |
