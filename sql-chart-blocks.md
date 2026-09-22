# SQL chart blocks

Each chart is one `select` chart block. Stack them with `union all`. By keeping a consistent format, it allows us to select and pre-aggregate the data for many charts in a single SQL query and include information about the shape and format of each chart.

Every branch must return the same columns in the same order, however most queries will only require the basic column set. Only use more advanced columns if there is a chart that requires them in the query. Alias every column in every branch (`as chart_name`, `as chart_type`, and so on).

Usually the preferred style is to select from the tables you need, and conduct your analysis in CTEs at the top of the query and then use the chart blocks to format the chart.

Use `null` for a column a chart does not need. `segment`, `pivot_order`, and `chart_type` can stay null if needed for that chart

## Chart columns

| Column | Required | Basic | Data Type | Meaning |
| --- | --- | --- | --- | --- |
| `chart_name` | yes | yes | String | Display name. Also the id when `chart_id` is null. |
| `chart_id` |  |  | Integer | Stable id for the template. Falls back to `chart_name`. |
| `chart_type` |  | yes | String | An `XL_CHART_TYPE` name from python-pptx. Grouped columns are `COLUMN_CLUSTERED`, stacked bars are `BAR_STACKED`, and 100% stacked columns are `COLUMN_STACKED_100`. A null uses the template default. |
| `segment` |  | yes | String | Series name. |
| `segment_order` |  | yes |  | Sort order of the series. |
| `pivot` |  | yes | String | Category on the axis (month, region, and so on). |
| `pivot_order` |  | yes | Integer | Sort order of the categories. |
| `value` | yes | yes | Integer/Float | The number plotted. |
| `value_format` |  |  | String | Excel format or an f-string (`$#,##0`, `{:.1%}`). |
| `combo_series` |  |  | String | Series drawn as the other mark on a combo chart. |
| `second_value` |  |  | Integer | Series plotted on the secondary axis. |
| `loop_group_id` |  |  | Integer/Float | Charts that share an id are one loop and repeat from the same template slide. |
| `multi_axis_major` |  |  | String | Major axis label when a chart has more than one x axis. |
| `multi_axis_minor` |  |  | String | Minor axis label paired with the major. |

## Simple Example

Basic columns only. Orders over time is grouped columns, orders by store over time is 100% stacked columns, and orders by region is a stacked bar (stores stacked within each region).

```sql
with orders_rollup as (
select
    store_name
    ,region
    ,order_date
    ,count(distinct order_id) as orders
from orders
group by all
)

select
    'Orders over time' as chart_name
    ,'COLUMN_CLUSTERED' as chart_type
    ,cast(null as varchar) as segment
    ,cast(null as integer) as segment_order
    ,cast(order_date as varchar) as pivot
    ,cast(null as integer) as pivot_order
    ,sum(orders) as value
from orders_rollup
group by all

union all

select
    'Orders by store over time' as chart_name
    ,'COLUMN_STACKED_100' as chart_type
    ,store_name as segment
    ,cast(null as integer) as segment_order
    ,cast(order_date as varchar) as pivot
    ,cast(null as integer) as pivot_order
    ,sum(orders) as value
from orders_rollup
group by all

union all

select
    'Orders by region' as chart_name
    ,'BAR_STACKED' as chart_type
    ,store_name as segment
    ,cast(null as integer) as segment_order
    ,region as pivot
    ,cast(null as integer) as pivot_order
    ,sum(orders) as value
from orders_rollup
group by all
```

## Complex Example

Add the extra columns when a chart needs them. Every branch then lists the full set, in the same order.

This query uses `chart_id`, `value_format`, a combo series on `second_value`, a `loop_group_id` so each region repeats one template, and a year/month axis.

```sql
with orders_rollup as (
select
    store_name
    ,region
    ,order_date
    ,count(distinct order_id) as orders
    ,count(distinct customer_id) as customers
from orders
group by all
)

select
    'Orders and customers' as chart_name
    ,1 as chart_id
    ,'COLUMN_CLUSTERED' as chart_type
    ,cast(null as varchar) as segment
    ,cast(null as integer) as segment_order
    ,cast(order_date as varchar) as pivot
    ,cast(null as integer) as pivot_order
    ,sum(orders) as value
    ,'#,##0' as value_format
    ,'Customers' as combo_series
    ,sum(customers) as second_value
    ,cast(null as integer) as loop_group_id
    ,cast(null as varchar) as multi_axis_major
    ,cast(null as varchar) as multi_axis_minor
from orders_rollup
group by all

union all

select
    region || ' orders by store' as chart_name
    ,2 as chart_id
    ,'BAR_CLUSTERED' as chart_type
    ,cast(null as varchar) as segment
    ,cast(null as integer) as segment_order
    ,store_name as pivot
    ,cast(null as integer) as pivot_order
    ,sum(orders) as value
    ,'#,##0' as value_format
    ,cast(null as varchar) as combo_series
    ,cast(null as integer) as second_value
    ,1 as loop_group_id
    ,cast(null as varchar) as multi_axis_major
    ,cast(null as varchar) as multi_axis_minor
from orders_rollup
group by all

union all

select
    'Orders by month' as chart_name
    ,3 as chart_id
    ,'COLUMN_CLUSTERED' as chart_type
    ,cast(null as varchar) as segment
    ,cast(null as integer) as segment_order
    ,monthname(order_date) as pivot
    ,month(order_date) as pivot_order
    ,sum(orders) as value
    ,'#,##0' as value_format
    ,cast(null as varchar) as combo_series
    ,cast(null as integer) as second_value
    ,cast(null as integer) as loop_group_id
    ,cast(year(order_date) as varchar) as multi_axis_major
    ,monthname(order_date) as multi_axis_minor
from orders_rollup
group by all
```

## Tables

A table is its own column list. Use one `select`, or a separate `union all` when one query builds more than one table.

| Column | Required | Meaning |
| --- | --- | --- |
| `table_id` | yes | Which table in the deck. |
| `row_number` | yes | Row position. |
| `row_label` |  | Row header. |
| `value` | yes | Cell value. |
| `value_format` |  | Excel format or an f-string. |

## KPIs

| Column | Required | Meaning |
| --- | --- | --- |
| `kpi_id` | yes | Matched to `{{ kpi_id }}` in the deck, including titles and axis labels. |
| `value` | yes | The text or number to insert. |
| `value_format` |  | Excel format or an f-string. |
