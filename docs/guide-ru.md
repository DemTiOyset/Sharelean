# Гайд: запуск Sharelean_Umarov и сдача работы через Pull Request

Гайд рассчитан на то, что ты **никогда раньше не публиковал проект на GitHub**.
Делай шаги по порядку, ничего не пропуская. Все команды выполняются из корня проекта
(папка `Sharelean`) — там, где лежит `pom.xml`.

Данные этой работы (уже подставлены в команды):

| Параметр | Значение |
|---|---|
| Твой GitHub | <https://github.com/DemTiOyset> (username: `DemTiOyset`) |
| Имя репозитория | `Sharelean_Umarov` |
| Преподаватель-проверяющий | <https://github.com/Tunia666> (username: `Tunia666`) |
| Локальная папка проекта | `testing_pril/Sharelean` |

---

## 0. Общая картина: что мы делаем и зачем

```
твой компьютер                    GitHub
┌──────────────────┐              ┌─────────────────────────────────────────┐
│ папка Sharelean  │  git push    │ репозиторий Sharelean_Umarov            │
│  pom.xml         │ ───────────► │   main      ← защищённая ветка          │
│  src/test/...    │              │   feature/sharelane-e2e ← ветка с PR    │
└──────────────────┘              │                                          │
                                  │   PR #1  ◄── Approve от Tunia666        │
                                  │            (автор PR не может           │
                                  │             одобрить свой PR сам)       │
                                  └─────────────────────────────────────────┘
```

Схема сдачи именно такая, как просят преподаватели: **не** просто «залил код в `main`»,
а «сделал ветку → открыл Pull Request → пригласил проверяющего → получил Approve → смержил».
Это доказывает, что ты умеешь работать с git-флоу, а не только писать тесты.

---

## 1. Установка окружения

Нужны три вещи: **JDK 21**, **Maven**, **Google Chrome**.

### 1.1 Windows

1. **JDK 21.** Скачай установщик Temurin JDK 21 (x64, `.msi`):
   <https://adoptium.net/temurin/releases/?version=21> → при установке оставь галочку
   *Set JAVA_HOME variable* и *Add to PATH*.
2. **Maven.** Скачай `apache-maven-3.9.x-bin.zip`:
   <https://maven.apache.org/download.cgi> → распакуй, например, в `C:\tools\maven`
   (внутри должен оказаться `C:\tools\maven\bin\mvn.cmd`).
3. Добавь Maven в PATH: *Пуск* → «Изменение переменных среды текущего пользователя» →
   `Path` → *Изменить* → *Создать* → `C:\tools\maven\bin` → ОК.
4. **Chrome.** <https://www.google.com/chrome/> — обычная установка.
5. Закрой и заново открой терминал (PowerShell), затем раздел 1.3.

### 1.2 Linux (Ubuntu/Debian) и macOS

Ubuntu / Debian:

```bash
sudo apt update
sudo apt install -y openjdk-21-jdk maven
```

macOS (через Homebrew):

```bash
brew install openjdk@21 maven
```

Chrome: <https://www.google.com/chrome/> (Linux) или `brew install --cask google-chrome` (macOS).

### 1.3 Проверка окружения

```bash
java -version
mvn -version
```

Ожидаемо: `openjdk version "21.x"` и `Apache Maven 3.9.x`, причём в выводе `mvn -version`
в строке `Java version:` должно быть **21**, а не 17 или 8. Если там другая версия —
Maven берёт не ту Java, поправь `JAVA_HOME`.

Chrome отдельно проверять не нужно: Selenium 4 сам найдёт браузер и скачает подходящий
ChromeDriver (через Selenium Manager).

### 1.4 Альтернатива: запуск из IntelliJ IDEA

Если тебе привычнее IDEA: `File → Open` → выбери **папку `Sharelean`** (именно папку с `pom.xml`,
не весь `testing_pril`) → IDEA подтянет Maven-проект → в файле `SharelaneE2ETest.java`
нажми зелёный треугольник рядом с методом. IDEA сама скачает зависимости.

