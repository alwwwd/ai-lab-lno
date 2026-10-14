# Pandas — шпаргалка

Короткий справочник по операциям, которые нужны в заданиях и пригодятся при подготовке данных для машинного обучения.

## 1. Загрузка данных

```python
import pandas as pd

data = pd.read_csv(
    "school_progress.csv",
    index_col="enrollment_id"
)
```

- `pd.read_csv(...)` — загружает CSV-файл в `DataFrame`.
- `index_col="enrollment_id"` — использует столбец `enrollment_id` как подписи строк.
- `parse_dates=["start_date", "end_date"]` — сразу преобразует указанные столбцы в даты.

---

## 2. DataFrame и Series

- **DataFrame** — таблица из строк и столбцов.
- **Series** — один столбец таблицы.

```python
type(data)                         # DataFrame
scores = data["average_score"]
type(scores)                       # Series
```

Один столбец в одинарных скобках → `Series`:

```python
data["course"]
```

Несколько столбцов в двойных скобках → `DataFrame`:

```python
data[["course", "average_score"]]
```

---

## 3. Быстрый просмотр таблицы

```python
data.head()        # первые 5 строк
data.head(10)      # первые 10 строк
data.tail()        # последние 5 строк

data.shape         # (число строк, число столбцов)
data.size          # общее число ячеек
data.columns       # названия столбцов
data.index         # индекс строк
data.dtypes        # тип каждого столбца
data.info()        # краткая сводка по таблице
```

Для одного Series:

```python
scores.dtype       # тип данных
scores.size        # число элементов
scores.shape       # размерность
scores.name        # имя Series
```

---

## 4. Выбор строк и столбцов

### `loc` — выбор по подписи индекса

```python
data.loc[10025]
data.loc[[10025, 10050]]
data.loc[10020:10030]

data.loc[
    [10025, 10050],
    ["student_name", "course", "average_score"]
]
```

### `iloc` — выбор по номеру позиции

```python
data.iloc[0]          # первая строка
data.iloc[:10]        # первые 10 строк
data.iloc[[0, 10, 20]]
data.iloc[:10, :6]    # первые 10 строк и 6 столбцов
```

### Одна ячейка

```python
data.at[10025, "average_score"]   # по индексам
data.iat[0, 3]                    # по позициям
```

---

## 5. Статистика Series

```python
scores.count()       # число непустых значений
scores.isna().sum()  # число пропусков
scores.nunique()     # число уникальных значений
scores.unique()      # сами уникальные значения

scores.mean()        # среднее
scores.median()      # медиана
scores.std()         # стандартное отклонение
scores.min()         # минимум
scores.max()         # максимум
scores.describe()    # основные статистики сразу
```

### Минимальные и максимальные значения

```python
scores.nsmallest(5)  # 5 минимальных
scores.nlargest(5)   # 5 максимальных
```

---

## 6. Векторные операции

Операция над `Series` выполняется сразу для всех строк.

```python
data["hours_spent"] * 60
```

Новый признак можно вычислить из нескольких столбцов:

```python
data["progress_percent"] = (
    data["lessons_completed"]
    / data["total_lessons"]
    * 100
)
```

Стандартизация:

```python
z_scores = (
    scores - scores.mean()
) / scores.std()
```

---

## 7. Сортировка

```python
data.sort_values("average_score")
```

По убыванию:

```python
data.sort_values(
    "average_score",
    ascending=False
)
```

По нескольким столбцам:

```python
data.sort_values(
    ["course", "average_score"],
    ascending=[True, False]
)
```

По индексу:

```python
data.sort_index()
```

---

## 8. Частоты категорий

```python
data["course"].value_counts()
```

Показывает, сколько раз встречается каждое значение.

Доли:

```python
data["course"].value_counts(normalize=True)
```

Проценты:

```python
data["course"].value_counts(normalize=True) * 100
```

---

## 9. Фильтрация

Сравнение столбца создаёт булеву маску из `True` и `False`.

```python
mask = data["average_score"] >= 80
data[mask]
```

### Несколько условий

`&` — И:

```python
data[
    (data["grade"] >= 8)
    & (data["average_score"] >= 75)
]
```

`|` — ИЛИ:

```python
data[
    (data["status"] == "active")
    | (data["status"] == "completed")
]
```

`~` — НЕ:

```python
data[~(data["status"] == "dropped")]
```

### Несколько допустимых значений

```python
data[
    data["course"].isin([
        "Python",
        "Анализ данных"
    ])
]
```

### Диапазон

```python
data[data["grade"].between(8, 11)]
```

---

## 10. Пропуски

Проверка:

```python
data.isna().sum()
data["average_score"].isna()
data["average_score"].notna()
```

Удалить строки с пропуском в нужном столбце:

```python
data.dropna(subset=["average_score"])
```

Заполнить пропуски:

```python
data["comment"] = (
    data["comment"]
    .fillna("Нет данных")
)
```

`NaN` — пропущенное обычное значение.  
`NaT` — пропущенная дата.

---

## 11. Дубликаты

Найти полные дубликаты:

```python
data.duplicated()
data.duplicated().sum()
```

Найти повторы по выбранным столбцам:

```python
data.duplicated(
    subset=["student_id", "course"],
    keep=False
)
```

Удалить повторы:

```python
data.drop_duplicates(
    subset=["student_id", "course"]
)
```

Оставить запись с лучшим баллом:

```python
clean = (
    data
    .sort_values("average_score", ascending=False)
    .drop_duplicates(
        subset=["student_id", "course"]
    )
)
```

---

## 12. Работа с текстом: `.str`

`.str` позволяет применять строковые операции сразу ко всему столбцу.

### Удаление лишних пробелов

```python
data["comment"].str.strip()
```

