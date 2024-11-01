## Demo
- pg_statistic is internal table, data wont be readable.
- pg_stats is a view and data will be in readable format.
- select * from pg_stats where tablename='pgbench_tellers';
schemaname             | public   => schema name
tablename              | pgbench_tellers
attname                | tid        => Attribute/column name
inherited              | f          => not partitioned
null_frac              | 0          => no null values
avg_width              | 4          => average width of the rows
n_distinct             | -1         => no duplicates
most_common_vals       |            => duplicates
most_common_freqs      |            => 
histogram_bounds       | {1,5,10,15,20,25,30,35,40,45,50,55,60,65,         (number of bounds)
70,75,80,85,90,95,100,105,110,115,120,125,130,135,140,145,150,155,
160,165,170,175,180,185,190,195,200,205,210,215,220,225,230,235,24
0,245,250,255,260,265,270,275,280,285,290,295,300,305,310,315,320,
325,330,335,340,345,350,355,360,365,370,375,380,385,390,395,400,40
5,410,415,420,425,430,435,440,445,450,455,460,465,470,475,480,485,
490,495,500}
correlation            | 1          => if its close to 1 or plus, then the planned will go for index scan.
most_common_elems      |
most_common_elem_freqs |
elem_count_histogram   |


schemaname             | public           => Schema
tablename              | pgbench_tellers  => tablename
attname                | bid              => column
inherited              | f                => not partitioned
null_frac              | 0                => no null values
avg_width              | 4                => average length of the rows
n_distinct             | 50               => 50 duplicates
most_common_vals       | {1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,4
0,41,42,43,44,45,46,47,48,49,50}
most_common_freqs      | {0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02
,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02,0.02
,0.02}
histogram_bounds       |
correlation            | 1   => it will go for index scan and not sequencial scan.
most_common_elems      |
most_common_elem_freqs |
elem_count_histogram   |

most_common_freqs      | {0.02)  -> it is calculated as number of occurances/total rows

select count(*) from pgbench_tellers where bid=17;
-[ RECORD 1 ]
count | 10
 10/500=0.02
                                                                  ';
```
```
CREATE TABLE data_stats(a int, b int);
INSERT INTO data_stats SELECT x/100, x/1000 FROM generate_series(1,1000000) g(x);
ANALYZE VERBOSE data_stats;
set max_parallel_workers_per_gather =0;
Explain analyze select * from data_stats where a=1;
Explain analyze select * from data_stats where a=1 and b=0;
Create statistics data_stats_ext(dependencies) on a,b from data_stats;
Analyze VERBOSE data_stats;
Explain analyze select * from data_stats where a=1 and b=0;
```