---

## 2. Запуск тестов

### 2.1 Первый запуск

```bash
mvn test
```

Первый запуск идёт долго (3–10 минут): Maven качает Selenium, TestNG и их зависимости в
локальный кэш `~/.m2/repository`, а Selenium Manager — ChromeDriver. Это нормально.
Повторные запуски занимают ~30–60 секунд.

### 2.2 Признак успеха

В конце вывода должно быть примерно так:

```
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

Тест по умолчанию работает в headless-режиме (браузер не открывается визуально).
Хочешь посмотреть глазами, как бегает браузер:

```bash
mvn test -Dheadless=false
```

Полезные параметры:

| Команда | Что делает |
|---|---|
| `mvn test` | обычный прогон, headless |
| `mvn test -Dheadless=false` | показать окно Chrome |
| `mvn test -Dtimeout.seconds=60` | увеличить таймауты ожиданий (медленный интернет / VPN) |
| `mvn test -Dtest=SharelaneE2ETest` | запустить только этот класс |
| `mvn clean test` | очистить `target/` и прогнать заново |

### 2.3 Отчёты и разбор падений

Результаты лежат в `target/surefire-reports/`:

- `index.html` — открыть в браузере, удобный отчёт TestNG;
- `emailable-report.html` — тот же отчёт, который удобно приложить к сдаче;
- `org.example.sharelane.SharelaneE2ETest.txt` — краткая текстовая сводка по классу;
- `TEST-org.example.sharelane.SharelaneE2ETest.xml`, `testng-results.xml` — машинные отчёты.

Если тест упал, смотри в отчёте строку `at org.example.sharelane.SharelaneE2ETest...` —
это место, где не совпало ожидание, и текст сообщения из `Assert`. Скриншоты в коде теста
намеренно не делаются, чтобы не усложнять проект: снимки для отчёта снимаются с консоли
вручную — как это сделать, см. раздел 2.5.

### 2.4 На что не надо пугаться

- `Tests run: 1, Failures: 1` со словами `Invalid card number` — значит сайт не вернул
  ожидаемый текст ошибки (изменился стенд), а не «тест сломан» сам по себе;
- тест создаёт нового пользователя при каждом запуске — так и задумано, ShareLane удаляет
  тестовые аккаунты в начале следующего часа;
- оплата всегда отклоняется: настоящей карты у нас нет, тест это и проверяет.

### 2.5 Скриншоты консоли для отчёта

Код теста для этого менять не нужно — всё делается в терминале.

**Что должно быть видно на скриншотах:** сама команда, все 8 шагов, итоговая строка
`Tests run: 1, Failures: 0` и `BUILD SUCCESS`.

1. Открой терминал: Windows Terminal / PowerShell / Git Bash, либо окно **Run** в IntelliJ IDEA.
2. Разверни окно на весь экран, увеличь шрифт (в Windows Terminal — `Ctrl` + прокрутка колесом),
   очисти экран: `cls` (Windows) или `clear` (Linux/macOS).
3. Запусти тест в batch-режиме (`-B` убирает цветные коды и прогресс-бары, скриншот читается
   лучше) и сразу сохрани лог в файл:
   - Linux / macOS / Git Bash: `mvn -B test 2>&1 | tee test-run.log`
   - Windows PowerShell: `mvn -B test 2>&1 | Tee-Object test-run.log`
   - cmd.exe: просто `mvn -B test`, а лог потом скопируешь из окна.
4. Дождись `BUILD SUCCESS` (первый запуск — до 10 минут, повторный — меньше минуты).
5. Сделай три скриншота:
   - **№ 1 — начало:** видно команду `mvn -B test` и запуск сборки;
   - **№ 2 — середина:** строки `[step 1]`…`[step 8]` и ответ сайта
     `[step 8] ShareLane response: Oops, error. Invalid card number or not enough balance for purchase`;
   - **№ 3 — итог:** `Tests run: 1, Failures: 0, Errors: 0, Skipped: 0` и `BUILD SUCCESS`.
6. Как снимать экран:
   - **Windows:** `Win + Shift + S` — выделить область (снимок попадает в буфер и вставляется
     в Word через `Ctrl + V`). Либо `Win + PrtScn` — весь экран, файл сохранится в
     «Изображения → Снимки экрана».
   - **macOS:** `Cmd + Shift + 4`, затем выделить область.
   - **Linux (GNOME):** `PrtScn` — весь экран, `Shift + PrtScn` — область.
7. Дополнительно к скриншотам приложи:
   - файл `test-run.log` — полный текст прогона;
   - скриншот отчёта `target/surefire-reports/index.html`, открытого в браузере: там аккуратный
     отчёт TestNG с процентом пройденных тестов.

**Совет:** прокрути консоль так, чтобы в кадр попали и команда, и результат — тогда видно,
что снимок относится именно к этому прогону.

---

## 3. Публикация проекта на GitHub

### 3.1 Подготовка git (один раз на компьютере)

```bash
git config --global user.name "Timur Umarov"
git config --global user.email "твой-email@example.com"
git config --global init.defaultBranch main
```

> `user.email` лучше указать тот же, что привязан к GitHub-аккаунту, иначе коммиты не
> свяжутся с профилем.

### 3.2 Создать локальный репозиторий

Из корня проекта (`testing_pril/Sharelean`):

```bash
git init -b main
git add .
git status          # проверь: в списке нет target/ и .idea/
git commit -m "feat: add ShareLane end-to-end Selenium test with TestNG assertions"
```

`target/` и `.idea/` уже перечислены в `.gitignore`, поэтому в коммит не попадут.
Если `git status` показывает `target/` — значит ты запускаешь команды не из папки
`Sharelean` или `.gitignore` не сохранился.

### 3.3 Создать пустой репозиторий на GitHub

1. Открой <https://github.com/new>.
2. **Repository name:** `Sharelean_Umarov`
3. **Description:** `ShareLane UI autotests (Java 21, Selenium, TestNG)`
4. **Public** — обязательно публичный (см. шаг 6 про защиту ветки).
5. **Не** ставь галочки `Add a README file`, `Add .gitignore`, `Choose a license` —
   иначе при первом `push` будет конфликт «историй».
6. Нажми **Create repository**.

### 3.4 Связать локальный репозиторий с GitHub и запушить

```bash
git remote add origin https://github.com/DemTiOyset/Sharelean_Umarov.git
git push -u origin main
```

При `push` GitHub спросит логин и пароль. **Пароль от аккаунта не подходит** — нужен
Personal Access Token (PAT):

1. <https://github.com/settings/tokens> → **Tokens (classic)** → **Generate new token (classic)**;
2. Note: `sharelean-push`, Expiration: 30 дней;
3. галочка **`repo`** (полный доступ к репозиториям);
4. **Generate token** → скопируй строку вида `ghp_...` (она показывается один раз);
5. в терминале: Username — `DemTiOyset`, Password — вставь этот токен.

Если хочется без токена — настрой SSH-ключ
(<https://docs.github.com/ru/authentication/connecting-to-github-with-ssh>) и используй
remote вида `git@github.com:DemTiOyset/Sharelean_Umarov.git`.

Альтернатива для тех, у кого стоит GitHub CLI:

```bash
gh auth login                 # выбор: HTTPS, авторизация через браузер
gh repo create Sharelean_Umarov --public --source=. --push
```

### 3.5 Проверка

Открой <https://github.com/DemTiOyset/Sharelean_Umarov> — файлы `pom.xml`, `README.md`,
`docs/guide-ru.md` и `src/test/java/org/example/sharelane/SharelaneE2ETest.java` на месте,
папки `target/` нет.

---

## 4. Ветка и Pull Request

Дальше работаем **не в `main`**, а в отдельной ветке: именно её мы превратим в Pull Request,
который должен подтвердить преподаватель.

### 4.1 Создать ветку и внести осмысленную правку

```bash
git checkout -b feature/sharelane-e2e
```

Что-то поправь по мелочи, чтобы в PR был diff (иначе GitHub откажется создавать PR из пустой
ветки). Например, добавь в `README.md` строку с датой сдачи или уточни комментарий в тесте.

```bash
git add README.md
git commit -m "docs: add submission date to README"
git push -u origin feature/sharelane-e2e
```

### 4.2 Открыть Pull Request

1. После `push` GitHub покажет баннер **Compare & pull request** — нажми его
   (или вручную: вкладка **Pull requests** → **New pull request**).
2. Проверь направления: **base: `main`** ← **compare: `feature/sharelane-e2e`**.
3. **Title:** `ShareLane e2e test: sign up, login, search, cart, checkout`
4. **Description** (можно скопировать как есть):

```markdown
## Что сделано
- Сквозной e2e-тест ShareLane на Selenium 4 + TestNG (Java 21).
- Покрыт путь покупателя: регистрация с ZIP-кодом → логин → поиск книги →
  добавление в корзину → checkout → проверка отказа оплаты по тестовой карте.
