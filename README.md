# Scripting API — Ebullience Client

Скрипты пишутся на **JavaScript** и загружаются из папки `%APPDATA%/Ebullience/scripts/`.
Каждый файл `.js` — один модуль.

---

## Структура скрипта

```js
var script = {
    name: "MyScript",        // Название модуля (обязательно)
    description: "...",      // Описание
    key: 0,                  // Клавиша включения (0 = нет)
    type: Type.MAIN,         // Тип: Type.MAIN

    onEnable:  function() {},
    onDisable: function() {},
    onUpdate:  function(event) {},
    onRender2D: function(event) {},
    onRender3D: function(event) {},
    onChat:     function(event) { return false; }, // return true = отменить сообщение
    onEntityMove: function(event) {}
};

register(); // Регистрирует модуль в клиенте — обязательно в конце
```

Все методы кроме `name` и `register()` — опциональны. Определяй только те, что нужны.

---

## Опции

Опции регистрируются **до** объекта `script` и добавляются через `registerOption()`.
Они появляются в GUI клиента при выборе модуля.

```js
var myBool   = new BooleanOption("Enabled", true);
var myFloat  = new FloatOption("Speed", 1.0, 0.1, 5.0).addIncrementValue(0.1);
var myColor  = new ColorOption("Color", 0xFF00AAFF);
var mySelect = new SelectOption("Mode", 0,
    new SelectOptionValue("Normal"),
    new SelectOptionValue("Fast")
);
var myMulti  = new MultiOption("Flags",
    new MultiOptionValue("Jump", true),
    new MultiOptionValue("Speed", false)
);

registerOption(myBool);
registerOption(myFloat);
registerOption(myColor);
registerOption(mySelect);
registerOption(myMulti);
```

### Чтение значений

```js
myBool.getValue()    // true / false
myFloat.getValue()   // число
myColor.getValue()   // int (ARGB)
mySelect.getValueByIndex(0)  // true если выбран индекс 0
myMulti.isSelected("Jump")   // true если флаг включён
```

---

## Встроенные переменные

| Переменная | Тип | Описание |
|---|---|---|
| `mc` | `Minecraft` | Главный объект игры |
| `mc.player` | `EntityPlayerSP` | Локальный игрок |
| `mc.world` | `WorldClient` | Текущий мир |
| `mc.gameSettings` | `GameSettings` | Настройки игры |
| `client` | `Main` | Объект клиента |

**Всегда проверяй** `mc.player !== null` и `mc.world !== null` перед использованием.

---

## Рендер 2D

Рисование происходит в методе `onRender2D`.

### Прямоугольник

```js
// drawRect(x, y, ширина, высота, цвет)
RenderUtility.drawRect(10, 10, 100, 20, 0x90000000);
```

### Скруглённый прямоугольник

```js
// drawRoundedRect(x, y, x2, y2, радиус, цвет)
RenderUtility.drawRoundedRect(10, 10, 110, 30, 4, 0xFF333333);
```

### Текст

Текст рисуется через шрифты из `FontManager`.

```js
var font = FontManager.SEMI_BOLD_14;

font.drawString("Hello", x, y, 0xFFFFFFFF);
font.getStringWidth("Hello")  // ширина строки в пикселях
font.getFontHeight()          // высота шрифта
```

### Доступные шрифты

```
FontManager.REGULAR_14 / REGULAR_16 / REGULAR_18 / REGULAR_20
FontManager.SEMI_BOLD_10 / SEMI_BOLD_12 / SEMI_BOLD_14 / SEMI_BOLD_16 / SEMI_BOLD_18
FontManager.BOLD_16 / BOLD_20
FontManager.MEDIUM_20
```

### Цвета

Цвет задаётся в формате `0xAARRGGBB`:

```js
0xFF000000  // чёрный, непрозрачный
0x90000000  // чёрный, полупрозрачный
0xFFFFFFFF  // белый
0xFF00FF44  // зелёный
0xFFFF4444  // красный
0xFF00AAFF  // синий
```

---

## Логирование

```js
log.info("сообщение")   // в консоль
log.warn("сообщение")
log.error("сообщение")
```

---

## Команды

```
.script list              — список загруженных скриптов и их статус
.script toggle <name>     — включить / выключить модуль
.script reload            — перезагрузить все скрипты
.script reload <file.js>  — перезагрузить один скрипт
```

---

## Полный пример

