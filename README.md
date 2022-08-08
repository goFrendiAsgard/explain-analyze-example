# How to start

```bash
docker-compose up

# in other terminal:
docker exec -it demomysql bash
mysql -u root -pAlch3mist

docker exec -it demopostgre
psql -U postgres -h postgres -d postgres -W
# password: Alch3mist

# or just use dbeaver
```

# Case Study

Compare gross revenue by film's category

# Inner Join

```sql
explain analyze select 
	category.name as category_name
	, sum(payment.amount) as payment_amount
from payment
inner join rental on rental.rental_id = payment.rental_id
inner join inventory on inventory.inventory_id = rental.inventory_id
inner join film on film.film_id = inventory.inventory_id
inner join film_category on film_category.film_id = film.film_id
inner join category on category.category_id = film_category.category_id
group by category.name
order by sum(payment.amount) desc;
```

```
-> Sort: payment_amount DESC  (actual time=16.723..16.723 rows=16 loops=1)
    -> Table scan on <temporary>  (actual time=16.694..16.699 rows=16 loops=1)
        -> Aggregate using temporary table  (actual time=16.693..16.693 rows=16 loops=1)
            -> Nested loop inner join  (cost=2587.38 rows=3464) (actual time=0.104..14.511 rows=3475 loops=1)
                -> Nested loop inner join  (cost=1374.96 rows=3455) (actual time=0.095..5.099 rows=3475 loops=1)
                    -> Nested loop inner join  (cost=785.40 rows=974) (actual time=0.088..2.706 rows=1000 loops=1)
                        -> Nested loop inner join  (cost=444.50 rows=974) (actual time=0.083..1.573 rows=1000 loops=1)
                            -> Nested loop inner join  (cost=103.60 rows=974) (actual time=0.071..0.414 rows=1000 loops=1)
                                -> Table scan on category  (cost=1.85 rows=16) (actual time=0.022..0.050 rows=16 loops=1)
                                -> Covering index lookup on film_category using fk_film_category_category (category_id=category.category_id)  (cost=0.65 rows=61) (actual time=0.012..0.020 rows=62 loops=16)
                            -> Single-row covering index lookup on film using PRIMARY (film_id=film_category.film_id)  (cost=0.25 rows=1) (actual time=0.001..0.001 rows=1 loops=1000)
                        -> Single-row covering index lookup on inventory using PRIMARY (inventory_id=film_category.film_id)  (cost=0.25 rows=1) (actual time=0.001..0.001 rows=1 loops=1000)
                    -> Covering index lookup on rental using idx_fk_inventory_id (inventory_id=film_category.film_id)  (cost=0.25 rows=4) (actual time=0.001..0.002 rows=3 loops=1000)
                -> Index lookup on payment using fk_payment_rental (rental_id=rental.rental_id)  (cost=0.25 rows=1) (actual time=0.002..0.003 rows=1 loops=3475)
```

## Left Join

```sql
explain analyze select 
	category.name as category_name
	, sum(payment.amount) as payment_amount
from payment
left join rental on rental.rental_id = payment.rental_id
left join inventory on inventory.inventory_id = rental.inventory_id
left join film on film.film_id = inventory.inventory_id
left join film_category on film_category.film_id = film.film_id
left join category on category.category_id = film_category.category_id
group by category.name
order by sum(payment.amount) desc;
```

```
-> Sort: payment_amount DESC  (actual time=79.439..79.440 rows=17 loops=1)
    -> Table scan on <temporary>  (actual time=79.413..79.415 rows=17 loops=1)
        -> Aggregate using temporary table  (actual time=79.412..79.412 rows=17 loops=1)
            -> Nested loop left join  (cost=27520.80 rows=14863) (actual time=0.090..72.300 rows=16049 loops=1)
                -> Nested loop left join  (cost=22318.75 rows=14863) (actual time=0.087..67.943 rows=16049 loops=1)
                    -> Nested loop left join  (cost=17116.70 rows=14863) (actual time=0.086..61.268 rows=16049 loops=1)
                        -> Nested loop left join  (cost=11914.65 rows=14863) (actual time=0.061..30.991 rows=16049 loops=1)
                            -> Nested loop left join  (cost=6712.60 rows=14863) (actual time=0.057..18.790 rows=16049 loops=1)
                                -> Table scan on payment  (cost=1510.55 rows=14863) (actual time=0.041..3.179 rows=16049 loops=1)
                                -> Single-row index lookup on rental using PRIMARY (rental_id=payment.rental_id)  (cost=0.25 rows=1) (actual time=0.001..0.001 rows=1 loops=16049)
                            -> Single-row covering index lookup on inventory using PRIMARY (inventory_id=rental.inventory_id)  (cost=0.25 rows=1) (actual time=0.001..0.001 rows=1 loops=16049)
                        -> Single-row covering index lookup on film using PRIMARY (film_id=inventory.inventory_id)  (cost=0.25 rows=1) (actual time=0.002..0.002 rows=0 loops=16049)
                    -> Covering index lookup on film_category using PRIMARY (film_id=film.film_id)  (cost=0.25 rows=1) (actual time=0.000..0.000 rows=0 loops=16049)
                -> Single-row index lookup on category using PRIMARY (category_id=film_category.category_id)  (cost=0.25 rows=1) (actual time=0.000..0.000 rows=0 loops=16049)

```

# Tips

- Use index
    - Primary keys
    - foreign keys
    - Searchable columns
    - But not free text
- Good table structure over advanced query/algorithm
- Understand the question
    - Start/test small components
    - Seek for alternatives
    - i.e: 
        - This query is pretty fast, but if I join it with another table, it becomes slow
        - Should I filter the table first before performing join?
- Use caching if you have to (this added complexities to your app)
    - Created/updated when you read the data
        - Better if data is rarely updated
        - Smaller cache
        - Expect inconsistencies
        - Use expiration
    - Created/update when you create/update the data
        - Better if data is frequently updated
        - Bigger cache
        - Lower inconsistencies

# Sources

- [Interpret explain analyze](https://www.cybertec-postgresql.com/en/how-to-interpret-postgresql-explain-analyze-output/)
- [Understanding explain analyze](https://www.youtube.com/watch?v=Mll5SqR4RYk)