- В отличие от референса используются настоящие ассерты TestNG, а логин/пароль
  считываются со страницы ShareLane (не захардкожены).

## Как проверить
1. `mvn test` (нужны JDK 21, Maven, Chrome)
2. Ожидаемый результат: `Tests run: 1, Failures: 0, Errors: 0` и `BUILD SUCCESS`
3. Отчёт: `target/surefire-reports/index.html`

## Примечания
- Оплата намеренно отклоняется сайтом: настоящей тестовой карты нет, тест фиксирует
  корректный ответ ShareLane `Invalid card number...`.
```

5. Справа в блоке **Reviewers** нажми шестерёнку и выбери `Tunia666`. Если её там ещё нет —
   сначала выполни шаг 5 (приглашение), потом вернись.
6. Нажми **Create pull request**.

> **Почему это ключевой момент.** GitHub запрещает автору Pull Request одобрять собственный PR
> (кнопка Approve у тебя будет недоступна или неактивна). Значит, «подтверждённый PR» может
> появиться только от второго человека — от преподавателя. Это ровно то, что от тебя требуется.

---

## 5. Приглашение преподавателя

1. Открой <https://github.com/DemTiOyset/Sharelean_Umarov/settings/access>.
   (Вкладка **Settings** видна только владельцу репозитория.)
2. **Add people** → введи `Tunia666` → роль **Write** → **Add**.
   - Роль **Write** нужна обязательно: голос проверяющего засчитывается правилом
     «Required approvals» только для пользователей с правами записи.
   - Если GitHub не находит её по username — попроси точный ник или ссылку на профиль
     (в этой работе: <https://github.com/Tunia666>).
3. Преподаватель получит письмо/уведомление **Invitation to collaborate** со ссылкой
   `https://github.com/DemTiOyset/Sharelean_Umarov/invitations` — она должна нажать
   **Accept invitation**.
