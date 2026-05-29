# ChemAI: Predict the Cure (Команда #20)

По 210 молекулярным RDKit-дескрипторам предсказать IC50, CC50 и SI для 250 тестовых молекул. Метрика - среднее RMSE по трём таргетам.

**Ноутбук воспроизводит: LB ~278** — простой ансамбль из 4 моделей (ExtraTrees + LightGBM + XGBoost + CatBoost), равные веса, log1p только для SI.

**[Презентация проекта](https://docs.google.com/presentation/d/1njf9kHm3Q9d_ZIsu_WLK7MtnycCdus3q5o0ToknkWHM/edit?usp=sharing)** (Google Slides)

---

## Задача

- **IC50 (mM)**: концентрация, подавляющая 50% активности вируса гриппа
- **CC50 (mM)**: токсичная концентрация для 50% клеток
- **SI = CC50/IC50**: индекс селективности

---

## Данные

`data/` в `.gitignore` (закрытые лабораторные данные) - положить CSV в `data/raw/` локально перед запуском:

```
data/raw/train.csv             - 751 объект, IC50/CC50/SI
data/raw/test.csv              - 250 объектов без таргетов
data/raw/sample_submission.csv - формат сабмита
```

210 числовых RDKit-дескрипторов: молекулярные свойства (MolWt, TPSA, MolLogP), электронные (PEOE_VSA*), топологические (Chi*, BertzCT), поверхностные (SMR_VSA*, EState_VSA*), функциональные группы (fr_*) и BCUT2D_*.

---

## Прогресс на Kaggle

| Шаг | LB |
|---|---|
| Baseline (log1p, SLSQP-бленд) | 316 |
| + Rank-Gauss target | 307 |
| Расширенный SLSQP-бленд из 55 моделей (секции 11-16b) | 294.67 |
| **Простой ансамбль 4 моделей (секция 16c)** | **~278** |
| + Псевдо-разметка IC50 (CB pw=1.0, спец. якорь, скрипты вне ноутбука) | 274.96 |
| + Псевдо-разметка CC50 (CB pw=3.0, скрипты вне ноутбука) | **274.09** |

---

## Запуск

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

```bash
jupyter lab notebook.ipynb
# Run All (~25 мин холодный старт, ~5 мин с кешем oof/)
# Выход: submission.csv (LB ~278, простой ансамбль из секции 16c)
```

### Воспроизводимость

- SEED=42 для всех источников случайности
- KFold(5, shuffle=True, random_state=42) один для всех моделей
- CatBoost детерминирован при фиксированном random_seed
