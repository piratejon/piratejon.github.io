---
layout: post
title:  "SQL 🏝"
date:   2019-05-22 18:57:54 -0500
categories: sql proofs
---
Sometimes in SQL you need to find sequences of consecutive identifiers in a
column. This can happen in a [SCD Type 2 table][scd-type-2], when not all
attributes may change with each new row, and you need to find the first and
last row for each consecutive run of values, as opposed to the overall first
and last row where each value occurs. One approach can be found with not much
Internet searching:
{% highlight sql %}
select
  date
  , animal
  , row_number() over (order by date)                       overall_seq
  , row_number() over (partition by animal order by date)   animal_seq
  , row_number() over (order by date)
  - row_number() over (partition by animal order by date)   island_id
from some_table
;
{% endhighlight %}

| Date | Animal | Sales | `overall_seq` | `animal_seq` | `island_id` |
|------|--------|-------|--------|--------|--------|
| 5/1  | 🐭 | $75 | 1 | 1 | 0 |
| 5/2  | 🐭 | $80 | 2 | 2 | 0 |
| 5/3 | 🐭 | $80 | 3 | 3 | 0 |
| 5/4 | 🐸 | $88 | 4 | 1 | 3 |
| 5/5 | 🐸 | $80 | 5 | 2 | 3 |
| 5/6 | 🐭 | $75 | 6 | 4 | 2 |
| 5/7 | 🦏 | $75 | 7 | 1 | 6 |
| 5/8 | 🦏 | $80 | 8 | 2 | 6 |
| 5/9 | 🦏 | $81 | 9 | 3 | 6 |
| 5/10 | 🦏 | $83 | 10 | 4 | 6 |
| 5/11 | 🦏 | $79 | 11 | 5 | 6 |
| 5/12 | 🦏 | $80 | 12 | 6 | 6 |
| 5/13 | 🐸 | $88 | 13 | 3 | 10 |
| 5/14 | 🐸 | $87 | 14 | 4 | 10 |
| 5/15 | 🐸 | $86 | 15 | 5 | 10 |
| 5/16 | 🐸 | $72 | 16 | 6 | 10 |

It looks great! It seems like `island_id` can be used to identify the different
sequences -- even different sequences of the same animal. But how can we know
for sure? How do we know it doesn't just happen to work out in this case? Can
we prove that `island_id`s are unique to their groups?

## Part 1: Each group has exactly one `island_id`

## Part 2: No two groups share an `island_id`
Res ipsa loquitor

## Conclusion
Given the members of each group all have the same `island_id`, and no two
groups can have the same `island_id`, it follows that `island_id`s uniquely
identify their associated groups.

For an encore, I am interested to explore functional language approaches to the
same problem. It is not difficult to do something similar to SQL window
functions (`row_number`, at least) in functional languages with map, apply,
fold, etc. The problem probably can likely also be tackled recursively. I
wonder if we can find an isomorphism between the window function-based approach
described here and a recursive approach one might implement in a functional
language? Perhaps the topic of a future article!

[scd-type-2]: https://en.wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row
