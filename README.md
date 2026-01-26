# api:

# Документация Scripting API

Данная система скриптинга построена на базе **ChaiScript**. Это встраиваемый скриптовый язык для C++, синтаксис которого очень напоминает C++ и JavaScript.

*   **Официальный сайт:** [https://chaiscript.com/](https://chaiscript.com/)
*   **Особенность:** Язык строго типизирован внутри (C++), но динамичен снаружи. Переменные объявляются через `var` или `global` (если нужно сохранить значение между кадрами).

---

## 1. Базовые типы и Математика

### Вектор 3D (`vec3`)
Используется для координат в мире.

*   **Создание:**
    ```javascript
    var v1 = vec3();          // (0, 0, 0)
    var v2 = vec3(10, 20, 30);// (x, y, z)
    var v3 = vec3(10, 20, 30).clone(); // Копия вектора
    ```
*   **Поля:** `v.x`, `v.y`, `v.z`
*   **Операции:** `v1 + v2`, `v1 - v2`, `v1 = v2` (присваивание).
*   **Методы:**
    *   `v.dist()` — возвращает длину вектора (magnitude).
    *   `to_string(v)` — возвращает строку вида `"(x, y, z)"`.

### Вектор 2D (`vec2`)
Используется для экранных координат и размеров интерфейса.

*   **Создание:** `var v = vec2(x, y);`
*   **Поля:** `v.x`, `v.y`
*   **Операции:** Сложение `+`, вычитание `-`, `clone()`.

### Цвет (`Color`)
Используется во всех функциях отрисовки.

*   **Создание:** `Color(r, g, b, a)` (0-255).
*   **Поля:** `.r`, `.g`, `.b`, `.a`.
*   **Пример:** `var red = Color(255, 0, 0, 255);`

---

## 2. Unity API: Transform и Компоненты

Это самая важная часть. В движке есть два типа структур Transform: **Internal** (внутренние структуры Unity) и **Normal** (обертки C#). Важно знать, какой использовать.

### Основные функции
*   `Component_get_transform(ptr)` — Получает Transform из любого компонента (например, из Player).
    *   ⚠️ **Возвращает Internal Transform**.
*   `Biped_get_head(biped_map)` / `Biped_get_body(...)` — Получают Transform кости.
    *   ✅ **Возвращает Normal Transform**.

### Получение позиции
Для работы с разными типами трансформов есть две функции:

1.  **`Transform_get_position(transform)`**
    *   Автоматически вызывает версию с `false` (считает, что трансформ обычный). Подходит для костей.
2.  **`Transform_get_position_ex(transform, is_internal)`**
    *   Позволяет указать тип трансформа вручную.
    *   `is_internal = true` — если трансформ получен через `Component_get_transform`.
    *   `is_internal = false` — если трансформ получен через кости (`Biped`).

**Пример:**
```javascript
// 1. Позиция игрока (Internal)
var trans_comp = Component_get_transform(player); 
var pos_player = Transform_get_position_ex(trans_comp, true); 

// 2. Позиция головы (Normal)
var view = Player_get_view(player);
var map = View_get_biped_map(view);
var trans_head = Biped_get_head(map);
var pos_head = Transform_get_position_ex(trans_head, false); 
// или просто: var pos_head = Transform_get_position(trans_head);
```

### World to Screen
*   `update_camera()` — **Обязательно** вызывать один раз в начале `on_render`. Кэширует матрицу.
*   `w2s(vec3_world, vec3_screen_out)` — Конвертирует 3D в 2D.
    *   Возвращает `true`, если объект на экране.
    *   Записывает результат во второй аргумент (передача по ссылке).

---

## 3. Игровые сущности (Game SDK)

### Player Manager
*   `PlayerManager_get_player_manager()` — Получить адрес менеджера.
*   `PlayerManager_get_local_player(mgr)` — Адрес вашего игрока.
*   `PlayerManager_get_players(mgr)` — Возвращает `std::map` (словарь) всех игроков.

### Player
Любой `player_ptr` наследуется от Component.
*   `Player_get_photon(player)` — Сетевые данные.
*   `Player_get_team(player)` — Команда (int).
*   `Player_get_view(player)` — Визуальные данные (скелет).
*   `Player_is_visible(player)` — Проверка видимости (Raycast check).

### Photon (Сетевая часть)
*   `Photon_get_name(photon)` — Имя игрока (string).
*   `Photon_get_int(photon, "prop_name")` — Чтение кастомных пропов (также есть `_float`, `_bool`, `_double`).
*   `Photon_set_int(photon, "prop_name", value)` — Запись пропов (также `_float` и т.д.).

---

## 4. Память (Memory)

Функции для прямого чтения/записи памяти по адресу.

*   `is_valid(addr)` — Проверка указателя на валидность.
*   **Чтение:**
    *   `read_int(addr)`, `read_float(addr)`, `read_bool(addr)`
    *   `read_vec3(addr)`
    *   `read_ptr(addr)` (читает uint32_t)
*   **Запись:**
    *   `write_int(addr, val)`, `write_float(addr, val)` и т.д.

*   **Работа с массивами Unity:**
    *   `Array_get_pointers(array_ptr)` — Возвращает вектор адресов (`uint32_list`) из стандартного массива Unity.

---

## 5. Отрисовка (Render API)

Все координаты экранные (`vec2` или `float x, y`).

*   `get_display_size()` — Возвращает `vec2` с разрешением экрана.
*   `calc_text_size(text, font_size)` — Возвращает `vec2` размера текста (для центрирования).

**Примитивы:**
*   `draw_line(x1, y1, x2, y2, color, thickness)`
*   `draw_rect(x, y, w, h, color, rounding, thickness)`
*   `draw_rect_filled(x, y, w, h, color, rounding)`
*   `draw_circle(x, y, radius, color, segments, thickness)`
*   `draw_text(x, y, text, color, shadow_bool)`

---

## 6. Интерфейс (GUI / Menu)

Используются внутри функции `on_menu()`. Функции принимают глобальные переменные для изменения их состояния.

*   `gui_text("Text")` — Просто текст.
*   `gui_button("Label")` — Кнопка, возвращает `true` при нажатии.
*   `gui_checkbox("Label", global_var)` — Чекбокс.
*   `gui_slider_float("Label", global_var, min, max)` — Слайдер.
*   `gui_slider_int("Label", global_var, min, max)` — Слайдер целочисленный.

---

## Пример полного скрипта

```javascript
// Настройки (Global - сохраняются между кадрами)
global cfg_enable = true;
global cfg_dist = 100.0;
global col_box = Color(255, 0, 0, 255);

// Меню
def on_menu()
{
    gui_text("My Script");
    gui_checkbox("Enable ESP", cfg_enable);
    if (cfg_enable)
    {
        gui_slider_float("Distance", cfg_dist, 10.0, 500.0);
    }
}

// Рендер
def on_render()
{
    if (!cfg_enable)
    {
        return;
    } 

    // 1. Обновляем камеру
    update_camera();
    var screen_size = get_display_size();

    // 2. Получаем данные игры
    var mgr = PlayerManager_get_player_manager();
    if (!is_valid(mgr))
    {
        return;
    } 
    
    var local = PlayerManager_get_local_player(mgr);
    if(!is_valid(local))
    {
        return;
    }

    var local_trans = Component_get_transform(local);
    var local_pos = Transform_get_position_ex(local_trans, true);

    var players = PlayerManager_get_players(mgr);

    // 3. Цикл по игрокам (kv - key/value пара)
    for (kv : players.range()) 
    {
        var player = kv.second; // Получаем указатель из Map

        if (player == local) 
        {
            continue;
        } 

        // --- ВАЖНО: Работа с Transform ---
        // Получаем Internal Transform компонента
        var trans = Component_get_transform(player);
        if (!is_valid(trans))
        {
            continue;
        } 

        // Для Internal передаем true вторым аргументом
        var pos_3d = Transform_get_position_ex(trans, true);

        // Проверка дистанции
        if (is_valid(local))
        {
            if ((pos_3d - local_pos).dist() > cfg_dist)
            {
                continue;
            } 
        }

        // --- W2S и отрисовка ---
        var pos_screen = vec3();
        if (w2s(pos_3d, pos_screen)) 
        {
            // Рисуем линию от низа экрана
            draw_line(screen_size.x / 2, screen_size.y, pos_screen.x, pos_screen.y, col_box, 1.0);
            
            // Текст над головой
            var name = "Enemy";
            var photon = Player_get_photon(player);
            if (is_valid(photon))
            {
                name = Photon_get_name(photon);
            }
            
            var t_size = calc_text_size(name, 15.0);
            draw_text(pos_screen.x - t_size.x/2, pos_screen.y - 20, name, Color(255,255,255,255), true);
        }
    }
}
```
