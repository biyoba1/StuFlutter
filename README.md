# Студенты

Учебный проект по мобильной разработке. Форма из задания сохраняет имя, специальность и год выпуска. После входа приложение показывает имя пользователя, список студентов, форму добавления и кнопку «Выйти».

Эта копия основана на [LisLisich/StudyFlutter](https://github.com/LisLisich/StudyFlutter). Веб-демонстрация и Flutter-приложение оформлены в единой фиолетовой палитре с карточками и обновлёнными полями ввода.

## 1. Запуск веб-демонстрации

Откройте PowerShell в папке проекта и выполните:

```powershell
node preview.cjs
```

Откройте [http://127.0.0.1:8080/](http://127.0.0.1:8080/). Локальная демонстрация включена по умолчанию. Она хранит вымышленные учебные записи в браузере и имитирует авторизацию. Облачные режимы работают после настройки собственных проектов и отключения галочки «Локальная демонстрация».

| Вариант | Чтение | Добавление | Что происходит при записи из другой вкладки |
| --- | --- | --- | --- |
| Supabase без Realtime | `select()` | `insert()` | Нужно нажать «Обновить список» |
| Supabase с Realtime | `select()` | `insert()` | `postgres_changes` запускает повторный `select()` |
| Firebase без Realtime | `get()` | `push()` | Нужно нажать «Обновить список» |
| Firebase с Realtime | `onValue()` | `push()` | `onValue()` передаёт новый снимок списка |

**Проверка.** Откройте две вкладки с одним адресом. В первой выберите ручной режим, во второй выберите Realtime той же базы. Войдите в обеих вкладках и добавьте запись в первой. Во второй она появится автоматически. Добавьте другую запись во второй и убедитесь, что в первой она появится только после ручного обновления. Повторите с другой базой. Облачные списки Supabase и Firebase независимы.

## 2. Supabase

1. Создайте проект в [Supabase Dashboard](https://supabase.com/dashboard). В SQL Editor выполните `setup/supabase.sql`. Скрипт создаёт отдельную таблицу `public.lab_students`, выдаёт вошедшим пользователям права на чтение и добавление, включает RLS и добавляет таблицу в публикацию `supabase_realtime`.
2. Укажите Project URL и publishable key в `config.supabaseUrl` и `config.supabaseKey` внутри `docs/index.html`.
3. В **Authentication → URL Configuration** укажите адрес сайта и разрешите возврат на адрес страницы. При точной настройке добавьте варианты URL с `?cloud=1&mode=supabase-manual` и `?cloud=1&mode=supabase-live`. Для Flutter добавьте `studentslab://auth`.
4. В **Authentication → Sign In / Providers** включите Google, GitHub и Apple. Для OAuth-приложений у провайдеров используйте callback, показанный в настройках Supabase, обычно `https://PROJECT.supabase.co/auth/v1/callback`. Укажите Client ID и секреты в кабинете Supabase. Подробности есть в документации [Google](https://supabase.com/docs/guides/auth/social-login/auth-google), [GitHub](https://supabase.com/docs/guides/auth/social-login/auth-github) и [Apple](https://supabase.com/docs/guides/auth/social-login/auth-apple).

## 3. Firebase

1. В [Firebase Console](https://console.firebase.google.com/) войдите в Google, создайте проект и выберите **Project settings → Your apps → Web (`</>`) → Register app**. Analytics для задания можно оставить выключенным.
2. Создайте **Realtime Database** в Locked mode. На вкладке **Rules** опубликуйте `setup/database.rules.json` для нового проекта. В существующем проекте добавьте ветку `lab_students` к своим правилам, сохранив правила других веток. Для ограничения прав на `lab_students` корень не должен предоставлять широкий доступ. Правила из примера позволяют вошедшим читать учебный список и создавать новые записи с тремя полями.
3. Скопируйте `firebaseConfig` своего Web App в `config.firebase` внутри `docs/index.html`. Включите в него `databaseURL` из Realtime Database.
4. В **Authentication → Sign-in method** включите Google, GitHub и Apple. GitHub и Apple настраиваются отдельными OAuth-данными и Firebase callback вида `https://PROJECT.firebaseapp.com/__/auth/handler`. Для Apple потребуются реквизиты Apple Developer, перечисленные в инструкции Firebase. В **Authentication → Settings → Authorized domains** добавьте `USERNAME.github.io`, а для локального облачного запуска добавьте имя хоста, которое используете в браузере. [Google](https://firebase.google.com/docs/auth/web/google-signin) · [GitHub](https://firebase.google.com/docs/auth/web/github-auth) · [Apple](https://firebase.google.com/docs/auth/web/apple).
5. Используйте собственные проекты. На скриншотах преподавателя видны пример конфигурации и база `students`, а в этом проекте используется отдельная ветка `lab_students`. В клиентский HTML помещается публичная конфигурация. Сервисный ключ Firebase и OAuth-секреты хранятся на сервере или в кабинетах провайдеров.

## 4. Яндекс ID и VK ID

Кнопки используют отдельный OAuth-сервер. Сервер проверяет `state` и браузерную сессию, обменивает код провайдера с PKCE и запрашивает профиль. Для доступа к учебному списку Firebase выдаёт приложению сессию по серверному custom token. В браузере и Flutter выберите режим Firebase. При веб-входе возврат по умолчанию ведёт в режим Firebase Realtime.

1. Создайте приложения в [Яндекс OAuth](https://oauth.yandex.ru/client/new) и [VK ID](https://id.vk.ru/about/business). Для Яндекса нужен доступ `login:info`. Зарегистрируйте точные HTTPS callback URL `https://ВАШ-СЕРВЕР/callback/yandex` и `https://ВАШ-СЕРВЕР/callback/vk` соответственно. Для VK укажите домен сервера и веб-платформу.
2. Разместите папку `oauth` на Node.js-хостинге с HTTPS. Скопируйте `.env.example` в `.env`. Заполните `PUBLIC_ORIGIN`, `WEB_RETURN`, данные Яндекс/VK и подключите Firebase Admin с учётными данными **того же** Firebase-проекта. Для учебного запуска используйте один экземпляр сервера, поскольку незавершённые входы и одноразовые коды находятся в памяти процесса.
3. Из `oauth` выполните `npm ci`, затем `npm start`. Запрос `GET /health` должен вернуть `{"ok":true}`. Укажите `config.oauthServer` в HTML и `OAUTH_URL` при запуске Flutter. На GitHub Pages публикуется только папка `docs`.

Документация: [Яндекс ID](https://yandex.ru/dev/id/doc/ru/codes/code-url), [VK ID SDK](https://github.com/VKCOM/vkid-web-sdk/blob/master/src/auth/auth.ts), [Firebase custom tokens](https://firebase.google.com/docs/auth/admin/create-custom-tokens).

## 5. GitHub Pages

1. В ветке `main` стартовая страница находится в `docs/index.html`.
2. В **Settings → Pages → Build and deployment** выберите **Deploy from a branch**, `main` и `/docs`.
3. Проверьте адрес из Pages, обычно `https://biyoba1.github.io/StuFlutter/`. Для прямого открытия настроенного облачного режима добавьте к адресу `?cloud=1&mode=firebase-live`. [Инструкция GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site).

## 6. Flutter

В `mobile` находятся код приложения и зависимости. Для Android нужны Flutter SDK, Android SDK и эмулятор или телефон. Для iOS нужны macOS и Xcode.

1. Выполните `flutter pub get` из папки `mobile`. Проекты Android и iOS уже находятся в репозитории.
2. Для Firebase зарегистрируйте Android App с package name `org.studentlab.students_lab`, добавьте SHA-1/SHA-256 сертификата сборки и положите `google-services.json` в `mobile/android/app/`. Gradle подключит Google Services, когда файл присутствует. Для iOS зарегистрируйте Bundle ID `org.studentlab.studentsLab`, добавьте `GoogleService-Info.plist` в цель Runner и внесите значение `REVERSED_CLIENT_ID` в URL Types для Google. Для входа через Apple настройте Sign in with Apple capability в Xcode.
3. Добавьте `studentslab://auth` в **Supabase → Authentication → URL Configuration → Redirect URLs**. Этот адрес уже внесён в конфигурацию Android и iOS.
4. Запустите с публичной конфигурацией проекта. Параметр `GOOGLE_SERVER_CLIENT_ID` содержит Web OAuth Client ID Google, если он не подхватывается из платформенной конфигурации. Параметр `OAUTH_URL` нужен для Яндекса и VK.

```powershell
flutter run --dart-define=SUPABASE_URL=https://PROJECT.supabase.co --dart-define=SUPABASE_KEY=sb_publishable_REPLACE --dart-define=FIREBASE_DATABASE_URL=https://studyflutter-6a402-default-rtdb.europe-west1.firebasedatabase.app --dart-define=GOOGLE_SERVER_CLIENT_ID=REPLACE.apps.googleusercontent.com --dart-define=OAUTH_URL=https://oauth.example.com
```

[Flutter и Firebase](https://firebase.google.com/docs/flutter/setup) · [Вход через провайдеров во Flutter](https://firebase.google.com/docs/auth/flutter/federated-auth) · [Глубокие ссылки Supabase](https://supabase.com/docs/guides/auth/native-mobile-deep-linking)

## 7. Как работает Realtime

**Firebase Realtime Database** хранит дерево JSON. `onValue(query, callback)` сначала передаёт снимок текущего списка, затем передаёт его снова после изменений по этому пути. `get(query)` читает список один раз, `push(ref, row)` добавляет запись с новым ключом. Функция, возвращённая `onValue()`, отписывает слушателя. Во Flutter те же шаги выполняют `get()`, `push().set()` и `onValue.listen()`, а подписку закрывает `cancel()`. [Документация Firebase](https://firebase.google.com/docs/database/web/read-and-write).

**Supabase Realtime** читает изменения PostgreSQL из публикации `supabase_realtime`. Для таблицы `public.lab_students` нужны доступ на `SELECT` и разрешающая RLS-политика. JavaScript-клиент вызывает `channel(...).on('postgres_changes', {event:'*', schema:'public', table:'lab_students'}, callback).subscribe()`. После статуса `SUBSCRIBED` приложение загружает начальный список. Каждое событие запускает повторный `select()`. При выходе и смене режима вызывается `removeChannel()`. Во Flutter обработчик называется `onPostgresChanges()`. [Документация Supabase](https://supabase.com/docs/guides/realtime/postgres-changes).

## Проверено локально

В двух вкладках проверены все четыре варианта с учебными данными: вход, добавление, ручное обновление, доставка обновления во вторую вкладку и выход. Вход в облачные аккаунты, развёртывание на GitHub Pages и сборка Flutter требуют окружения и настроек владельца. Облачная интеграция с XupaStack не проверялась.
