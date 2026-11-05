# Tool Map — привязка шагов к инструментам Libertex MCP

Используй именно эти инструменты; не выдумывай символы и поля.

| Шаг | Инструмент | Что берём |
|---|---|---|
| Каталог типов | `Libertex__list_instrument_types` | наличие типа `stock` |
| Поиск акции | `Libertex__search_instruments` (`types:["stock"]`) | символы и названия |
| Проверка символа | `Libertex__get_instruments` | валидность тикера |
| Условия контракта | `Libertex__get_instrument_specification` | комиссия, своп, мультипликатор, мин/макс объём, stop-out, дивиденды, `next_session_start` |
| Текущая цена | `Libertex__get_current_quotes` | `rate`, `spread`, `is_market_open`, `timestamp` |
| История/тренд | `Libertex__get_candles_history` | OHLCV (D1/h4/m30) для SMA/EMA/ATR и уровней |
| Новости | `Libertex__get_latest_news` | заголовки по символу |
| Текст новости | `Libertex__get_news_text` | суть новости |
| Макро-календарь | `Libertex__get_economic_calendar` | важные события в горизонте ≤10 дней |

## Важные нюансы полей

- `get_candles_history`: `interval` **чувствителен к регистру** (`m1` минуты, `M1` — месяц), `limit` ≤ 100 → для 200+ свечей D1 делай несколько запросов по датам.
- `get_current_quotes`: до 30 символов за вызов.
- `get_news_text`: до 5 id за вызов.
- `get_economic_calendar`: диапазон дат ≤ 30 дней, по умолчанию только `high` важность.

## Семантика спецификации

- `open_commission.rate_per_1000_of_volume` — ставка за $1000 объёма; умножать на `coef` из `coef_by_volume` по объёму.
- `swap.buy_rate_per_1000_of_volume` — отрицательный для покупок (издержка).
- `swap.times_applied_by_night` — множитель списания по дням (пятница = 3).
- `mult_settings.values` — доступные мультипликаторы (x3 должен быть в списке).
- `stop_out_levels` — уровень принудительного закрытия и margin call.
