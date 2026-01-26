# hypertelevisor
api and binaries of this


# Api:

# Документация API (ChaiScript)

### 1. Основная структура скрипта
Скрипт должен содержать глобальные переменные для настроек и две обязательные функции:

```javascript
// Настройки (должны быть global, чтобы сохраняться)
global enable_esp = true;

// Вызывается для отрисовки интерфейса (ImGui)
def on_menu() {
    gui_checkbox("Enable", enable_esp);
}

// Вызывается каждый кадр для отрисовки (ESP)
def on_render() {
    if (!enable_esp) return;
    // логика...
}
```

### 2. Интерфейс (GUI)
Функции меняют значение переданной глобальной переменной.

| Функция | Описание | Пример |
| :--- | :--- | :--- |
| `gui_checkbox(label, var)` | Чекбокс (On/Off). Меняет `bool`. | `gui_checkbox("ESP", esp_enable)` |
| `gui_slider_float(label, var, min, max)` | Ползунок для дробных чисел. | `gui_slider_float("Dist", dist, 0.0, 1000.0)` |
| `gui_slider_int(label, var, min, max)` | Ползунок для целых чисел. | `gui_slider_int("Team", team, 0, 10)` |
| `gui_text(text)` | Просто текст. | `gui_text("Меню чита")` |
| `gui_button(text)` | Кнопка. Возвращает `true` при клике. | `if (gui_button("Reset")) { ... }` |

### 3. Отрисовка (Render)
Координаты экранные. Цвета задаются через `Color(r, g, b, a)`.

| Функция | Аргументы |
| :--- | :--- |
| `draw_line` | `x1, y1, x2, y2, color, thickness` |
| `draw_rect` | `x, y, w, h, color, rounding, thickness` |
| `draw_rect_filled`| `x, y, w, h, color, rounding` |
| `draw_circle` | `x, y, radius, color, segments, thickness` |
| `draw_text` | `x, y, text, color, shadow_bool` |

### 4. Работа с Игрой (Game SDK)

**Системные:**
*   `update_camera()` — **Важно:** вызывать в начале `on_render`. Обновляет матрицу для работы `w2s`.
*   `w2s(vec3_pos, vec3_out)` — Преобразует 3D координаты в 2D. Возвращает `true`, если точка на экране. Результат пишется во второй аргумент.

**Игроки (PlayerManager):**
*   `PlayerManager_get_player_manager()` — возвращает указатель на менеджер.
*   `PlayerManager_get_local_player(mgr)` — указатель на себя.
*   `PlayerManager_get_players(mgr)` — возвращает Map игроков.
    *   *Итерация:* `for(kv : players.range()) { var p = kv.second; ... }`

**Компоненты (Player/Component):**
*   `Component_get_transform(player)` — получить Transform.
*   `Transform_get_position_ex(transform, use_visual_pos)` — получить координаты (`vec3`).
*   `Player_get_photon(player)` — данные сети.
*   `Photon_get_name(photon)` — имя игрока (строка).

**Вектора (vec3):**
*   `vec3()`, `vec3(x,y,z)` — создание.
*   Доступ: `.x`, `.y`, `.z`.
*   Математика: `v1 + v2`, `v1 - v2`, `v1.dist()` (длина).
