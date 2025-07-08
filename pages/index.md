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
      date_part('quarter', order_datetime) as quarter ,
      sum(sales) as sales_usd
  from needful_things.orders
  where ${inputs.tree}
  group by all
  order by sales_usd desc
```


<Grid cols=2>
  <MapsTree treeData={dgrid} columns={['category','year']} title="Maps Tree" name="tree"/>
  <DataTable data={orders_by_tree} />
</Grid>


<PivotTable data={orders_by_tree} columns={['category','year']} rows={['quarter','month']} metric={'sales_usd'}>
</PivotTable>

