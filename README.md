# ParadiseFoodOrderApp_README

# Paradise Food Order App – Public Documentation

Това е публичната документация за вътрешното приложение **Paradise Food Order App** (система за поръчка на храна).

## Достъп до оригиналния код
Оригиналното репо е **private**. Ако искаш достъп – пиши ми.

Достъп — [https://nekrato.com/menu/](https://nekrato.com/menu/)

🔒 **Private repo:** *(изисква одобрение)*  
📩 Контакт: [veslin.lilov@gmail.com](mailto:veslin.lilov@gmail.com) • GitHub: [@vegaFlex](https://github.com/vegaFlex)

---

## Съдържание
1. [Функционалности](#функционалности)  
2. [Инсталация и старт](#инсталация-и-старт)  
3. [Основни модули](#основни-модули)  
4. [Админ панел](#админ-панел)  
5. [Модели (данни)](#модели-данни)  
6. [JavaScript / UX](#javascript--ux)  
7. [URL-и (примерни)](#url-и-примерни)  
8. [Технологии](#технологии)  
9. [Бележки за деплой](#бележки-за-деплой)  
10. [Контакт и поддръжка](#контакт-и-поддръжка)

---

## Функционалности

- **Вход без имейл**: потребител се идентифицира с **име, фамилия + последни 4 цифри от ЕГН** и **локация** (Бухово / Яна / Офис).
- **Меню по дати**: ястия с дата на наличност; потребителят вижда и поръчва според датите.
- **Количка с бройки**: добавяне на ястия с количество; брояч в хедъра.
- **Поръчка**: съхранение на поръчки с разбивка на ястия и дати (чрез `OrderMeal`).
- **Отказ на ястие**: частичен отказ (брой отказани порции) и цялостен отказ.
- **Дневен лимит**: глобална настройка за **макс. сума/ден** (singleton модел `AppSetting`).
- **Админ справки**:
  - Месечна справка по потребител/период
  - Дневна обобщена справка (по локация/дата)
  - Списък с отказани ястия
  - Дневни ястия по потребител
  - Експорт за кетъринг (обобщение по ястия)
- **По-удобен админ**: autocomplete/търсене, datepicker-и, бутони към справки, custom index, визуализации на дата+ден към всяко ястие в поръчка.
- **Контекстови процесори**: текущ потребител в шаблоните и брояч на количката.

---

## Инсталация и старт

> Пример за локална разработка с Python 3.11+.

### 1) Клониране
```bash
git clone https://github.com/vegaFlex/paradise-food-order-app-public-docs.git
cd paradise-food-order-app
```

### 2) Виртуална среда
```bash
python -m venv venv
# Linux/macOS:
source venv/bin/activate
# Windows (PowerShell):
venv\Scripts\Activate.ps1
```

### 3) Зависимости
```bash
pip install -r requirements.txt
```

### 4) Миграции
```bash
python manage.py makemigrations
python manage.py migrate
```

### 5) Суперпотребител
```bash
python manage.py createsuperuser
```

### 6) Старт
```bash
python manage.py runserver
```

Достъп: локално — [http://127.0.0.1:8000/](http://127.0.0.1:8000/) • продукция — [https://nekrato.com/menu/](https://nekrato.com/menu/)
 
Админ: http://127.0.0.1:8000/admin/


---

## Основни модули

### 1) Идентификация (без имейл)
- **`orders/forms.py → IDForm`**: полета *Име*, *Фамилия*, *Последни 4 от ЕГН*, *Място* (choices от `UserID.location`).
- **`orders/context_processors.py`**: `add_user_to_context` слага текущия `UserID` в шаблоните.

### 2) Меню и поръчки
- **`Meal`**: име, описание, **`date_available`**, цена, **`discount_percent`**.
- **`Order`** + **`OrderMeal` (through)**: всяко ястие в поръчка пази **`meal_date`**, **`quantity`**, както и отказани бройки/статус.

### 3) Количка
- Сесийно съхранение: новият формат е масив от обекти:  
  `[{ "meal_id": <int>, "quantity": <int> }, ...]`.
- **`cart_count`** (context processor) сумира количествата независимо от формат.

### 4) Ограничения и правила
- **Дневен лимит** чрез `AppSetting.daily_limit`.
- Логика за **дата на ястие** в поръчката: `OrderMeal.meal_date` се попълва и показва в админа (форматирана дата + ден).

---

## Админ панел

Достъп: `/admin/`

### Разширения
- **Custom admin URLs** (в `orders/admin_urls.py`):
  - `orders/monthly-summary/` → `monthly_summary_view`
  - `orders/daily-summary/`   → `daily_summary_view`
  - `orders/cancelled-meals/` → `cancelled_meals_view`
  - `orders/daily-user-meals/`→ `daily_user_meals_view`
  - `orders/app-settings/`    → `app_settings_view` (singleton за дневния лимит)
- **UserIDAdmin**: бутони „📊 Месечна справка“ и „📜 История“.
- **OrderAdmin**:
  - Inline `OrderMeal` с визуално поле „Дата и ден“
  - Колона „Ястия + Дата“ (формат „Име (ДД.ММ.ГГГГ – ден) × бр.“)
  - **Екшън** „📤 Експортирай обобщена заявка към кетъринга“ (plain text файл с обобщение по ястия)
- **CancelledOrderMealAdmin**:
  - Показва само отказани записи (`cancelled_quantity > 0`)
  - Колони: потребител, място, ястие+дата, отказани бройки, дата/час на поръчката
- **MealAdmin**:
  - `MealAdminForm` с валидации (ограничение за дубликати по нормализирано име)
  - jQuery UI datepicker за дати
  - Търсене по нормализирано име (casefold)

---

## Модели (данни)

- **`UserID`**: `first_name`, `last_name`, `last_4_digits`, `location` (choices: `buhovo`, `yana`, `office`).  
  `unique_together = (first_name, last_name, last_4_digits, location)`.

- **`Meal`**: `name`, `description`, `date_available`, `price`, `discount_percent`.

- **`Order`**: `user (FK UserID)`, M2M към `Meal` чрез `OrderMeal`, `order_time`, `total_price`.

- **`OrderMeal` (through)**:  
  `order (FK)`, `meal (FK)`, `meal_date`, `quantity`,  
  `cancelled_quantity`, `is_cancelled`; помощни методи:  
  - `get_discounted_price()` – единична цена с отстъпка  
  - `get_total_price()` – общо с отстъпка  
  - `get_total_without_discount()` – общо без отстъпка

- **`CancelledOrderMeal`**: proxy към `OrderMeal` за админа.

- **`AppSetting`** (singleton): `daily_limit (Decimal)` + `get_daily_limit()` клас метод.

---

## JavaScript / UX

- Динамична количка и брояч в хедъра (session cart).  
- Тостове/съобщения при успешно действие (добавяне/отказ).  
- Datepicker-и и по-удобна навигация в админа.

---

## URL-и (примерни)

Публични:
- `/` – начало / вход  
- `/menu` – меню по дати  
- `/cart` – количка  
- `/submit_order` – финализиране поръчка  
- `/history` – история (за текущ потребител; за админ – по `user_id`)

Админски (през `/admin/…`):
- `orders/monthly-summary/`  
- `orders/daily-summary/`  
- `orders/cancelled-meals/`  
- `orders/daily-user-meals/`  
- `orders/app-settings/`

> Имената може да се различават според `urls.py` в конкретната инсталация.

---

## Технологии

- **Backend:** Django  
- **DB:** SQLite (dev) / PostgreSQL (prod)  
- **Frontend:** HTML/TailwindCSS + базов JS  
- **Админ:** Django Admin с custom темплейти/JS/CSS  
- **Деплой:** Passenger (SuperHosting)

---

## Бележки за деплой

- Пусни миграции и рестартирай приложението след деплой.  
- При Passenger (SuperHosting):  
  ```bash
  git fetch origin
  git reset --hard origin/main
  touch tmp/restart.txt
  ```
- Провери статични файлове и `ALLOWED_HOSTS`.

---

## Контакт и поддръжка

Автор: **Веселин Лилов** — GitHub: [@vegaFlex](https://github.com/vegaFlex)  
За въпроси: **vegaFlex@github.com**, **veslin.lilov@gmail.com**

---

### Бележка
Това репо съдържа **само публична документация**. Кодът е в private репозиториум и се предоставя при нужда.
