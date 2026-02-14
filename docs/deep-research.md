По фотографиям и по названию прошивки (`jorne_left-nice_nano_v2-zmk.uf2`) твою конфигурацию достаточно хорошо видно, давай её сначала формализуем, а потом разберёмся с прошивками.

***
## 1. Что за железо у тебя стоит
### PCB
- На третьем фото явно подписано: **`Jorne-WL v3.0.1 PCB by krikun98`**.  
- krikun98 — автор плат **Nijuni**, которые «полностью pin‑совместимы с Jorne и используют Jorne‑прошивки (QMK и ZMK)». [github](https://github.com/krikun98/nijuni)
- В официальном списке поддерживаемого железа ZMK Jorne идёт как отдельный шилд: `jorne_left` и `jorne_right`. [zmk](https://zmk.dev/docs/hardware)
- Твоя плата визуально ближе к Corne (3×6 + 3), но судя по подписи, по **разводке она совместима именно с Jorne**, а не с классической Corne. Это типичная ситуация: платы «в стиле Corne», но с проводкой Jorne, чтобы использовать уже готовые прошивки.

Итого по PCB: **беспроводная плата Jorne‑совместимого расклада, версия Jorne‑WL v3.0.1, шилд в ZMK — `jorne_left` / `jorne_right`.**
### Контроллеры
- На первом/втором фото — компактная плата на **nRF52840** в форм‑факторе Pro Micro с USB‑C, подпаянным SWD‑разъёмом `VDD / DIO / CLK / GND` и выводами `P0.xx` по краям.  
- Такие платы в магазинах обычно продаются как **«ProMicro nRF52840, совместима с nice!nano v2.0»** и прямо рекомендуют при сборке ZMK выбирать контроллер `nice_nano_v2`. [smdx](https://smdx.ru/nrf52840)
- Официальная документация nice!nano говорит, что он является drop‑in заменой Pro Micro и имеет тот же пиновый вывод. [nicekeyboards](https://nicekeyboards.com/docs/nice-nano/)

Поэтому можно считать, что контроллеры для целей ZMK — это **аналог `nice_nano_v2`** (UF2‑бутлоадер, BLE, питание от Li‑Po).
### Тип прошивки
- Имена рабочих файлов: `jorne_left-nice_nano_v2-zmk.uf2` / `jorne_right-nice_nano_v2-zmk.uf2`.  
- Такой формат имени полностью совпадает со стандартным артефактным именованием ZMK (`<shield>-<board>-zmk.uf2`, напр. `corne_left-nice_nano_v2-zmk.uf2` в доках). [zmk](https://zmk.dev/docs/customization)
- В wiki по nRFMicro Jorne собирается командой вроде `west build ... -b nrfmicro_13 -- -DSHIELD=jorne_left`, то есть именно через шилд `jorne_left/jorne_right`. [github](https://github.com/joric/nrfmicro/wiki/ZMK/424d80933d500326847a85a039d26b71d55e5a09)

Из этого следует:

> **Рабочая прошивка продавца — это ZMK с платой `nice_nano_v2` и шилдами `jorne_left` и `jorne_right`.**  
> Не `corne_left/right`.

***
## 2. Почему прошивки Corne не определяются как BT‑устройства
1. **Шилд не совпадает.**  
   Corne и Jorne в ZMK — разные шилды с разной матрицей и GPIO‑пинами. Когда ты заливаешь прошивку, собранную с `shield: corne_left/right`, ZMK: [zmk](https://zmk.dev/docs/hardware)
   - сканирует другие ножки контроллера (их на твоём PCB могли повесить на питание подсветки, батарею и т.п.);
   - по‑другому настраивает split‑режим и питание.

2. **Аппаратные особенности Jorne‑совместимых плат.**  
   В Jorne‑ветке ZMK и в нRFMicro‑wiki для Jorne дополнительно настраиваются сервисные пины (power‑MOSFET, батарея и т.д.). Если использовать шилд Corne, можно: [github](https://github.com/joric/nrfmicro/wiki/ZMK)
   - случайно выключать питание периферии;
   - мешать корректной инициализации BLE.  

   В результате контроллер либо вообще не доходит до стадии рекламы по Bluetooth, либо падает в ошибку до этого.

3. **Продавец явно собирал прошивку как Jorne.**  
   Если бы железо требовало Corne‑шилд, не было бы смысла называть артефакт `jorne_left‑nice_nano_v2‑zmk.uf2`.

Поэтому логичный вывод: **прошивки для Corne тебе в принципе не подходят, нужно использовать прошивки для Jorne (или Jorne‑совместимых плат), но собранные под `board: nice_nano_v2`.**

***
## 3. Какая конфигурация прошивки тебе нужна
Базовая конфигурация ZMK для твоей клавиатуры:

- **board**: `nice_nano_v2` (для обоих половин) [nicekeyboards](https://nicekeyboards.com/docs/nice-nano/)
- **shield**:
  - левый половин: `jorne_left`
  - правый половин: `jorne_right` [github](https://github.com/joric/nrfmicro/wiki/ZMK)
- режим split:
  - `jorne_left` по умолчанию играет роль **central**,  
  - `jorne_right` — **peripheral** (это прописано в Kconfig shield’а, как и для других сплитов). [zmk](https://zmk.dev/docs/development/hardware-integration/new-shield)
- опционально: отдельный билд со щитом `settings_reset` для сброса BLE‑привязок при экспериментах. [zmk](https://zmk.dev/docs/troubleshooting/connection-issues)

Пример `build.yaml` для репозитория `zmk-config`:

```yaml
include:
  - board: nice_nano_v2
    shield: jorne_left
  - board: nice_nano_v2
    shield: jorne_right
  - board: nice_nano_v2
    shield: settings_reset   # одноразовая прошивка для сброса настроек
```

После пуша в GitHub Actions ты получишь `.uf2` примерно с такими именами:

- `jorne_left-nice_nano_v2-zmk.uf2`
- `jorne_right-nice_nano_v2-zmk.uf2`
- `settings_reset-nice_nano_v2-zmk.uf2`

Именно первые два должны повести себя так же, как исходная прошивка продавца (по части BLE и матрицы), а их keymap уже будет зависеть от твоих файлов `*.keymap` / `*.conf`. [zmk](https://zmk.dev/docs/config)

***
## 4. Какие репозитории можно использовать как основу
### 4.1. Официальный конфиг Jorne для ZMK
- Репозиторий: **`joric/jorne-zmk-config`**. [github](https://github.com/joric/jorne-zmk-config)
- Это «официальный» конфиг для Jorne, предназначенный для работы с Jorne/nRFMicro, но логика клавиатуры и раскладки там уже отлажена.

Как адаптировать под твоё железо:

1. Форкнуть репозиторий.  
2. В `build.yaml` поменять `board: nrfmicro_13` (если он там ещё есть) на:

   ```yaml
   include:
     - board: nice_nano_v2
       shield: jorne_left
     - board: nice_nano_v2
       shield: jorne_right
   ```

3. В `west.yml` оставить ссылку на основной ZMK (или обновить до актуальной ветки main).  
4. При необходимости почистить всё, что связано с RGB/OLED, если у тебя их физически нет (в конфиге Jorne они могут быть включены).  

Плюс этого пути — **минимум возни с матрицей и split‑логикой**, всё уже сделано.
### 4.2. Готовые Corne‑репозитории как шаблон, но с шилдом Jorne
Многие популярные репозитории ZMK под Corne (включая те, что ты уже смотрел, и, вероятно, твой `neversad-dev/zmk-config`) уже настроены под: [github](https://github.com/neversad-dev/zmk-config)

- `board: nice_nano` или `nice_nano_v2`;
- split‑режим;
- GitHub Actions / keymap editor, и т.п.

Их можно использовать как шаблон, заменив только шилды:

1. Форк любого удобного репозитория Corne, например:
   - `mctechnology17/zmk-config` (поддержка corne/sofle/lily58), [github](https://github.com/mctechnology17/zmk-config)
   - один из тех, что ты привёл (`coleyoungsh/zmk-config-corne`, `josean-dev/zmk-config`),  
   - или свой же, если он уже есть.

2. В `build.yaml` заменить строки вида:

   ```yaml
   - board: nice_nano_v2
     shield: corne_left
   - board: nice_nano_v2
     shield: corne_right
   ```

   на:

   ```yaml
   - board: nice_nano_v2
     shield: jorne_left
   - board: nice_nano_v2
     shield: jorne_right
   ```

3. В `config/`:
   - либо переименовать `corne.keymap` → `jorne.keymap` и `corne.conf` → `jorne.conf`,
   - либо оставить общее имя без `_left/_right` (ZMK умеет использовать общий `.keymap` для обеих половинок, если нет специфичных файлов). [zmk](https://zmk.dev/docs/config)

4. Подогнать раскладку под фактические 42 клавиши:
   - Jorne‑шилд описывает 7 колонок; у тебя физически 6 — просто **не используй одну колонку в keymap** (позиции без ключей никогда не замкнутся, так что это безопасно).

Результат: **получаешь Corne‑подобный конфиг (слои, моды и т.п.), но с корректной аппаратной частью для твоего PCB.**
### 4.3. Собственный репозиторий с нуля по официальной инструкции
Если хочется всё под полный контроль:

- ZMK рекомендует создать свой `zmk-config` через их шаблон / CLI. [zmk](https://zmk.dev/docs/customization)
- В процессе выбора клавиатуры можно сразу указать `jorne` (она официально поддерживается). [zmk](https://zmk.dev/blog/2021/01/27/zmk-sotf-4)
- Дальше редактируешь только `jorne.keymap` и `jorne.conf` под себя.

Этот вариант даёт **максимальную предсказуемость** и точно не тянет за собой чужие хаки из готовых конфигов.

***
## 5. Практический план действий
1. **Сделать бэкап** рабочих `jorne_left/jorne_right ... uf2` (и не трогать их).  
2. Создать или форкнуть `zmk-config` (из `joric/jorne-zmk-config` или любого готового Corne‑репо).  
3. В `west.yml` оставить upstream ZMK, без сторонних форков, если они не нужны.  
4. В `build.yaml` задать:

   ```yaml
   include:
     - board: nice_nano_v2
       shield: jorne_left
     - board: nice_nano_v2
       shield: jorne_right
     - board: nice_nano_v2
       shield: settings_reset
   ```

5. В `config/jorne.keymap` описать свою раскладку (можно взять любой Corne‑keymap как основу и просто перекинуть символы на нужные позиции).  
6. Пушнуть в GitHub, дождаться сборки, скачать `firmware.zip`, прошить:
   - левый половин — `jorne_left-nice_nano_v2-zmk.uf2`,
   - правый половин — `jorne_right-nice_nano_v2-zmk.uf2`.

7. Если macOS вдруг не видит/не коннектит:
   - прошить на каждом контроллере `settings_reset-nice_nano_v2-zmk.uf2` (сброс настроек/BLE‑бондов); [zmk](https://zmk.dev/docs/troubleshooting/connection-issues)
   - удалить старое устройство в настройках Bluetooth macOS;
   - снова прошить основные `.uf2` и заново спарить.

После этого у тебя будет **полностью воспроизводимая ZMK‑конфигурация**, собираемая из репозитория, с которой можно безопасно экспериментировать с раскладкой, не потеряв исходную рабочую прошивку.