### Регистр

```python
s.str.lower()       # нижний регистр
s.str.upper()       # верхний регистр
s.str.title()       # каждое слово с большой буквы
```

### Поиск текста

```python
s.str.contains(
    "проект",
    case=False,
    na=False
)
```

- `case=False` — не учитывать регистр.
- `na=False` — пропуски считать `False`.

Начало/конец строки:

```python
s.str.startswith("Мария", na=False)
s.str.endswith("@school.example", na=False)
```

### Замена

```python
s.str.replace("старое", "новое", regex=False)
```

### Длина строки

```python
s.str.len()
```

### Разделение строки

```python
s.str.split("|")
```

Разделить сразу в несколько столбцов:

```python
data[["login", "domain"]] = (
    data["email"]
    .str.split("@", n=1, expand=True)
)
```

---

## 13. `apply()`

`apply()` вызывает функцию для каждого значения Series.

```python
def score_group(score):
    if pd.isna(score):
        return "нет данных"
    if score >= 85:
        return "высокий"
    if score >= 70:
        return "средний"
    return "низкий"

levels = data["average_score"].apply(score_group)
```

Если задачу можно решить обычной арифметикой над столбцом, лучше использовать векторную операцию:

```python
# лучше
minutes = data["hours_spent"] * 60
```

---

## 14. `GroupBy`

`groupby()` разбивает строки на группы по значениям столбца.

```python
groups = data.groupby("course")
```

Размер групп:

```python
groups.size()
```

Средний балл в каждой группе:

```python
data.groupby("course")["average_score"].mean()
```

Группировка по нескольким столбцам:

```python
data.groupby(["grade", "course"]).size()
```

### Несколько показателей сразу: `agg()`

```python
report = data.groupby("course").agg(
    records=("student_id", "size"),
    students=("student_id", "nunique"),
    mean_score=("average_score", "mean"),
    mean_hours=("hours_spent", "mean")
)
```

Каждая строка результата — одна группа.

---

## 15. Объединение таблиц: `merge()`

`merge()` соединяет строки двух таблиц по общему ключу.

### Left join

Сохраняет все строки левой таблицы и добавляет найденные данные из правой.

```python
result = left.merge(
    right,
    how="left",
    on="student_id"
)
```

### Inner join

Оставляет только ключи, которые есть в обеих таблицах.

```python
result = left.merge(
    right,
    how="inner",
    on="student_id"
)
```

### Outer join

Сохраняет все ключи из обеих таблиц.

```python
result = left.merge(
    right,
    how="outer",
    on="student_id",
    indicator=True
)
```

Столбец `_merge` покажет источник строки:

```text
left_only
right_only
both
```

Если ключи называются по-разному:

```python
left.merge(
    right,
    left_on="student_id",
    right_on="id"
)
```

---

## 16. Конкатенация: `concat()`

Добавить строки одной таблицы к другой:

```python
pd.concat([df1, df2], ignore_index=True)
```

Склеить таблицы по столбцам:

```python
pd.concat([left, right], axis="columns")
```

`concat()` просто склеивает таблицы.  
`merge()` ищет совпадения по ключу.

---

## 17. Сводная таблица: `pivot_table()`

`pivot_table()` превращает значения категорий в строки/столбцы и одновременно агрегирует данные.

```python
scores_wide = data.pivot_table(
    index="student_id",
    columns="course",
    values="average_score",
    aggfunc="mean"
)
```

Здесь:

- `index` — что станет строками;
- `columns` — что станет столбцами;
- `values` — какие значения помещаются внутрь;
- `aggfunc` — что делать, если для одной ячейки найдено несколько строк.

---

## 18. Wide ↔ Long

### `melt()` — широкая таблица → длинная

```python
long = wide.melt(
    id_vars="student_id",
    var_name="course",
    value_name="score"
)
```

### `explode()` — список в ячейке → несколько строк

```python
clubs = data.copy()
clubs["clubs"] = clubs["clubs"].str.split("|")
clubs = clubs.explode("clubs")
```

---

## 19. Даты и время

Преобразовать текст в дату:

```python
data["start_date"] = pd.to_datetime(
    data["start_date"]
)
```

Для Series с датами используется `.dt`.

```python
data["start_date"].dt.year        # год
data["start_date"].dt.month       # номер месяца
data["start_date"].dt.day         # день месяца
data["start_date"].dt.dayofweek   # 0=понедельник ... 6=воскресенье
data["start_date"].dt.day_name()  # название дня недели
```

Разница двух дат даёт длительность:

```python
duration = (
    data["end_date"]
    - data["start_date"]
)
```

Количество дней:

```python
data["duration_days"] = duration.dt.days
```

Фиксированная дата:

```python
pd.Timestamp("2026-09-01")
```

---

## 20. Экспорт и повторная загрузка

### CSV

```python
data.to_csv("result.csv", index=False)
check = pd.read_csv("result.csv")
```

### JSON

```python
data.to_json(
    "result.json",
    orient="records"
)

check = pd.read_json("result.json")
```

### Excel

```python
with pd.ExcelWriter("report.xlsx") as writer:
    data.to_excel(
        writer,
        sheet_name="Data",
        index=False
    )
```

Чтение:

```python
pd.read_excel("report.xlsx")
```

---

## 21. Мини-чеклист перед машинным обучением

```python
data.shape

data.dtypes

data.isna().sum()

data.duplicated().sum()
```

Проверьте:

1. одна строка соответствует одному наблюдению;
2. целевой столбец не содержит лишних пропусков;
3. числовые признаки действительно числовые;
4. категории и текст обработаны осознанно;
5. нет случайных дубликатов;
6. признаки не содержат информацию из будущего относительно момента прогноза.