4. После принятия приглашения снова открой свой PR → блок **Reviewers** → выбери `Tunia666`
   (это отправит ей уведомление «review requested»).

Если преподаватель не хочет принимать приглашение — репозиторий публичный, поэтому она
может оставить ревью и как обычный пользователь GitHub. Но тогда её Approve **не будет
засчитан** защитой ветки, и смержить PR придётся тебе самому (в истории PR всё равно
останется её ревью — это тоже доказательство проверки).

---

## 6. Защита ветки `main`: без Approve смержить нельзя

Чтобы подтверждение преподавателя было не «пожеланием», а техническим требованием:

1. Открой `https://github.com/DemTiOyset/Sharelean_Umarov/settings/rules` (новый интерфейс Rulesets)
   или `.../settings/branches` (классические Branch protection rules).
2. **Rulesets:** **New ruleset** → **New branch ruleset**.
   **Или:** **Branches** → **Add branch protection rule**.
3. **Ruleset Name / Branch name pattern:** `main`
4. Включи:
   - ✅ **Require a pull request before merging**
     - ✅ **Required approvals: 1**
     - ✅ **Dismiss stale pull request approvals when new commits are pushed**
     - ✅ **Require conversation resolution before merging**
   - ✅ **Block force pushes**
5. **Create** / **Save changes**.

