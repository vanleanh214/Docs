## HOW TO INSTALL POSTGRES MASTER-SLAVE DOCKER COMPOSE

🔥 **Build Folder Tree**

**1. FOLER IN MASTER** 

```md
POSTGRES/
   initdb.d/
	00_init.sql
   .env
   docker-compose.yml
```
**2. FOLER IN SLAVE**

```md
POSTGRES/
   .env
   docker-compose.yml
```
---

⚙️  **CONFIG**

**1. CONFIG MASTER**

*File 00_init.sql*

```bash
-- Create replication role if it doesn't exist
DO $$ BEGIN
  IF NOT EXISTS (SELECT FROM pg_catalog.pg_roles WHERE rolname = 'replica') THEN
    CREATE ROLE replica WITH REPLICATION LOGIN PASSWORD '${POSTGRES_PASSWORD}';
  END IF;
END $$;

-- Create replication slot if it doesn't exist
SELECT pg_create_physical_replication_slot('replica_slot', true); 
```
*.env*

```bash
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=postgres
```

*docker-compose.yml*

```bash
services:
  postgres_primary:
    image: postgres:17
    container_name: postgres_primary
    env_file:
      - .env
    ports:
      - "5432:5432"
    volumes:
      - postgres_primary_data:/var/lib/postgresql/data
      - ./initdb.d/00_init.sql:/docker-entrypoint-initdb.d/00_init.sql
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    command: postgres -c 'wal_level=replica' -c 'max_wal_senders=10' -c 'max_replication_slots=10' -c 'listen_addresses=*'
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5
volumes:
  postgres_primary_data:
```

**2. CONFIG SLAVE**

*docker-compose.yml*

```bash
services:  
postgres_replica:
    image: postgres:17
    container_name: postgres_replica
    env_file:
      - .env
    ports:
      - "5433:5432"
    volumes:
      - postgres_replica_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    command: >
      bash -c "
        echo 'Waiting for primary to be ready...';
        while ! pg_isready -h $HOST_MASTER -p 5432 -U ${POSTGRES_USER}; do sleep 1; done;
        echo 'Primary is ready, starting replica...';
        rm -rf /var/lib/postgresql/data/* 2>/dev/null || true;
        PGPASSWORD=${POSTGRES_PASSWORD} pg_basebackup -h postgres_primary -U replica -D /var/lib/postgresql/data -Fp -Xs -R -P;
        chown -R postgres:postgres /var/lib/postgresql/data;
        echo 'Replication setup complete, starting PostgreSQL...';
        docker-entrypoint.sh postgres
      "
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_replica_data:
```
*.env*

```bash
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=postgres 
```
> [!NOTE]
> - Thay đổi host trỏ đến IP master
> - Trên master:
>   ```sql
>   ALTER USER replica WITH PASSWORD 'postgres';
>   ```
> - Sửa file `/var/lib/postgresql/data/pg_hba.conf` trên master, thêm:
>   ```conf
>   host    replication     replica     all    scram-sha-256
>   ```
> - Reload cấu hình:
>   ```sql
>   SELECT pg_reload_conf();
>   ```

