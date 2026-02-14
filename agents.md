# agents.md

Этот файл содержит инструкции для Claude Code (claude.ai/code) при работе с этим репозиторием.

## Обзор проекта

**Статус**: ✅ Полностью рабочая конфигурация

Этот репозиторий управляет прошивкой ZMK для беспроводной **Jorne-совместимой** клавиатуры (не Corne!) с поддержкой Bluetooth на контроллерах nice!nano v2.

⚠️ **ВАЖНО**: Это конфигурация для **Jorne**, а не Corne. Различаются shields, матрица клавиш и нумерация позиций.

## Аппаратное обеспечение

- **Клавиатура**: Jorne-WL v3.0.1 (pin-совместима с Jorne, NOT Corne)
- **Контроллеры**: nice!nano v2 (nRF52840) на обеих половинах
- **Подключение**: Bluetooth Low Energy (BLE)
- **Количество клавиш**: 42 физических (44 позиции в Jorne layout)
- **Платформа**: macOS

## Версия ZMK и совместимость

**Текущая конфигурация**:
- **ZMK**: v0.2.1 (закреплена, последний стабильный релиз перед Zephyr 4.1)
- **Board**: `nice_nano_v2`
- **Shields**: `jorne_left`, `jorne_right`

**Почему v0.2.1, а не main?**
- ZMK main включает Zephyr 4.1 с переименованием board definitions
- `nice_nano_v2` был консолидирован в `nice_nano@2.0.0`
- v0.2.1 гарантирует совместимость с аппаратом и стабильность BLE
- Последний файл прошивки продавца (`jorne_left-nice_nano_v2-zmk.uf2`) собран на этой версии

## Структура репозитория

### Конфигурация ZMK (основные файлы)
- `build.yaml` - Матрица сборки для GitHub Actions
  - Board: `nice_nano_v2`
  - Shields: `jorne_left`, `jorne_right`, `settings_reset`
- `.github/workflows/build.yml` - GitHub Actions workflow
  - Версия: `@v0.2.1` (совпадает с ZMK версией в west.yml)
- `/config/` - Директория конфигурации ZMK
  - `jorne.keymap` - Раскладка (3 слоя: base, L1 symbols/BT, L2 numbers/nav)
  - `jorne.conf` - Настройки прошивки (минимальная конфигурация для стабильности)
  - `west.yml` - Манифест зависимостей ZMK (version: v0.2.1)

### Справочные материалы
- `/firmware` - Бинарные прошивки (результаты сборок)
  - `jorne_left-nice_nano_v2-zmk.uf2` - левая половинка
  - `jorne_right-nice_nano_v2-zmk.uf2` - правая половинка
  - `settings_reset-nice_nano_v2-zmk.uf2` - сброс Bluetooth/настроек (ВАЖНО: BLE отключен в этой прошивке!)
- `/docs` - Документация (на русском языке)
  - `deep-research.md` - Анализ аппаратной части и выбор Jorne вместо Corne
  - `configuration-summary.md` - Финальная техническая конфигурация
  - `README.md` - Полное руководство для пользователей
- `keys_ru.h` - Заголовок с русской раскладкой (если используется)

## Конфигурация раскладки

### Текущая раскладка
**Файл**: `/config/jorne.keymap`

- **3 слоя**:
  1. **base**: QWERTY с hold-tap модификаторами (hml/hmr behaviors)
  2. **L1**: символы, номера, Bluetooth управление (BT_SEL 0-4, BT_CLR)
  3. **L2**: цифры, навигация, медиа

- **Custom behaviors**:
  - `hml`/`hmr`: Hold-tap для левых/правых модификаторов (CTRL, ALT, SHIFT, CMD)
  - `tkl`/`tkr`: Layer toggle (hold для слоя, tap для клавиши)

- **Combos**:
  - `pos 38+43`: Toggle game mode
  - `pos 2+3`: ESC

### Редактирование раскладки

**Способ 1: Прямое редактирование**
1. Отредактировать `config/jorne.keymap`
2. Пуш в GitHub
3. GitHub Actions автоматически соберёт прошивку
4. Скачать `.uf2` файлы из Actions → Artifacts

**Способ 2: Веб-редактор (если нужна синхронизация)**
⚠️ Веб-редактор может потребовать настройки для Jorne (не Corne!).
Рекомендуется прямое редактирование в репозитории.

## Процесс прошивки

### Первоначальная прошивка
1. **Удалить из Bluetooth** (macOS): System Settings → Bluetooth → забыть старые устройства
2. Для **каждой половинки**:
   - Подключить USB-C контроллер
   - Двойной клик кнопку RESET или замкнуть RST-GND на 1 сек
   - Контроллер появится как USB диск (NICENANO или аналогичный)
   - Скопировать `.uf2` файл на диск
   - Диск автоматически отключится
