---
toc: false
title: Problem with site name "Silver"
---

```js
const sites_list = [
   {
     name: "Test1",
	 short: "test1",
   },
   {
     name: "Silver",
	 short: "test2",
   },
   {
     name: "Test3",
	 short: "test3",
   },
   {
     name: "Test4",
	 short: "test4",
   },
];
```

```js
const sites = view(
  Inputs.checkbox(
    sites_list,
    {
      value: sites_list, 
      label: "Site",
      format: (t) => t.name,
    }
  )
);
```

```js
// Timestamps are in milliseconds
const now = 1730065000000
const begin = 1730040000000

function s2q(site) {
  const short = site.short;
  const sitename = site.name;
  // Note #1. When I remove the 'X' from '${sitename}X' below, then the
  // sitename, which is "Silver" in one case, somehow causes trouble.
  // Without the 'X', the site name is also a color name.  Below, we use
  // the SITE column as the stroke, but the result when the SITE is "Silver"
  // causes: (a) the site does not appear in the color legend, (b) the 
  // corresponding mark is grey.
  return `SELECT '${sitename}X' as SITE, "VarA" as VARA, "VarB" as VARB, "VarC" as VARC, Timestamp*1000 as UTC from ${short} where UTC >= ${begin}`;
}
```

```js
var duck = await DuckDBClient.of({
    test1: FileAttachment("./data/test1.csv").csv({typed: true}),
    test2: FileAttachment("./data/test2.csv").csv({typed: true}),
    test3: FileAttachment("./data/test3.csv").csv({typed: true}),
    test4: FileAttachment("./data/test4.csv").csv({typed: true}),
});

```

```js
const bysite = sites.map(site => {
   const q = s2q(site);
   return duck.query(q);
});
const results = await Promise.all(bysite);
```

```js
// Question #2: Why does this Inputs.table() not display anything?
console.log(results[0]);
Inputs.table(results[0]);
```

```js
var plot1 = results.map(result => {
	return Plot.lineY(result, {x: "UTC", y: "VARA", stroke: "SITE"});
});
var plot2 = results.map(result => {
	return Plot.lineY(result, {x: "UTC", y: "VARB", stroke: "SITE"});
});
var plot3 = results.map(result => {
	return Plot.lineY(result, {x: "UTC", y: "VARC", stroke: "SITE"});
});
```

<div class="grid grid-cols-1">
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Plot 1",
      width,
	  x: {grid: true, type: "time", label: "Date", domain: [begin, now]},
      y: {grid: true, label: "Unit1", domain: [0, 10]},
      color: {legend: true},
      marks: plot1
    }))
  }</div>
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Plot 2",
      width,
	  x: {grid: true, type: "time", label: "Date", domain: [begin, now]},
      y: {grid: true, label: "Unit2", domain: [0, 10]},
      color: {legend: true},
      marks: plot2
    }))
  }</div>
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Plot 3",
      width,
	  x: {grid: true, type: "time", label: "Date", domain: [begin, now]},
      y: {grid: true, label: "Unit3", domain: [0, 10]},
      color: {legend: true},
      marks: plot3
    }))
  }</div>
</div>

