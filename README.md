# help-stankoman (Hugo Wiki)

Документация/база знаний для `help.stankoman.ru` на **Hugo** с темой **hugo-theme-techdoc**.

---

## Требования

- Hugo **>= 0.120.0**
- Git
- Тема подключена как **git submodule**

---

## Структура проекта

Исходники (редактируешь ты):

- `content/` — страницы в Markdown
- `layouts/` — переопределения темы (в т.ч. shortcodes)
- `layouts/shortcodes/` — shortcodes (`rutube.html` и т.д.)
- `static/` — статика «как есть» (картинки/файлы/видео, если хостишь у себя)
- `assets/` — “умные” ассеты (Hugo Pipes), обычно трогают при кастомизации фронта
- `themes/` — тема как submodule
- `hugo.toml` — конфигурация сайта

Сборка (генерится Hugo, **в git не хранить**):

- `public/` — готовый статик (HTML/CSS/JS), используется для публикации/предпросмотра

### Про `_index.md` и `index.md`

- `content/section/_index.md` — страница раздела (list page)
- `content/page/index.md` — “bundle” страница (удобно хранить медиа рядом)

Пример:

```

content/
_index.md
docs/
_index.md
install/
_index.md
windows.md
linux.md
howto/
rutube/
index.md
cover.png

````

---

## Локальная разработка (Windows 11)

### Установка Hugo Extended

**Рекомендовано (официально):**
```powershell
winget install Hugo.Hugo.Extended
````

Проверка:

```powershell
hugo version
```

В выводе должно быть слово `extended`.

### Клон репозитория (с сабмодулями)

Если клонируешь с нуля:

```powershell
git clone --recurse-submodules https://github.com/edgarkosul/help-stankoman.git
cd help-stankoman
```

Если уже клонировал без сабмодулей:

```powershell
git submodule update --init --recursive
```

### Предпросмотр локально

```powershell
cd C:\Users\edgar\projects\help-stankoman
hugo server -D
```

Открыть: `http://localhost:1313/`

---

## Ежедневный workflow (Win11 → GitHub → Server)

### 1) Правки локально (Win11)

* Редактируешь `content/**.md` (и при необходимости `layouts/`, `static/`, `hugo.toml`)
* Проверяешь через `hugo server -D`

Коммит + пуш:

```powershell
git add -A
git commit -m "Docs: update <что сделал>"
git push
```

### 2) Публикация на сервере (wds)

На сервере исходники лежат тут:

* `/var/www/help.stankoman.ru/src`  ← это “как локальная папка проекта”
  Сборка для nginx лежит тут:
* `/var/www/help.stankoman.ru/public` ← это root для nginx

После пуша на GitHub заходишь на сервер и запускаешь деплой:

```bash
ssh stankoman
deploy-help
```

---

## Деплой-скрипт на сервере (deploy-help)

Создать на сервере:

```bash
sudo tee /usr/local/bin/deploy-help >/dev/null <<'SH'
#!/usr/bin/env bash
set -euo pipefail

cd /var/www/help.stankoman.ru/src

git pull --recurse-submodules
git submodule update --init --recursive

hugo --minify -d /var/www/help.stankoman.ru/public

# если нужно, можно перезагрузить nginx:
# sudo systemctl reload nginx
SH

sudo chmod +x /usr/local/bin/deploy-help
```

---

## RuTube shortcode

Файл:

* `layouts/shortcodes/rutube.html`

Использование в Markdown:

```md
{{< rutube id="7163336" title="Скринкаст: оформление заказа" >}}
```

Со стартом (секунды):

```md
{{< rutube id="7163336" params="bmstart=45" >}}
```

---

## Видео на своём сервере (опционально)

1. Положи mp4 в:

* `static/videos/my-screencast.mp4` → будет доступно как `/videos/my-screencast.mp4`

2. Вставь в Markdown (HTML внутри MD):

```html
<video controls preload="metadata" playsinline style="max-width: 100%; height: auto;">
  <source src="/videos/my-screencast.mp4" type="video/mp4">
</video>
```

Важно: в `hugo.toml` должен быть включен HTML в Markdown:

```toml
[markup.goldmark.renderer]
unsafe = true
```

---

## Восстановление “как в GitHub” (если на сервере что-то поменяли руками)

```bash
cd /var/www/help.stankoman.ru/src
git fetch origin
git reset --hard origin/main
git clean -fd
git submodule update --init --recursive
hugo --minify -d /var/www/help.stankoman.ru/public
```

---

## Рекомендуемый .gitignore

Добавь в корне проекта `.gitignore`:

```
/public/
.hugo_build.lock
/resources/
.DS_Store
Thumbs.db
```

---

## Подсказки

* На сервере **не редактировать контент руками**, чтобы не ловить конфликты.
* Тему не правим в `themes/…` напрямую — переопределяем в `layouts/…`.

