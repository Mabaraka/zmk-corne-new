# Финальная конфигурация ZMK для Jorne-совместимой клавиатуры

## Аппаратное обеспечение

**PCB**: Jorne-WL v3.0.1 (pin-совместима с Jorne, NOT Corne)
**Контроллеры**: nice!nano v2 (nRF52840) на обеих половинах
**Количество клавиш**: 42 физических (Jorne layout: 44 позиции, 2 фантомные)

## Конфигурация репозитория

### Версия ZMK
**Закреплена на**: `v0.2.1` (последний стабильный релиз перед Zephyr 4.1)

**Почему v0.2.1, а не main?**
- ZMK main включает Zephyr 4.1 с большими изменениями в структуре board definitions
- Board `nice_nano_v2` был консолидирован в `nice_nano` с системой ревизий
- В v0.2.1 `nice_nano_v2` остаётся валидным именем (как в прошивке продавца)
- Гарантирует совместимость с hardware и стабильность BLE

### Структура конфигов

#### `build.yaml`
```yaml
include:
  - board: nice_nano_v2
    shield: jorne_left
  - board: nice_nano_v2
    shield: jorne_right
  - board: nice_nano_v2
    shield: settings_reset
```

#### `config/west.yml`
```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: v0.2.1
      import: app/west.yml
  self:
    path: config
```

#### `.github/workflows/build.yml`
```yaml
on: [push, pull_request, workflow_dispatch]

jobs:
  build:
    uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.2.1
```

## Конфигурационные файлы

### `config/jorne.conf`
**Минимальная конфигурация** для стабильного BT-подключения:
- Только `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y` включена
- RGB Underglow и Display отключены (физически отсутствуют)
- Pointing device **отключен** (требует аппаратной поддержки, ломает инициализацию)
- Sleep и экспериментальные BLE опции **отключены** (могут вызвать проблемы с открытием)

### `config/jorne.keymap`
**44 позиции** (Jorne layout):
- Row 1: `&none` (pos 0) + 12 клавиш + `&none` (pos 13)
- Row 2–3: 12 клавиш каждая
- Thumb cluster: 6 клавиш
- **3 слоя**: base, L1 (symbols/bluetooth), num (numbers/navigation)
- **Custom behaviors**: hml/hmr (hold-tap для модификаторов), tkl/tkr (layer toggles)
- **Combos**: game mode toggle (pos 38+43), ESC (pos 2+3)

**Важно**: Первые 2 позиции (0 и 13) помечены как `&none` — они соответствуют фантомным клавишам Jorne layout, которых физически нет.

## Порядок действий при прошивке

### 1. Первоначальная прошивка (заводская)
Прошивка `jorne_left-nice_nano_v2-zmk.uf2` и `jorne_right-nice_nano_v2-zmk.uf2` от продавца используют именно эту конфигурацию.

### 2. Пересборка своей версии

1. **Форк репозитория** или клонирование этой конфигурации
	- https://github.com/joric/jorne-zmk-config - можно за основу брать эту, но заменить файл build.yaml на то, что указано выше 
2. **Пуш в GitHub** (или локальная сборка через `west`)
3. **Дождаться сборки** (GitHub Actions)
4. **Загрузить артефакты**:
   - `jorne_left-nice_nano_v2-zmk.uf2` для левой половины
   - `jorne_right-nice_nano_v2-zmk.uf2` для правой половины
   - `settings_reset-nice_nano_v2-zmk.uf2` для сброса BLE (при необходимости)

### 3. Прошивка контроллеров

Для каждой половины:
1. Подключить USB-C контроллер к компьютеру
2. Двойной клик на кнопку RESET
3. Контроллер появится как USB-диск (UF2 bootloader)
4. Скопировать `.uf2` файл на диск
5. Контроллер перезагрузится и отключится. В процессе может возникать ошибка записи - это нормально

### 4. Bluetooth спаривание (macOS)

1. Включить батарею на обеих половинах
2. Удалить старые Bluetooth устройства "Jorne" из macOS (если были)
3. Система должна автоматически обнаружить **"Jorne"** в доступных устройствах (имя по умолчанию)
4. Выбрать и подключиться

**Если не видна:**
- Убедиться, что обе половины прошиты нормальной прошивкой (не settings_reset)
- Если была прошита settings_reset, нужно прошить нормальную прошивку на обе половины
- Можно попробовать сначала прошить settings_reset на обе, потом нормальную (сброс BLE-состояния)

## Отличия от типовых конфигов

| Вопрос | Типовой Corne config | Наша конфигурация |
|---|---|---|
| Board name | `nice_nano_v2` ОК в main ZMK | `nice_nano_v2` (v0.2.1 required) |
| Shield | `corne_left/right` | `jorne_left/right` (другая матрица) |
| Keymap positions | 42 | 44 (2 фантомные) |
| Pointing | Может быть включено | **Отключено** (нет hardware) |
| ZMK version | main (Zephyr 4.1) | v0.2.1 (Zephyr 3.x) |

## Частые ошибки

### ❌ "No board named 'nice_nano_v2' found"
**Причина**: ZMK main (Zephyr 4.1) переименовал board
**Решение**: Использовать ZMK v0.2.1 (текущая конфигурация)

### ❌ Клавиатура не видна как BT устройство
**Возможные причины**:
1. Прошита `settings_reset` (Bluetooth отключен) — нужна нормальная прошивка
2. `CONFIG_ZMK_POINTING=y` включен без hardware — отключить
3. Old BT pairing на macOS — удалить и переподключить

### ❌ Keymap не компилируется
**Частые причины**:
- Неправильная нумерация позиций (помнить про 44 вместо 42)
- Несуществующие keycodes в v0.2.1 (например, `&kp GLOBE`)

## Обновление конфигурации

Если нужно добавить опции (RGB, sleep, pointing):
1. Добавить строки в `config/jorne.conf`
2. Убедиться, что опция поддерживается в ZMK v0.2.1
3. Пересобрать и протестировать

## Ссылки

- Официальный Jorne config: https://github.com/joric/jorne-zmk-config
- ZMK документация: https://zmk.dev
- ZMK v0.2.1 release: https://github.com/zmkfirmware/zmk/releases/tag/v0.2.1
- Jorne shield в ZMK: https://zmk.dev/docs/hardware
