# Scripts pour l'analyse des ventes de produits

## Introduction

Ce document contient les scripts MongoDB nécessaires pour répondre aux questions de l'analyse des ventes de produits pour une chaîne de magasins.

## 1. Nombre total de transactions

```js
db.sales.countDocuments();
```

## 2. Vente total par jour

```js
db.sales.aggregate([
  {
    $group: {
      _id: { $dateToString: { format: "%Y-%m-%d", date: "$date" } },
      totalSales: { $sum: "$total_amount" },
    },
  },
  { $sort: { _id: 1 } },
]);
```

## 3. Ventes totales par produit :

```js
db.sales.aggregate([
  {
    $group: {
      _id: "$product_id",
      totalSales: { $sum: "$total_amount" },
    },
  },
  { $sort: { totalSales: -1 } },
]);
```

## 4. Top 5 des produits les plus vendus :

```js
db.sales.aggregate([
  {
    $group: {
      _id: "$product_id",
      count: { $sum: 1 },
    },
  },
  { $sort: { count: -1 } },
  { $limit: 5 },
]);
```

## 5. Revenu moyen par transaction :

```js
db.sales.aggregate([
  {
    $group: {
      _id: null,
      averageRevenue: { $avg: "$total_amount" },
    },
  },
]);
```

## 6. Nombre de clients uniques :

```js
db.sales.distinct("customer_id").length;
```

## 7. Répartition des ventes par magasin :

```js
db.sales.aggregate([
  {
    $group: {
      _id: "$store_id",
      totalSales: { $sum: "$total_amount" },
    },
  },
  { $sort: { totalSales: -1 } },
]);
```

## 8. Écart type des montants des transactions :

```js
db.sales.aggregate([
  {
    $group: {
      _id: null,
      stdDevAmount: { $stdDevPop: "$total_amount" },
    },
  },
]);
```

## 9. Distribution des quantités vendues par produit :

```js
db.sales.aggregate([
  {
    $group: {
      _id: "$product_id",
      quantities: { $push: "$quantity" },
      avgQuantity: { $avg: "$quantity" },
      stdDevQuantity: { $stdDevPop: "$quantity" },
    },
  },
]);
```

## 10. Médiane des ventes par magasin :

```js
db.sales.aggregate([
  {
    $group: {
      _id: "$store_id",
      sales: { $push: "$total_amount" }
    }
  },
  {
    $project: {
      _id: 1,
      sales: { $sortArray: { input: "$sales", sortBy: 1 } }, 
      count: { $size: "$sales" }
    }
  },
  {
    $project: {
      _id: 1,
      sales: 1,
      count: 1,
      median: {
        $cond: {
          if: { $eq: [{ $mod: ["$count", 2] }, 0] },
          then: {
            $avg: [
              { $arrayElemAt: ["$sales", { $divide: ["$count", 2] }] },
              { $arrayElemAt: ["$sales", { $subtract: [{ $divide: ["$count", 2] }, 1] }] }
            ]
          },
          else: { $arrayElemAt: ["$sales", { $floor: { $divide: ["$count", 2] } }] }
        }
      }
    }
  }
])

```