Теперь кнопка **Merge pull request** в PR будет серой с подписью
*«Review required — At least 1 approving review is required»* до тех пор, пока преподаватель
не нажмёт Approve. Это самый убедительный скриншот для отчёта: «до» и «после».

> На бесплатном тарифе GitHub правила защиты ветки работают **только в публичных**
> репозиториях. Если репозиторий приватный, пункт будет недоступен — сделай его публичным:
> `Settings → General → Danger Zone → Change repository visibility → Make public`.

---

## 7. Что делает преподаватель (передай ей этот блок)

> 1. Открой ссылку на Pull Request: `https://github.com/DemTiOyset/Sharelean_Umarov/pull/1`
> 2. Вкладка **Files changed** → просмотри код теста.
> 3. Кнопка **Review changes** (справа сверху) → выбери **Approve** →
>    при желании напиши комментарий → **Submit review**.
> 4. Если есть замечания — выбери **Comment** или **Request changes** и напиши, что поправить.
>
> После Approve у PR появится зелёная отметка **Approved** и станет доступна кнопка Merge.

Полезно приложить к сообщению преподавателю короткий текст:

```text
Здравствуйте! Сдаю практическую работу по ShareLane.
Репозиторий: https://github.com/DemTiOyset/Sharelean_Umarov
Pull Request: https://github.com/DemTiOyset/Sharelean_Umarov/pull/1
Тест запускается командой mvn test (JDK 21 + Maven + Chrome), ожидаемый результат —
BUILD SUCCESS, Tests run: 1, Failures: 0. Отчёт: target/surefire-reports/index.html.
Отправил Вам приглашение в коллабораторы (роль Write) — нужно принять его,
затем в PR нажать Review changes → Approve. Спасибо!
```

### 7.1 Где взять ссылку, которую отправлять преподавателю

Заранее этой ссылки не существует: **адрес PR появляется только после того, как ты запушил
ветку и нажал Create pull request**. Дальше взять её можно четырьмя способами.

| Что нужно | Где взять |
|---|---|
| **Ссылка на PR** (её и отправляешь) | Адресная строка, когда PR открыт: `https://github.com/DemTiOyset/Sharelean_Umarov/pull/1`. Шаблон: `.../pull/<номер>` |
| Быстрый способ без браузера | После `git push -u origin feature/sharelane-e2e` GitHub печатает в консоли строку *Create a pull request for 'feature/sharelane-e2e' on GitHub by visiting:* и ссылку — скопируй её оттуда |
| Ссылка сразу на код для ревью | К той же ссылке добавь `/files`: `https://github.com/DemTiOyset/Sharelean_Umarov/pull/1/files` — открывается вкладка Files changed с кнопкой **Review changes** |
| Список всех PR (если номер забыл) | `https://github.com/DemTiOyset/Sharelean_Umarov/pulls` — номер PR виден в списке, это просто его порядковый номер |

Дополнительно ей может понадобиться:

| Ссылка | Кому и зачем |
|---|---|
| `https://github.com/DemTiOyset/Sharelean_Umarov` | репозиторий целиком |
| `https://github.com/DemTiOyset/Sharelean_Umarov/invitations` | **ей** — принять приглашение в коллабораторы (без принятия Approve не засчитается) |
| `https://github.com/DemTiOyset/Sharelean_Umarov/pull/1` | открыть PR, нажать **Review changes → Approve → Submit review** |

Короткое сообщение, которое можно отправить целиком:

```text
Здравствуйте! Приглашаю Вас проверить мою работу по ShareLane.
Репозиторий: https://github.com/DemTiOyset/Sharelean_Umarov
Pull Request: https://github.com/DemTiOyset/Sharelean_Umarov/pull/1
Проверить код можно сразу здесь: https://github.com/DemTiOyset/Sharelean_Umarov/pull/1/files
Пожалуйста, примите приглашение в коллабораторы (письмо от GitHub)
и в PR нажмите Review changes → Approve → Submit review. Спасибо!
```