```js
// AutoSprint.js

var showHUD  = new BooleanOption("Show HUD", true);
var hudColor = new ColorOption("Color", 0xFF00AAFF);

registerOption(showHUD);
registerOption(hudColor);

var script = {
    name: "AutoSprint",
    description: "Автоматический спринт",
    key: 0,
    type: Type.MAIN,

    onEnable: function() {
        log.info("AutoSprint включён");
    },

    onDisable: function() {
        if (mc.player !== null) {
            mc.player.setSprinting(false);
        }
    },

    onUpdate: function(event) {
        if (mc.player === null) return;
        if (mc.world  === null) return;

        if (mc.gameSettings.keyBindForward.isKeyDown()) {
            mc.player.setSprinting(true);
        }
    },

    onRender2D: function(event) {
        if (mc.player === null)      return;
        if (!showHUD.getValue())     return;

        var font  = FontManager.SEMI_BOLD_14;
        var label = mc.player.isSprinting() ? "SPRINT ON" : "SPRINT OFF";
        var w     = font.getStringWidth(label) + 10;
        var h     = font.getFontHeight() + 6;

        RenderUtility.drawRoundedRect(5, 5, 5 + w, 5 + h, 3, 0x90000000);
        font.drawString(label, 10, 8, hudColor.getValue());
    }
};

register();
```

---

## Частые ошибки

**`TypeError: X is not a function`**
Метода нет в том классе.

**Модуль не появляется в списке**
Скорее всего ошибка в скрипте при загрузке. Проверь консоль - там будет строка с ошибкой и номером строки.

**`mc.player is null`**
Скрипт выполняется раньше чем игрок вошёл в мир. Всегда проверяй `mc.player !== null`.

**Скрипт не обновился после изменения**
Используй `.script reload <file.js>` - файл перечитается без перезапуска клиента.

---

## Справочник API

### RenderUtility

```js
// Прямоугольник (x, y, ширина, высота, цвет)
RenderUtility.drawRect(x, y, w, h, color)

// Прямоугольник от точки до точки (x1, y1, x2, y2, цвет)
RenderUtility.drawCRect(x, y, x2, y2, color)

// Градиентный прямоугольник (4 угловых цвета)
RenderUtility.drawGradientRect(x, y, w, h, color1, color2, color3, color4)

// Скруглённый прямоугольник (x, y, x2, y2, радиус, цвет)
RenderUtility.drawRoundedRect(x, y, x2, y2, radius, color)

// Скруглённый прямоугольник с градиентом (4 угловых цвета)
RenderUtility.drawRoundedGradientRect(x, y, x2, y2, radius, color1, color2, color3, color4)

// Скруглённый контур (x, y, x2, y2, радиус, толщина, цвет)
RenderUtility.drawRoundedOutlineRect(x, y, x2, y2, radius, thickness, color)

// Тень под прямоугольником (x, y, x2, y2, мягкость, радиус, цвет)
RenderUtility.drawRoundedShadow(x, y, x2, y2, softness, radius, color)

// Круговой прогресс-бар (x, y, радиус, прогресс 0.0-1.0, толщина, цвет)
RenderUtility.drawARCCircle(x, y, radius, progress, thickness, color)

// Круговой прогресс-бар с градиентом
RenderUtility.drawARCCircle(x, y, radius, progress, thickness, color1, color2)

// Изображение (ResourceLocation, x, y, x2, y2)
RenderUtility.drawImage(resourceLocation, x, y, x2, y2)
```

---

### WFontRenderer (шрифт)

```js
var font = FontManager.SEMI_BOLD_14; // любой шрифт из FontManager

// Нарисовать строку
font.drawString("текст", x, y, color)

// Ширина строки в пикселях
font.getStringWidth("текст")

// Высота шрифта в пикселях
font.getFontHeight()
```

---

### ColorUtility

```js
// Получить RGBA компоненты как float[] {r, g, b, a} (0.0 - 1.0)
ColorUtility.getRGBAf(color)

// Радужный цвет по времени
ColorUtility.rainbow(speed, offset)
```

---

### ChatUtility

```js
// Отправить сообщение в чат (отображается только локально)
ChatUtility.addChatMessage("текст")

// Отправить сообщение на сервер
ChatUtility.sendMessage("текст")
```

---

### MathHelper (Minecraft)

```js
MathHelper.clamp(value, min, max)   // ограничить значение
MathHelper.sqrt(value)
MathHelper.sin(radians)
MathHelper.cos(radians)
MathHelper.floor(value)
MathHelper.ceil(value)
MathHelper.abs(value)
MathHelper.wrapDegrees(degrees)     // нормализовать угол в -180..180
```

