# OBS Studio Barcode Generator Script

## Опис

Цей скрипт для OBS Studio дозволяє відображати штрихкод на вашій сцені. Ви можете налаштовувати дані для штрихкоду, його ширину та висоту безпосередньо через інтерфейс налаштувань OBS Studio.

## Вимоги

- **OBS Studio** (версія з підтримкою Python скриптів)
- **Python 3** встановлений на вашій системі
- **Pillow** бібліотека для Python

## Встановлення

### 1. Встановлення Python та необхідних бібліотек

Переконайтеся, що у вас встановлений Python 3. Якщо ні, встановіть його через ваш менеджер пакетів. Наприклад, для Debian/Ubuntu:

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

### 2. Створення віртуального середовища

Рекомендується створити віртуальне середовище для ізоляції пакетів.

```bash
python3 -m venv ~/obs_barcode_env
```

### 3. Активуйте віртуальне середовище

```bash
source ~/obs_barcode_env/bin/activate
```

### 4. Встановлення бібліотеки Pillow

```bash
pip install Pillow
```

## Налаштування скрипта в OBS Studio

### 1. Завантажте скрипт

Скопіюйте наступний код у файл `barcode_generator.py`:

```python
import obspython as obs
from PIL import Image, ImageDraw

# Параметри (ці значення будуть змінюватися в OBS)
barcode_data = ""
barcode_width = 300
barcode_height = 100

# Функція для оновлення штрихкоду
def update_barcode():
    global barcode_data, barcode_width, barcode_height

    # Перевірка, чи введені дані
    if not barcode_data:
        return

    # Створення зображення штрихкоду
    img = Image.new("1", (barcode_width, barcode_height), "white")
    draw = ImageDraw.Draw(img)

    # Генерація простого штрихкоду зі смугами
    x = 10
    for index, char in enumerate(barcode_data):
        if char.isdigit() and int(char) % 2 == 0:  # кожна цифра генерує чорну або білу смугу
            draw.rectangle([x, 0, x + 4, barcode_height], fill="black")
        x += 6  # відстань між смугами

    # Збереження штрихкоду у файл
    img.save("/tmp/barcode.png")

    # Оновлення зображення в OBS
    source = obs.obs_get_source_by_name("Barcode Image")
    if source:
        settings = obs.obs_data_create()
        obs.obs_data_set_string(settings, "file", "/tmp/barcode.png")
        obs.obs_source_update(source, settings)
        obs.obs_data_release(settings)
        obs.obs_source_release(source)

# Функція для оновлення параметрів з налаштувань OBS
def script_update(settings):
    global barcode_data, barcode_width, barcode_height
    barcode_data = obs.obs_data_get_string(settings, "barcode_data")
    barcode_width = obs.obs_data_get_int(settings, "barcode_width")
    barcode_height = obs.obs_data_get_int(settings, "barcode_height")
    update_barcode()

# Функція для створення інтерфейсу налаштувань OBS
def script_properties():
    props = obs.obs_properties_create()

    # Поле для введення даних штрихкоду
    obs.obs_properties_add_text(props, "barcode_data", "Дані для штрихкоду", obs.OBS_TEXT_DEFAULT)
    
    # Налаштування ширини та висоти штрихкоду
    obs.obs_properties_add_int(props, "barcode_width", "Ширина штрихкоду", 100, 1000, 10)
    obs.obs_properties_add_int(props, "barcode_height", "Висота штрихкоду", 50, 500, 10)

    return props

# Ініціалізація значень за замовчуванням
def script_defaults(settings):
    obs.obs_data_set_default_string(settings, "barcode_data", "1234567890")
    obs.obs_data_set_default_int(settings, "barcode_width", 300)
    obs.obs_data_set_default_int(settings, "barcode_height", 100)
```

### 2. Додавання скрипта в OBS Studio

1. **Відкрийте OBS Studio**.
2. Перейдіть до **Інструменти** → **Скрипти**.
3. Натисніть **+**, щоб додати скрипт, і виберіть файл `barcode_generator.py`.

### 3. Додавання джерела зображення

1. У вашій сцені натисніть **+** у розділі **Джерела** і виберіть **Джерело зображення**.
2. Назвіть його **Barcode Image**.
3. У налаштуваннях джерела встановіть шлях до файлу `/tmp/barcode.png` (скрипт автоматично створить цей файл).

### 4. Налаштування параметрів скрипта

1. У вікні **Скрипти** знайдіть завантажений `barcode_generator.py`.
2. Введіть необхідні дані для штрихкоду в поле **Дані для штрихкоду**.
3. Встановіть бажану **Ширину** та **Висоту** штрихкоду.
4. Скрипт автоматично оновить зображення штрихкоду у вашій сцені.

## Використання

1. **Запустіть OBS Studio** і завантажте скрипт, якщо ще не зробили цього.
2. **Налаштуйте параметри штрихкоду** через інтерфейс скрипта в OBS.
3. **Переконайтеся**, що джерело **Barcode Image** додане до вашої сцени.
4. **Використовуйте сцену** з відображеним штрихкодом у ваших трансляціях або записах.

## Налагодження

### Помилка `ModuleNotFoundError: No module named 'PIL'`

Переконайтеся, що ви встановили бібліотеку Pillow у вашому віртуальному середовищі:

```bash
pip install Pillow
```

### Створення віртуального середовища

Якщо ви ще не створили віртуальне середовище, дотримуйтесь кроків з розділу **Встановлення**.

### Переконайтеся, що OBS використовує правильний інтерпретатор Python

1. Відкрийте OBS Studio.
2. Перейдіть до **Інструменти** → **Налаштування** → **Скрипти**.
3. Переконайтеся, що шлях до інтерпретатора Python вказаний правильно (шлях до вашого віртуального середовища).

## Ліцензія

Цей проект ліцензовано за [MIT License](LICENSE).
