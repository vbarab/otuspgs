
```
# option or PGDATA environment variable, represented here as ConfigDir.

data_directory = '/var/lib/postgresql/15/main'          # use data in another directory
                                        # (change requires restart)
hba_file = '/etc/postgresql/15/main/pg_hba.conf'        # host-based authentication file
                                        # (change requires restart)
ident_file = '/etc/postgresql/15/main/pg_ident.conf'    # ident configuration file
                                        # (change requires restart)

# If external_pid_file is not explicitly set, no extra PID file is written.
external_pid_file = '/var/run/postgresql/15-main.pid'                   # write an extra PID file
                                        # (change requires restart)


```
