---
title: Maps Tree Component Demo
---



```sql dgrid
  select
      date_part('year', order_datetime) as year,
      date_part('month', order_datetime) as month,
      category
  from needful_things.orders
  group by all
  order by all
```

<!-- DataTable data={dgrid} / -->



```sql orders_by_tree
  select 
      category,
      date_part('year', order_datetime) as year,
      date_part('month', order_datetime) as month,
      sum(sales) as sales_usd
  from needful_things.orders
  where category = any(array${inputs.tree.category})
  and date_part('year', order_datetime) = any(array${inputs.tree.year})
  group by all
  order by sales_usd desc
```


<Grid cols=2>
  <MapsTree treeData={dgrid} columns={['category','year']} title="Maps Tree" name="tree"/>
  <DataTable data={orders_by_tree} />
</Grid>

