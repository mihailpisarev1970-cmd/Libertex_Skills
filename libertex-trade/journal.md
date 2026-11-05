# Журнал сделок Libertex

Формат записей. Добавляй новую запись в начало списка (свежие сверху).
Статусы: `OPEN` / `CLOSED_TP` / `CLOSED_SL` / `CLOSED_TIME` / `CLOSED_NEWS`.

## Шаблон записи

```
### <TICKER> — <дата открытия>
- Статус: OPEN
- Direction: LONG x3
- Entry: $<entry>
- SL: $<SL> | TP: $<TP>
- Qty: <qty> акций | Position: $<position_size> | Margin: $<margin>
- Commission est.: $<commission> | Swap est.: $<swap>
- Risk: $<risk> (≤ $207)
- Причина входа: <тренд + уровень + новости>
- Дата/причина закрытия: —
- P&L: —
```

## История

_(пока пусто — первая запись появится после первого плана)_