> Номер PR (`/pull/1`, `/pull/2`, …) — это просто порядковый номер. Первый PR в репозитории
> всегда `pull/1`. Точный адрес всегда видно в адресной строке открытого PR, гадать не нужно.

---

## 8. После одобрения: смержить и подчистить

1. Убедись, что в PR видны зелёная галочка **Approved** и зелёные проверки.
2. Нажми **Merge pull request** → **Squash and merge** → **Confirm squash and merge**
   (squash удобнее: история `main` остаётся линейной).
   Если кнопка серая — вернись к шагу 6 и проверь правило «Required approvals: 1».
3. **Delete branch** (кнопка появится после мержа) — ветка больше не нужна.
4. Локально обнови `main` и удали ветку:

```bash
git checkout main
git pull
git branch -d feature/sharelane-e2e
```

5. Собери доказательства для отчёта:
   - ссылка на репозиторий;
   - ссылка на PR (в нём видно Approve от преподавателя и статус Merged);
   - скриншот страницы PR с **Approved** и **Merged**;
   - скриншот/лог успешного `mvn test` (`BUILD SUCCESS`);
   - HTML-отчёт `target/surefire-reports/index.html`.

---

## 9. Следующие работы: как сохранить изоляцию

Договорённость: **каждая практическая работа — отдельная папка и отдельный репозиторий**.
Так преподавателю удобно проверять по одной ссылке, а работы не смешиваются в истории коммитов.

```
projects/testing_pril/
├── Sharelean/            ← эта работа, репозиторий Sharelean_Umarov
├── <СледующаяРабота>/    ← новая папка, новый репозиторий, например <Название>_Umarov
└── <ЕщёРабота>/          ← и так далее
```

Порядок для каждой новой работы:

1. Новая папка рядом с `Sharelean` (не внутри неё!). Внутри — свой `pom.xml`.
2. `git init -b main` **внутри новой папки** (не в `testing_pril`!).
3. Новый репозиторий на GitHub, новое имя.
4. Новый `git remote add origin ...` (адрес этого проекта не переиспользуй).
5. Своя ветка `feature/...`, свой PR, своё приглашение проверяющего — как в шагах 4–7.

Быстрый старт новой Maven-работы с тем же стеком: скопируй из `Sharelean` файлы
`pom.xml` и `.gitignore`, замени `<artifactId>`, `<name>` и тестовый класс.

Проверить, что ты всё-таки в изолированном репозитории, а не в общем:

```bash
git remote -v     # должен быть один origin — этот проект, и никаких других
git log --oneline # только коммиты этой работы
```

---

## 10. Типичные проблемы и решения

