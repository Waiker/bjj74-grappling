# Изменения (сессия 2026-05-24)

## Состояние «было → стало»

### Медиа
- **было:** 3 файла в `static/images/` (1 hero + svg + webp того же hero).
- **стало:** 48 файлов с Тильды в `static/images/tilda/` + 5 переименованных в семантические имена в `static/images/`:
  - `hero-grappling-real.jpg` — для нового hero
  - `sparring-bw.jpg` — B&W арт-фото
  - `gym-group.jpg` — реальная групповая фотка в зале (4 атлета, татами жёлто-синий)
  - `competition.png` — фото с турнира
  - `map-privilegiya.png` — скриншот карты с пином «Академия спорта»
- Каталог в `IMAGES.md`. Видно: реальных «школьных» фото мало (4 шт), большая часть — celebrity-блог-обложки и stock.

### Tracking
- **было:** ни VK pixel, ни Метрики.
- **стало:** `layouts/partials/analytics.html` с обоими счётчиками (Метрика `103922438`, VK Pixel `VK-RTRG-1964966-b8RM4`), подключён в `baseof.html`. Эмиссия только в production-сборке (`hugo.IsProduction`), чтобы не загрязнять локалкой.

### Ahrefs SEO/GEO ресёрч
- **было:** codex признался, что Ahrefs не использовался.
- **стало:** `AHREFS_BRIEF.md`. Ключевой вывод: рынок ультра-локальный (грэпплинг челябинск = 20/мес, KD=0; джиу джитсу челябинск = 50/мес; bjj74.ru сейчас 2 трафика/мес, kimurabjj.ru тоже 0 keywords). SEO — поддержка, не главный канал. Использовано ~3.4k units из 400k лимита.

### Шорткоды
- **обновлены:** `hero` (теперь принимает params), `factgrid` (override 4 items).
- **созданы:** `factoid` (sidebar quote с цифрой), `coursebanner` (баннер 6-нед курса), `photo` (figure+caption).
- **CSS:** добавлены стили `.factoid`, `.course-banner`, `.figure`.

### Контент
**Переписаны под vault-позиционирование («умный контактный фитнес 18+ без ударов в голову», «после офиса на ковёр», «6-недельный курс»):**
- `content/_index.md` — главная: новый hero с реальным фото, factgrid, factoid, 6-week banner, фото зала, локация.
- `content/grappling-chelyabinsk.md` — взрослый позиционинг, сравнение vs бокс/самбо/ММА, расписание.
- `content/grappling-for-women.md` — переделан под реальные страхи женского старта, отдельная стартовая группа, самооборона без агрессии.

**Созданы новые:**
- `content/start-course-6-weeks.md` — флагман новой воронки: расписана программа по неделям, FAQ, что входит/не входит.
- `content/blog/what-is-grappling.md` — новый SEO-блог под ключи «что такое грэпплинг» (300/мес), «грэпплинг что это» (200/мес), «грэпплинг» (800/мес). Алиас `/grappling-what-is-it`.

### Навигация
- **меню:** добавлен «Курс с нуля» (вес 5, первая позиция). Убран дубль «Новичкам» (есть `/novichkam/`, но в новой воронке курс важнее).
- **header CTA:** «Записаться» → «Курс с нуля» (ведёт на `/start-course-6-weeks/`).

### Hugo config
- `hugo.yaml`: комментарий к `reviews.*` — карточки Я.Бизнес/2ГИС НЕ заведены, текущие URL ведут на поиск как fallback.

### Документы для следующих сессий
- `AHREFS_BRIEF.md` — SEO/GEO brief, целевые ключи по приоритетам, GEO задачи.
- `IMAGES.md` — каталог скачанных фото + рекомендации по использованию.
- `IMAGE_PROMPTS.md` — 19 промптов для генерации недостающих изображений (hero для landing-страниц, OG картинки, иконки), brand-правила.

## Что НЕ сделано в этой сессии (осознанно — для checkpoint'а)

- Не переписан контент 11 оставшихся landing-страниц (`bjj-nogi-chelyabinsk`, `grappling-for-men-35-plus`, `self-defense-for-women-chelyabinsk`, `martial-arts-for-adults-chelyabinsk`, `privilegiya`, `first-training`, `trainer`, `schedule`, `prices`, `contacts`, `reviews`, `faq`, `novichkam`).
- Не переписаны 16 блог-постов (они смигрированы с Тильды, текст уже норм; нужен только cover image + проход по SEO-ключам).
- Не добавлены OG-картинки для каждой страницы (нужны через `IMAGE_PROMPTS.md`).
- Не сделаны JSON-LD BreadcrumbList и Article schema для блога (Article — codex уже сделал, BreadcrumbList — нет).
- Не заведены карточки Я.Бизнес / 2ГИС (это offline-задача, требует БД и фото).
- Не сделана фотосессия в зале (главное узкое место сайта).

## Бюджет Ahrefs
Использовано **~3400 units из 400 000** (~0.85%). Можно ещё много жечь в следующих сессиях.

## Hugo build status
`/tmp/hugo-0.160.1/hugo --environment production` — успешно, без ошибок. Главная 15.4 KB. Analytics в production-сборке присутствует.