3. **Включить батарею** на обеих половинах
4. **Система обнаружит** "Corne" в доступных Bluetooth устройствах
5. **Подключиться** через диалог паринга

### Обновление прошивки
Повторить шаги 2-4 для обновления конфигурации

### Сброс Bluetooth состояния (если не коннектит)
1. **Сначала**: прошить `settings_reset-nice_nano_v2-zmk.uf2` на обе половины
   - ⚠️ В этой прошивке Bluetooth отключен (это нормально, функция сброса)
2. **Дождаться** перезагрузки обеих половин
3. **Затем**: прошить нормальные `.uf2` файлы (`jorne_left` и `jorne_right`)
4. **Переподключиться** через Bluetooth

## Сборка прошивки

### GitHub Actions (рекомендуется)
1. Пуш изменений в GitHub
2. Перейти на вкладку Actions
3. Дождаться завершения workflow
4. Скачать `firmware.zip` из Artifacts
5. Распаковать и использовать `.uf2` файлы

### Локальная сборка
```bash
# Требует: west, Zephyr 3.x, ZMK v0.2.1

west build -b nice_nano_v2 -d build/jorne_left -- -DSHIELD=jorne_left
west build -b nice_nano_v2 -d build/jorne_right -- -DSHIELD=jorne_right
```

Результаты в `/build/jorne_left/zephyr/zmk.uf2` и т.д.

## Конфигурация в config/jorne.conf

**Текущие параметры**:
```ini
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y  # Увеличить мощность Bluetooth
# CONFIG_ZMK_RGB_UNDERGLOW=y     # RGB (если есть)
# CONFIG_ZMK_DISPLAY=y           # OLED дисплей (если есть)
```

**Отключено (для стабильности)**:
- `CONFIG_ZMK_POINTING=y` — требует аппаратной поддержки, может ломать BLE
- `CONFIG_ZMK_SLEEP=y` — не нужен для большинства
- `CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y` — экспериментальная опция

При добавлении опций убедиться, что они поддерживаются в ZMK v0.2.1!

## Важные замечания для Claude Code

### Jorne vs Corne
- **Shields**: `jorne_left/right` (не `corne_left/right`!)
- **Позиции клавиш**: 44 (а не 42), позиции 0 и 13 помечены как `&none` (фантомные)
- **Матрица**: другая разводка пинов, несовместима с типовыми Corne конфигами

### Версия ZMK важна!
- **main ZMK** (Zephyr 4.1): требует `nice_nano` и переименования shields
- **v0.2.1 ZMK**: использует `nice_nano_v2` и текущие названия shields
- При обновлении ZMK нужно менять board name и проверять совместимость shields

### Bluetooth паринг
- Settings_reset **отключает** BLE (для сброса состояния паринга)
- Нормальная прошивка **включает** BLE и автоматически рекламирует профиль
- На macOS старое устройство нужно удалить перед переподключением

### Редактирование keymap
- Помнить про 44 позиции, а не 42!
- Первый ряд: `&none` (pos 0) + 12 клавиш + `&none` (pos 13)
- При изменении behaviors/combos пересчитывать позиции!

## Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| "No board named 'nice_nano_v2' found" | Используется main ZMK | Использовать v0.2.1 (текущая конфиг) |
| Клавиатура не видна в Bluetooth | Прошита settings_reset или старое состояние | Прошить settings_reset на обе, потом нормальную |
| Keymap не компилируется | Неправильная позиция или несуществующий keycode | Проверить позиции (44!), проверить поддержку в v0.2.1 |
| BLE глючит/отключается | Конфликтующие опции в .conf | Использовать минимальную конфигурацию (текущая) |

## Полезные ресурсы

- **ZMK Документация**: https://zmk.dev
- **Официальный Jorne конфиг**: https://github.com/joric/jorne-zmk-config
- **ZMK GitHub**: https://github.com/zmkfirmware/zmk
- **ZMK v0.2.1 Release**: https://github.com/zmkfirmware/zmk/releases/tag/v0.2.1
- **Jorne в ZMK**: https://zmk.dev/docs/hardware
- **Bluetooth troubleshooting**: https://zmk.dev/docs/troubleshooting/connection-issues

## История изменений

**2026-02-15**: Финальная рабочая конфигурация
- ✅ ZMK v0.2.1 (стабильный)
- ✅ Jorne shields + nice_nano_v2
- ✅ Bluetooth паринг работает
- ✅ Все клавиши функционируют
- ✅ Документация полная

**Предыдущие версии**: См. git history
