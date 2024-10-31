- PostgreSQL Optimizer and planner uses table statistics for generating optimal query plans.
- Statistics generally provide information about most common values in each column in each column in a relation,
  average width of the column,number of distinct values in the column etc.
- Statistics are collected when we run analyze or when analyze is tirggered by auto vacuum and are stored in the pg_statistic
  system catalog (whose public readable view in pg_stats).
- The amount of samples considered by analyze depends on the default_statistics_target parameter.


  - Type of statistics
     - Data distribution statistics
     - Extended statisics

  - Extended statistics: Analyze commands gether and store statistics on a per-column per-table basis and therefore can't
          - capture any information about cross-column correlation.
          - It ideally treats each column individually and does not address dependencies between columns.
          - Multiple corelated columns used in a query often results in bad execution plans.
          - Normall stats will not gather relation between the columns.
          - Create statistics command can be used to create extended statistics for correlated columns.
          - When the columns are more inter related.
 
  - Controlling statistics collection
     - PostgreSQL gather and maintain table and column level statistics.
     - Statistics collection level can be controlled using:
       Alter table <table> alter column <column> set statistics <number>;
     - The number can be 1 to 10000(default value is 100)
     - A higher <number> will signal the server to gather and update more statistics but may have slow auto vacuum
     - and analyze operations on stat tables.
     - Higher number only useful for tables with large irregular data distribution.
  
  - 

