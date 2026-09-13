# Supermarket-Heist-Lost-Aisles

# 🛒 Supermarket Heist: Lost Aisles

> **Co-op First-Person Extraction Horror** на движке Unreal Engine 5 (Blueprints).  
> Игроки проникают в опасный супермаркет, собирают физический лут в руки и тележки, избегают Безликих Охранников и пытаются выполнив квоту эвакуироваться к фургону.

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
4. **Стелс и Выживание:** Избегание зрения Безликих Охранников (реакция на акт кражи).
5. **События:** Реакция на «Пожарную распродажу» (ивент ИИ-диктора Майка, x1.5 к ценам на 30 сек) или отключение света.
6. **Эвакуация:** Возврат к фургону, сдача лута в зону продаж, расчет квоты ($1000 за 3 дня).

---

## 🏗️ 2. АРХИТЕКТУРА И КЛАССЫ (TECHNICAL ARCHITECTURE)

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
| `BP_StorePhaseManager` | Actor | Управление 3 секциями освещения, дверями морозилки и таймером расширения (+2 мин за ключ-карту). |

---

## 🛠️ 3. СТЕК И ИНСТРУМЕНТЫ (TOOLS & ASSETS)

* **Движок:** Unreal Engine 5.3+ (Blueprint Only)
* **Сеть:** Advanced Sessions Plugin (Steam Spacewar AppID 480)
* **3D & Модели:** Fab / UE Marketplace, Mixamo (персонажи и анимации AI)
* **Аудио:** FreeSound, ElevenLabs (голос ИИ-диктора Майка)
* **Версионный контроль:** Git + Git LFS (Обязательно для `.uasset` и `.umap`)

---

## 📁 4. СТРУКТУРА ПРОЕКТА (FOLDER STRUCTURE)

```text
Content/
 ├── Core/                # GameMode, GameState, PlayerController, Interfaces (BPI)
 ├── Characters/
 │    ├── Player/         # BP_FirstPersonCharacter, Enhanced Input Assets
 │    └── Enemies/        # BP_Guard, AI Controllers, Behavior Trees, Blackboards
 ├── Environment/
 │    ├── Architecture/   # Модульные стены, полы, потолки, двери
 │    └── Props/          # Стеллажи, кассы, холодильники, декоры
 ├── Items/
 │    ├── Data/           # Data Tables (DT_LootItems), Structures (F_LootData)
 │    └── Blueprints/     # BP_LootItem_Base, BP_ShoppingCart, BP_Keycard
 ├── UI/                  # WBP_HUD, WBP_Inventory, WBP_ResultsScreen
 ├── Audio/               # SFX, Эмбиент, Звуки Майка
 └── Maps/                # Test_Bench (Dev map), ProductMarket_Main (MVP Level)