---

### Keyboard / Mouse (LWJGL)

```js
// Проверить нажата ли клавиша (коды клавиш LWJGL)
Keyboard.isKeyDown(keyCode)

// Примеры кодов:
// 17 = W, 31 = S, 30 = A, 32 = D
// 57 = Пробел, 42 = Shift, 29 = Ctrl
// 1 = Escape, 28 = Enter

// Мышь
Mouse.isButtonDown(0)  // ЛКМ
Mouse.isButtonDown(1)  // ПКМ
Mouse.isButtonDown(2)  // СКМ
```

---

### mc.player (EntityPlayerSP)

```js
mc.player.posX / posY / posZ           // текущая позиция
mc.player.prevPosX / prevPosY / prevPosZ  // позиция прошлого тика
mc.player.motionX / motionY / motionZ  // скорость
mc.player.rotationYaw / rotationPitch  // угол камеры
mc.player.onGround                     // стоит ли на земле
mc.player.isSprinting()                // в спринте?
mc.player.setSprinting(true/false)
mc.player.isSwimming()
mc.player.isInWater()
mc.player.isSneaking()
mc.player.getHealth()                  // текущее HP
mc.player.getMaxHealth()
mc.player.getFoodStats().getFoodLevel() // уровень насыщения
mc.player.inventory.getCurrentItem()   // предмет в руке (может быть null)
mc.player.getName()                    // ник игрока
```

---

### mc.world (WorldClient)

```js
mc.world.getTotalWorldTime()      // общее время мира в тиках
mc.world.getWorldTime()           // время суток (0-24000)
mc.world.isRaining()
mc.world.isThundering()
mc.world.playerEntities           // список игроков (List)
mc.world.loadedEntityList         // список всех сущностей (List)

// Получить блок по координатам
var pos = new BlockPos(x, y, z);
mc.world.getBlockState(pos).getBlock()
```

---

### Опции — методы

```js
// BooleanOption
new BooleanOption("Name", defaultValue)
option.getValue()        // boolean
option.setValue(true)

// FloatOption
new FloatOption("Name", value, min, max)
new FloatOption("Name", value, max)        // min = 0
option.addIncrementValue(0.1)              // шаг изменения
option.getValue()        // float
option.setValue(1.5)
option.getMin()
option.getMax()

// ColorOption
new ColorOption("Name", 0xFFFFFFFF)
option.getValue()        // int (ARGB)
option.setValue(0xFF00FF00)

// SelectOption
new SelectOption("Name", defaultIndex, value1, value2, ...)
option.getValueByIndex(0)   // true если выбран этот индекс
option.getValue().getName() // название выбранного значения

// MultiOption
new MultiOption("Name", value1, value2, ...)
option.isSelected("ValueName")  // boolean

// SelectOptionValue / MultiOptionValue
new SelectOptionValue("Name")
new MultiOptionValue("Name", defaultToggle)
```

---

### Доступные классы в скриптах

| Класс | Описание                        |
|---|---------------------------------|
| `RenderUtility` | Рисование 2D/3D примитивов      |
| `ColorUtility` | Работа с цветом                 |
| `ChatUtility` | Чат                             |
| `FontManager` | Доступ к шрифтам                |
| `MathHelper` | Математика Minecraft            |
| `Keyboard` | Клавиатура (LWJGL)              |
| `Mouse` | Мышь (LWJGL)                    |
| `GL11` | OpenGL константы и функции      |
| `GlStateManager` | Управление GL состоянием        |
| `BooleanOption` | Опция переключатель             |
| `FloatOption` | Числовая опция                  |
| `ColorOption` | Опция цвет                      |
| `SelectOption` | Опция выбор одного              |
| `SelectOptionValue` | Значение для SelectOption       |
| `MultiOption` | Опция множественный выбор       |
| `MultiOptionValue` | Значение для MultiOption        |
| `Type` | Тип модуля (MAIN) |
| `ScriptModule` | Базовый класс модуля скрипта    |
| `Module` | Базовый класс любого модуля     |

---

### Скорость игрока (пример расчёта)

```js
var dx    = mc.player.posX - mc.player.prevPosX;
var dz    = mc.player.posZ - mc.player.prevPosZ;
var speed = Math.sqrt(dx * dx + dz * dz) * 20; // блоков/сек
```