| Симптом | Причина | Решение |
|---|---|---|
| `java: command not found` / `'mvn' не является внутренней командой` | PATH не видит JDK/Maven | Переустанови/пропиши PATH, перезапусти терминал, проверь `java -version` |
| `invalid target release: 21` / `Unsupported class file major version` | Maven работает на старой Java | Поставь JDK 21, выставь `JAVA_HOME` на неё, проверь строку `Java version:` в `mvn -version` |
| Первый `mvn test` идёт очень долго | Maven качает зависимости | Подожди; далее они берутся из кэша `~/.m2` |
| `Could not resolve dependencies ... Connection timed out` | Прокси/файрвол/VPN | Повтори, попробуй без VPN, при корпоративном прокси настрой `settings.xml` |
| `session not created: This version of ChromeDriver only supports Chrome version...` | Устаревший ChromeDriver | Удали `~/.cache/selenium`, обнови Chrome, повтори `mvn test` (Selenium Manager скачает нужный драйвер) |
| `Chrome not found` / `cannot find Chrome binary` | Chrome не установлен | Установи Google Chrome; либо запусти `mvn test -Dheadless=false` после установки |
| Тест падает на `Account is created!` | Сайт ShareLane недоступен или изменился | Открой <https://sharelane.com> в браузере; проверь интернет; если вёрстка изменилась — поправь локаторы |
| `git push` → `Authentication failed` / `Support for password authentication was removed` | Пароль вместо токена | Создай PAT (шаг 3.4) и используй его как пароль |
| `remote origin already exists` | remote уже добавлен | `git remote set-url origin https://github.com/DemTiOyset/Sharelean_Umarov.git` |
| `fatal: not a git repository` | Команда запущена не из папки проекта | `cd` в `Sharelean` (там, где `pom.xml`) |
| `src refspec main does not match any` | Нет ни одного коммита | Сначала `git add .` и `git commit -m "..."`, потом `push` |
| При первом `create repo` GitHub ругается на непустой репозиторий | Ты поставил галочку README при создании | `git pull --rebase origin main`, затем `git push` |
| Нет пункта `Collaborators` | Открыт не свой репозиторий или нет прав | Открой `Settings` под своим аккаунтом-владельцем |
| Правила защиты ветки недоступны | Репозиторий приватный, тариф Free | Сделай репозиторий публичным (шаг 6) |
| Кнопка Merge серая: `Review required` | Нет Approve от проверяющего | Преподаватель должна нажать Review changes → Approve; проверь, что у неё роль Write |
| Преподаватель не может approve | Не принято приглашение / нет доступа | Пусть примет приглашение из письма; либо PR в публичном репо может ревьюить любой |
| Approve был, но после нового коммита исчез | Включён «Dismiss stale approvals» | Попроси подтвердить PR ещё раз после последнего коммита |
| `Timed out receiving message from renderer` | Chrome завис: не хватает ресурсов (частая история в виртуалках, при нехватке ОЗУ или в режиме энергосбережения) | Закрой лишние окна браузера и программы, повтори `mvn -B test`; попробуй `mvn test -Dheadless=false`; перезагрузи компьютер |

---

## 11. Финальный чек-лист сдачи

- [ ] `mvn test` локально даёт `BUILD SUCCESS`, `Tests run: 1, Failures: 0`
- [ ] Репозиторий создан: <https://github.com/DemTiOyset/Sharelean_Umarov> (public)
- [ ] В репозитории нет `target/` и `.idea/`
- [ ] Ветка `feature/sharelane-e2e` запушена, PR открыт и назначен на `Tunia666`
- [ ] Преподаватель добавлен в коллабораторы с ролью **Write** и принял приглашение
- [ ] Включено правило защиты `main`: «Require a pull request» + «Required approvals: 1»
- [ ] На PR есть зелёный статус **Approved** от `Tunia666`
- [ ] PR смержен (Squash and merge), ветка удалена
- [ ] Сделаны скриншоты консоли: команда `mvn -B test`, шаги `[step 1]`…`[step 8]`,
      строка `Tests run: 1, Failures: 0` и `BUILD SUCCESS` (см. 2.5)
- [ ] Сохранён лог прогона `test-run.log` и скриншот отчёта `surefire-reports/index.html`
- [ ] Собраны скриншоты GitHub: PR с **Approved** и **Merged**
- [ ] Следующие работы будут в своих отдельных папках и репозиториях

---

## 12. Шпаргалка команд

```bash
# запуск тестов
mvn test
mvn test -Dheadless=false -Dtimeout.seconds=60

# первая публикация
git init -b main
git add .
git commit -m "feat: add ShareLane end-to-end Selenium test with TestNG assertions"
git remote add origin https://github.com/DemTiOyset/Sharelean_Umarov.git
git push -u origin main

# ветка и PR
git checkout -b feature/sharelane-e2e
git add . && git commit -m "docs: describe how to run the tests"
git push -u origin feature/sharelane-e2e

# после мержа
git checkout main && git pull
git branch -d feature/sharelane-e2e

# проверка состояния
git status
git remote -v
git log --oneline --graph --all
```
