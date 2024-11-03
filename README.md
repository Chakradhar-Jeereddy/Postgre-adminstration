# Postgre-adminstration
# Basic overview of the database
- Cluster - data directory and a port, a cluster can house multiple databases.
- Concurrent SQL/Connections traffic - max_connections parameter
- Multiple postgreSQL clusters can run on one server with different port and data directory, defined in the configuration file.
- Ensure it is updated in postgresql.auto.conf as it is the last file always read by cluster during startup and it takes precedence over any other config files
