# Bookmarks API (TypeScript) — Критерии готовности

## 1. Настройка проекта
- [ ] Проект запускается одной командой (`npm run dev`)
- [ ] Зависимости зафиксированы в `package.json` + `package-lock.json`
- [ ] Используется SQLite (для честного сравнения с Python-версией)
- [ ] TypeScript strict mode включён в `tsconfig.json`
- [ ] При первом запуске база создаётся через `npx prisma migrate dev`

## 2. Prisma-схема
- [ ] Модель `Bookmark` — id, url, title, description (nullable), createdAt
- [ ] Модель `Tag` — id, name (уникальное)
- [ ] Связь many-to-many через implicit relation (`tags Tag[]` / `bookmarks Bookmark[]`)
- [ ] `prisma generate` создаёт типизированный клиент без ошибок
- [ ] Миграция применяется чисто

## 3. Структура API (Next.js App Router)
- [ ] `app/api/bookmarks/route.ts` — GET (список) и POST (создание)
- [ ] `app/api/bookmarks/[id]/route.ts` — GET, PUT, DELETE для конкретной закладки
- [ ] `app/api/bookmarks/search/route.ts` — GET поиск
- [ ] `app/api/tags/route.ts` — GET список тегов
- [ ] Каждый route handler использует типизированные request/response

## 4. CRUD закладок
- [ ] `POST /api/bookmarks` — создаёт закладку, принимает `{ url, title, description?, tags?: string[] }`
- [ ] Если переданный тег не существует — он создаётся через `connectOrCreate`
- [ ] Дублирующийся URL возвращает `409 Conflict`
- [ ] `GET /api/bookmarks` — возвращает список закладок с вложенными тегами (`include: { tags: true }`)
- [ ] `GET /api/bookmarks/[id]` — возвращает одну закладку; `404` если не найдена
- [ ] `PUT /api/bookmarks/[id]` — обновляет закладку (включая пересвязку тегов через `set` + `connectOrCreate`)
- [ ] `DELETE /api/bookmarks/[id]` — удаляет закладку; `404` если не найдена

## 5. Фильтрация и пагинация
- [ ] `GET /api/bookmarks?tag=python` — фильтрация через `where: { tags: { some: { name } } }`
- [ ] `GET /api/bookmarks?limit=10&offset=0` — пагинация через `take` / `skip`
- [ ] `limit` по умолчанию = 20, `offset` по умолчанию = 0
- [ ] Ответ содержит `total` (через отдельный `prisma.bookmark.count()`)
- [ ] Сортировка по `createdAt` от новых к старым (`orderBy: { createdAt: 'desc' }`)

## 6. Поиск
- [ ] `GET /api/bookmarks/search?q=текст` — ищет по `title` и `description`
- [ ] Поиск через `contains` с `mode: 'insensitive'` (регистронезависимый)
- [ ] Пустой запрос `q` возвращает `400 Bad Request`
- [ ] Результаты поиска тоже поддерживают пагинацию

## 7. Теги
- [ ] `GET /api/tags` — возвращает список тегов с полем `bookmarksCount`
- [ ] Используется `include: { _count: { select: { bookmarks: true } } }`
- [ ] Теги без закладок тоже отображаются (с `bookmarksCount: 0`)
- [ ] Имена тегов нормализуются к нижнему регистру при создании

## 8. Валидация (Zod)
- [ ] Установлен и используется `zod` для валидации входных данных
- [ ] `url` проверяется через `z.string().url()`
- [ ] `title` не может быть пустой строкой — `z.string().min(1)`
- [ ] Некорректные данные возвращают `400` с понятным сообщением из Zod
- [ ] Схемы валидации вынесены в отдельный файл (`lib/schemas.ts` или аналог)

## 9. Формат ответов
- [ ] Все ответы в JSON через `NextResponse.json()`
- [ ] Даты в формате ISO 8601 (Prisma отдаёт по умолчанию)
- [ ] Ответ на создание возвращает `201` — `NextResponse.json(data, { status: 201 })`
- [ ] Ответ на удаление возвращает `204` — `new NextResponse(null, { status: 204 })`
- [ ] Структура ответа списка: `{ items: [...], total: N, limit: N, offset: N }`

## 10. Типизация
- [ ] Типы ответов определены (можно через `Prisma.BookmarkGetPayload` или свои интерфейсы)
- [ ] Нет `any` в коде (или минимум с обоснованием)
- [ ] Query parameters парсятся типобезопасно
- [ ] `prisma client` полностью типизирован (автогенерация)

## 11. Финальная проверка (идентична Python-версии)
- [ ] Создать 5+ закладок с разными тегами
- [ ] Проверить фильтрацию по тегу — возвращает только нужные
- [ ] Проверить пагинацию — limit/offset работают корректно
- [ ] Проверить поиск — находит по части слова в title и description
- [ ] Удалить закладку — проверить что теги не удалились
- [ ] Попробовать создать закладку с дублирующимся URL — получить 409
- [ ] Попробовать получить несуществующую закладку — получить 404

## 12. Сравнение с Python-версией (записать после завершения)
- [ ] Время на реализацию: Python ___ ч / TypeScript ___ ч
- [ ] Количество файлов: Python ___ / TypeScript ___
- [ ] Строк кода (без конфигов): Python ___ / TypeScript ___
- [ ] Где LLM помог больше?
- [ ] Где пришлось больше гуглить?
- [ ] Что понравилось / не понравилось в каждом стеке?
