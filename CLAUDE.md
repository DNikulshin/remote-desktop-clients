# Контекст
## Цель
Форк aRDP (iiordanov/remote-desktop-clients, ветка no-donate-banner) для раздачи пользователям: RDP-подключение по шаблону, вход по своим логину и паролю. Сборка и подпись через GitHub Actions. Два варианта: personal (com.dnikulshin.ardp) и shared для общих устройств (com.dnikulshin.ardp.shared).
## Принятые решения (с датами)
- 2026-10-24: баннер/диалог доната отключён ранним return в IntroTextDialog.showIntroTextIfNecessary
- 2026-10-24: сборка в GitHub Actions (JDK 21, :aRDP-app:assembleDebug), релиз arm64-APK; подпись постоянным keystore через secrets KEYSTORE_B64, KEYSTORE_PASSWORD, KEY_ALIAS, KEY_PASSWORD
- 2026-10-24: applicationId com.dnikulshin.ardp + добавлен в Utils.standardPackageNames (иначе isCustom=true ломает режим приложения)
- 2026-10-25: скрыты (visibility=gone, код не удалён) карточка RDP Gateway и значок Morpheusly; блок «Показать дополнительные настройки» свёрнут, видны размер (по умолчанию «Другой» 260x400), режим курсора («запрещено»), режим безопасности, аудио, строка версии; умолчания заданы в ConnectionBean
- 2026-10-25: окно «Добавить подключение»: только «Ввести вручную»; фоновый поиск mDNS отключён (serviceType = null)
- 2026-10-26: ранний запрос логина: RdpCommunicator.connect() при пустом логине/пароле сразу шлёт GET_RDP_CREDENTIALS (раньше было 2 анонимные попытки на брокер до формы); после этого вход через брокер на тестовой учётке проходит стабильно
- 2026-10-26: shared определяется по имени пакета (Utils.isSharedDevice: содержит «shared»), сборка с -PsharedDevice=true; в shared: в форме входа нет Domain и чекбокса «Запомнить», в настройках подключения нет чекбокса, ConnectionBean.save принудительно keepPassword=false (логин запоминается, пароль нет)
- 2026-10-26: порядок полей в настройках RDP (оба варианта): сервер+порт → домен → логин → пароль (перестановка в aRDP.java)
- 2026-10-27: тихий возврат в список при выходе из сеанса на сервере (RdpCommunicator.reportSessionEnded: строка ошибки FreeRDP через рефлексию → DISCONNECT_NO_MESSAGE)
- 2026-10-27: меню «три точки»: оставлены «Новое соединение», «Импорт/экспорт», «Копировать logcat», остальное visible=false
- 2026-10-27: «глаз» показать/скрыть пароль в форме входа (GetTextFragment.addPasswordToggle, ic_pw_show/ic_pw_hide)
- Шаблоны XML для импорта: таблица CONNECTION_BEAN, одна строка, _id=1001; KEEPPASSWORD=1 для personal, 0 для shared; адрес, порт, ник, домен подставляются вручную
## Что сделано
- Форк, патчи, workflow apk.yml (два варианта, подпись apksigner, публикация в Releases build-N с двумя APK: ardp-personal и ardp-shared), проверка на телефоне
## Что осталось
- Проверить, что бэкап ardp-release.jks, паролей и отпечатка SHA-256 хранится минимум в двух местах вне Firebase Studio
- Брокер (18 серверов): остаются ли два диалога с отпечатком сертификата; если мешают, список доверенных отпечатков в X509KEYSIGNATURE (патч validateCert) или единый сертификат на серверах
- Опционально: описание релиза из RELEASE_NOTES.md (сейчас текст зашит в apk.yml), Pro-функции (гейты Utils.isFree), своё название/иконка у shared
- При обновлении с upstream проверять, не вышел ли libs новее libs-17 (тогда убрать заглушку setGlyphCache)
## Нюансы / баги / обходные пути
- libs-17 старее кода: setGlyphCache закомментирован в RdpCommunicator.java
- Play Protect ругается на неизвестного разработчика: «Все равно установить»
- .gitignore игнорирует build-*; workflow называется apk.yml
- Скрытые элементы не удаляются из layout: код обращается к ним через findViewById (visibility=gone)
- Импорт настроек: перезапись строк по _id (не слияние); в XML пароли открытым текстом; колонка, которой нет в схеме, роняет импорт; умолчания (260x400 и др.) только для новых подключений
- Сертификаты: хранится один отпечаток (X509KEYSIGNATURE); после первого «Да» флаг certificateAccepted остаётся true, и второй сертификат (целевой сервер) соединение не блокирует
- Любой обрыв до OnConnectionSuccess приложение считает ошибкой пароля и открывает форму входа (OnDisconnecting)
- Репозиторий и релизы публичные: не класть в них адреса серверов, пароли и keystore; шаблон XML раздавать закрыто
- В Firebase Studio нет python3 по умолчанию (nix-shell -p python3 / jdk21 для keytool); патчи делались sed и bash-скриптами (разовые, в репозиторий не коммитить)