1 Qyery statement processing.
  - For each client connection a user process is created in database
  - When a client fires a query it goes through various stages 
    - Parser/Analyzer stages
      - In parser stage the query syntax is validated if syntax is ok, it creates a parsed tree
      - In Analyzer takes parsed tree as input, it checks user has access to table, table exists, correct ness of data format and creates query tree.
      - The query tree become input to next stage Rule System to check for rules in system catalog, transform query and apply rules
      - Output of Rule system is rewritten query tree.
      - This rewritten query tree become input for the optimizer, based on the stats available and system resources it generates plan.
           1) Possible access path
           2) Best execution plan
      - The query plan it generates is taken as input for execution stage. In this stage query plan is executed and rows are fetched.
      - Data is sent to user.

2) High level steps

   - Parse: Check Syntax, break query in tokens, Generate Parse tree and identify query type.
   - Optimizer/Planner: Generates optimal plan, uses database statistics, calculate query cost, choose best plan.
   - Execute: Execute Query based on execution plan.
