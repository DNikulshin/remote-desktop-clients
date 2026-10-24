# Контекст
## Цель
Свой форк aRDP (iiordanov/remote-desktop-clients) без донат-баннера, сборка через GitHub Actions
## Принятые решения (с датами)
- 2026-10-24: баннер отключён ранним return в IntroTextDialog.showIntroTextIfNecessary
- 2026-10-24: сборка в GitHub Actions (JDK 21, :aRDP-app:assembleDebug), релиз только arm64-APK
- 2026-10-24: applicationId com.dnikulshin.ardp + добавлен в Utils.standardPackageNames (иначе isCustom=true)
- 2026-10-24: подпись постоянным keystore через secrets KEYSTORE_B64, KEYSTORE_PASSWORD, KEY_ALIAS, KEY_PASSWORD
## Что сделано
- Форк, патчи, workflow apk.yml, релизы build-N, установка и проверка на телефоне
## Что осталось
- Бэкап ardp-release.jks и паролей вне Firebase Studio (если ещё не сделан)
- Опционально: Pro-функции (гейты Utils.isFree), своё название/иконка
## Нюансы / баги / обходные пути
- libs-17 старее кода: setGlyphCache закомментирован в RdpCommunicator.java
- Play Protect ругается на неизвестного разработчика: «Все равно установить»
- .gitignore игнорирует build-*, workflow называется apk.yml