# Supermarket-Heist-Lost-Aisles
# 🛒 Supermarket Heist: Lost Aisles

> **Co-op First-Person Extraction Horror** на движке Unreal Engine 5 (Blueprints).  
> Игроки проникают в опасный супермаркет, собирают физический лут в руки и тележки, избегают Безликих Охранников и пытаются выполнить квоту, эвакуировавшись к фургону.

---

## 📌 1. ОБЗОР ПРОЕКТА (GAME OVERVIEW)

* **Жанр:** Co-op Extraction Horror / Heist
* **Движок:** Unreal Engine 5 (v5.3 / v5.4) — **Только Blueprints**
* **Платформа:** PC (Steam)
* **Режим игры:** Кооператив на 1–4 игроков (Listen Server via Steam / Advanced Sessions)
* **Целевой ориентир:** *Lethal Company*, *Phasmophobia*, *Backrooms: Escape Together*

### 🔄 Основной игровой цикл (Core Loop)
1. **Лобби / Фургон:** Спавн игроков, закупка расходников в Терминале за счет квоты.
2. **Заход в магазин:** Старт 5-минутного таймера текущего Дня.
3. **Сбор добычи:** Поиск товаров, сортировка по весу (легкий / тяжелый), складывание в тележку.
4. **Стелс и Выживание:** Избегание зрения Безликих Охранников (реакция на факт кражи товара на глазах).
5. **События:** Реакция на «Пожарную распродажу» (ивент роботизированного диктора **Canya**, x1.5 к ценам на 30 сек) или отключение света.
6. **Эвакуация:** Возврат к фургону, сдача лута в зону продаж, расчет квоты ($1000 за 3 дня).

---

## 👥 2. РАСПРЕДЕЛЕНИЕ РОЛЕЙ И ЗАДАЧ В КОМАНДЕ

Чтобы не путаться, кто чем занимается при разработке:

* **Blueprint / Tech Lead (Программирование логики):**
  * Создание актеров `BP_FirstPersonCharacter`, `BP_LootItem_Base`, `BP_ShoppingCart`.
  * Настройка сетевой репликации (Server RPC, GameMode, GameState).
  * Настройка логики касс, таймеров и диктора Canya.

* **Level / Environment Designer (Работа с картой):**
  * Сборка локации `Product Market` из модульных блоков (стены, полы, кассы, стеллажи).
  * Настройка NavMesh Bounds Volume (чтобы AI охранник мог ходить).
  * Настройка освещения (Lumen, RectLights, выключатели).

* **3D / Prop / Asset Artist (Ассеты и материалы):**
  * Поиск, импорт и подгонка 3D-моделей товаров в `DT_LootItems`.
  * Настройка коллизий у предметов (Box / Convex Collision) для физики.
  * Создание/настройка UI-иконки товаров, материалов и текселя.

* **Sound & Game Designer (Звуки и Баланс):**
  * Нарезка и озвучка реплик диктора **Canya** (ElevenLabs / свой голос).
  * Подбор звуков шагов, шуршания, падения коробок и эмбиента магазина.
  * Заполнение цен, веса и редкости товаров в Data Table.

---

## 🏗️ 3. АРХИТЕКТУРА И КЛАССЫ (TECHNICAL ARCHITECTURE)

| Класс / Ассет | Тип | Назначение и Зона ответственности |
| :--- | :--- | :--- |
| `BP_HeistGameMode` | Game Mode Base | **Server Authority:** Контроль квоты ($1000), подсчет дней (3 дня), лимит времени (300 сек/день). |
| `BP_HeistGameState` | Game State Base | **Replication:** Синхронизация общего счета, таймера и статуса ивентов на всех клиентов. |
| `BP_FirstPersonCharacter` | Character | Движение (Enhanced Input), Physics Handle (подбор физических тел), инвентарь (1–2 слота). |
| `BP_LootItem_Base` | Actor | Единый базовый предмет. Считывает параметры и Static Mesh из `DT_LootItems` при спавне. |
| `F_LootData` | Structure | Поля: `ItemName` (Text), `Price` (Float), `WeightType` (Enum: Light/Heavy), `Mesh` (StaticMesh), `Sound` (SoundBase). |
| `DT_LootItems` | Data Table | Единая база данных всех товаров магазина на основе структуры `F_LootData`. |
| `BP_LootDropOff` | Actor | Зона продажи у фургона. Триггер забирает предмет, уничтожает его и начисляет деньги в GameState. |
| `BP_ShoppingCart` | Actor / Pawn | Физическая тележка с виртуальными слотами (`AttachToComponent`) для транспортировки тяжелого лута. |
| `BP_GuardAIController` | AI Controller | Поведение охраны: Behavior Tree + AI Perception (Sight). Реагирует на проверку `IsHoldingLoot`. |
| `BP_StorePhaseManager` | Actor | Управление 3 секциями освещения и логикой диктора **Canya** (Fire Sale). |

---

## 📁 4. СТРУКТУРА ПАПОК В UNREAL ENGINE (`/Content/`)

```text
Content/
 ├── Core/                # GameMode, GameState, PlayerController, BPI_Interactable
 ├── Characters/
 │    ├── Player/         # BP


cter, Enhanced Input Assets
 │    └── Enemies/        # BP_Guard (Faceless Security), AI Controller, Behavior Tree
 ├── Environment/
 │    ├── Architecture/   # Модульные стены, полы, потолки, кассовые зоны
 │    └── Props/          # Стеллажи, холодильники, декоры, мусор
 ├── Items/
 │    ├── Data/           # Data Tables (DT_LootItems), Structures (F_LootData)
 │    └── Blueprints/     # BP_LootItem_Base, BP_ShoppingCart
 ├── UI/                  # WBP_HUD, WBP_Inventory, WBP_ResultsScreen
 ├── Audio/               # SFX, Эмбиент, Голос диктора Canya (Fire Sale Alerts)
 └── Maps/                # Test_Bench (Dev map), ProductMarket_Main (MVP Level)_FirstPersonChara
