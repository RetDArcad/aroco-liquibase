# aroco-liquibase

Liquibase migration repository for the `aroco` PostgreSQL database
(consumed by `aroco-java` / `aroco-react`).

The migrations are run **outside** the Java application — the Spring Boot
service does not embed Liquibase. Apply them with the Liquibase CLI before
starting the backend.

## Versions

| Version | Tables                                                        | Purpose                              |
|---------|---------------------------------------------------------------|--------------------------------------|
| 1.0.0   | `countries`, `families`, `customers`, `providers`, `orders`   | Initial schema, pre-articles feature |
| 1.1.0   | `articles`, `article_desc`, `article_providers`, `order_lines` | Adds the articles catalog feature    |

Each version lives in its own changelog file under `changelog/`. The master
changelog (`changelog/changelog-master.xml`) controls which versions get
applied — `include` directives are added as features ship.

## Prerequisites

- PostgreSQL reachable on `localhost:5434` with a database named `aroco`
- Liquibase CLI 4.20+ on the `PATH`
- PostgreSQL JDBC driver (`postgresql-*.jar`) on the Liquibase classpath
  (drop it into `$LIQUIBASE_HOME/lib/` or pass `--classpath=...`)

Create the empty database once:

```sql
CREATE DATABASE aroco;
```

## Apply migrations

From the repository root:

```bash
liquibase update
```

The connection info is read from `liquibase.properties`. Override at the CLI
if needed:

```bash
liquibase --url=jdbc:postgresql://otherhost:5432/aroco \
          --username=postgres --password=*** update
```

## Useful commands

```bash
liquibase status                  # what's pending
liquibase history                 # what's been applied
liquibase update-sql              # preview the SQL without running it
liquibase rollback-count 1        # roll back the last changeset
liquibase tag v1.0.0              # tag the current state (run between versions)
```

## Layout

```
.
├── liquibase.properties              # connection + master changelog pointer
├── changelog/
│   ├── changelog-master.xml          # master, includes per-version files
│   ├── v1.0.0-initial-schema.xml     # base tables
│   └── v1.1.0-articles-feature.xml   # articles feature tables
├── .gitignore
└── README.md
```

## Adding a new version

1. Create `changelog/vX.Y.Z-<short-name>.xml` with all the new changesets.
2. Append an `<include>` line for it in `changelog-master.xml`.
3. Run `liquibase update`.
