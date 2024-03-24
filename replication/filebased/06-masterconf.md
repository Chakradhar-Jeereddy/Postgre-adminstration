## Considerations
  -  Install PostgreSQL on both the machines preferably same version.
  - Minor version differences are acceptable (ex 12.3 and 12.5).
  - Ensure we are able to ping between the servers (Master/Slave).
  - Ensure password less file transfer is happening between the servers.
  - Ensure postgres user has read/write permissions on the directory.
  - Ensure contrib module is installed on both the machines.

## Master side configuration
  - Enable archive mode on the master server.
    - Parameter : archive_mode = ON
  - Set archive command parameter to copy the wal files on the destination server.
    - Parameter : Archive_command
    - Example: rync -a %p postgres@standbyserver:/location (linux)
          - copy "%P" "\\\\192.168.1.1\\archive\\%f"
  - set archive_timeout to force the server to switch to a new WAL segment file periodically.
    - Parameter : archive_timeout = 60

