# Risk Rules — формулы расчёта риска, объёма, комиссий и свопа

Все расчёты делай **явно**, показывай цифры. Никаких «примерно».

## Обозначения

| Символ | Смысл |
|---|---|
| `D` | Депозит (по умолчанию $6900) |
| `R_pct` | Риск на сделку = 3% |
| `risk_$` | Допустимый риск в деньгах = `D × 0.03` = $207 |
| `entry` | Цена входа (за 1 акцию) |
| `SL` | Цена стоп-лосса |
| `TP` | Цена тейк-профита |
| `qty` | Объём в акциях (аналог кол-ва лотов) |
| `position_size` | Стоимость позиции в деньгах = `qty × entry` |
| `mult` | Мультипликатор = 3 |
| `margin` | Маржа = `position_size / mult` |

## Шаг 1. Риск в деньгах

```
risk_$ = D × 0.03
```

## Шаг 2. Расстояние до стопа

```
sl_dist = |entry − SL|
sl_dist_pct = sl_dist / entry
```

## Шаг 3. Объём от риска (с учётом издержек!)

> ⚠️ Нельзя брать `qty = risk_$ / sl_dist` «в лоб» — тогда комиссии и своп добавятся к убытку и риск превысит 3%. Сначала посчитай издержки на 1 акцию.

```
cost_comm_unit  = entry × rate/1000 × coef × 2        # открытие + закрытие
cost_swap_unit  = entry × |swap_buy|/1000 × N_nights   # своп за срок
cost_per_unit   = cost_comm_unit + cost_swap_unit

qty            = risk_$ / (sl_dist + cost_per_unit)
position_size  = qty × entry
```

> Проверка: `sl_dist × qty + cost_per_unit × qty ≈ risk_$` (должно быть ≤). При необходимости уменьши `qty`.

## Шаг 4. Комиссия открытия

Из `open_commission`:

```
rate = open_commission.rate_per_1000_of_volume         # например 0.03
coef = coef_by_volume, где from_volume <= position_size < to_volume
commission_open = position_size × rate / 1000 × coef
```

Пример: `position_size = 70 000`, `rate = 0.03`, `coef = 1.0` → `commission = 70000 × 0.03/1000 = 2.1 $`.

> Закрытие позиции рассчитывай по той же ставке (в Libertex обычно берётся и при закрытии — уточняй у пользователя, если важно; по умолчанию учитывай комиссию дважды: открытие + закрытие).

## Шаг 5. Своп (buy) за срок удержания

Из `swap`:

```
night_rate = |swap.buy_rate_per_1000_of_volume|      # например 0.512
swap_total = position_size × night_rate / 1000 × N_nights
```

`N_nights` — число **ночей**, когда своп реально списывается, по `times_applied_by_night`:

| День недели | Множитель ночей |
|---|---|
| пн–чт | 1 |
| пятница | **3** |
| сб, вс | 0 |

Пример для удержания «открыли в чт, держим 10 дней»: считай по календарю, суммируя `times_applied_by_night` по каждой ночи.

> **Важно:** своп отрицательный для покупок (снимается с баланса) — это издержка, уменьшающая чистую прибыль. Всегда вычитай.

## Шаг 6. Проверка риска (не превышает 3%)

Полный риск = убыток по стопу + комиссии + своп:

```
total_risk = sl_dist × qty + commission_open + commission_close + swap_total
```

Требование: `total_risk ≤ risk_$` ($207).

Если `total_risk > risk_$` — **уменьши `qty`** так, чтобы выполнялось (пересчитай комиссию/своп от нового `position_size`). Удобно решать итеративно или по формуле:

```
qty_target = (risk_$ − estimated_costs) / sl_dist
```

## Шаг 7. Целевая прибыль (TP = 2 × SL)

```
tp_dist = 2 × sl_dist
TP = entry + tp_dist
```

Приблизительная чистая прибыль:

```
gross_profit = tp_dist × qty
net_profit   = gross_profit − commission_open − commission_close − swap_total
```

Требование: `net_profit` положительная и ≈ в 2 раза больше чистого риска.

## Шаг 8. Проверка маржи (хватает ли средств)

```
margin = position_size / 3
margin_required = margin × 1.20       # запас 20% на просадку/свопы
```

Требование: `margin_required ≤ доступные свободные средства`.

Если не хватает — уменьши объём (и тогда риск станет **меньше** 3%, это допустимо) или откажись от сделки. **Не увеличивай риск сверх 3%.**

## Шаг 9. Сводка в план

В итоговой таблице покажи: `qty`, `position_size`, `margin`, `entry`, `SL`, `TP`, `RR`, `commission_open`, `commission_close`, `swap_total`, `net_profit`, `total_risk` (должно быть ≤ $207) и `margin_required` (должно вписываться в свободные средства).

## Контроль stop-out

`stop_out_levels` из спецификации (например `stop_out_level: 20%`, `margin_call: 50/55%`) — при x3 и риске 3% стоп-аут не должен срабатывать раньше SL. Если расстояние до SL в марже близко к критическому — уменьши объём.
