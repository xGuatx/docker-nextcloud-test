# Nextcloud Test - Docker Deployment

Docker test environment for Nextcloud file hosting and collaboration platform.

## Description

Test deployment of Nextcloud with MariaDB/PostgreSQL for development and testing purposes.

## Prerequisites

- Docker
- Docker Compose

## Installation

```bash
# Start Nextcloud stack
docker-compose up -d
```

## Usage

### First Access

1. Navigate to: `http://localhost:8080`
2. Create admin account
3. Configure database connection:
   - **Database**: MariaDB/PostgreSQL
   - **Host**: `db:3306` (MariaDB) or `db:5432` (PostgreSQL)
   - **Database name**: `nextcloud`
   - **User**: `nextcloud`
   - **Password**: (from `.env` file)

### Default Ports

- Nextcloud: `8080`
- MariaDB: `3306` (internal)

## Configuration

### docker-compose.yml Example

```yaml
services:
  nextcloud:
    image: nextcloud:latest
    ports:
      - "8080:80"
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=secure_password
    volumes:
      - nextcloud_data:/var/www/html
    depends_on:
      - db

  db:
    image: mariadb:latest
    environment:
      - MYSQL_ROOT_PASSWORD=root_password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=secure_password
    volumes:
      - db_data:/var/lib/mysql
```

### Environment Variables

Create `.env` file:
```env
# Database
MYSQL_ROOT_PASSWORD=your_root_password
MYSQL_DATABASE=nextcloud
MYSQL_USER=nextcloud
MYSQL_PASSWORD=your_db_password

# Nextcloud
NEXTCLOUD_ADMIN_USER=admin
NEXTCLOUD_ADMIN_PASSWORD=your_admin_password
NEXTCLOUD_TRUSTED_DOMAINS=localhost 192.168.1.100
```

## Features

- File storage and sharing
- Calendar and contacts
- Collaborative editing
- Mobile app support
- Extensible with apps
- User management

## Common Commands

```bash
# Start
docker-compose up -d

# Stop
docker-compose down

# View logs
docker-compose logs -f nextcloud

# Access OCC command
docker-compose exec -u www-data nextcloud php occ
```

## OCC Commands

```bash
# Scan files
docker-compose exec -u www-data nextcloud php occ files:scan --all

# List users
docker-compose exec -u www-data nextcloud php occ user:list

# Add user
docker-compose exec -u www-data nextcloud php occ user:add username

# Maintenance mode
docker-compose exec -u www-data nextcloud php occ maintenance:mode --on
docker-compose exec -u www-data nextcloud php occ maintenance:mode --off
```

## Backup & Restore

### Backup

```bash
# Stop Nextcloud
docker-compose stop nextcloud

# Backup data
docker run --rm -v nextcloud_data:/data -v $(pwd):/backup \
  alpine tar czf /backup/nextcloud-backup.tar.gz /data

# Backup database
docker-compose exec db mysqldump -u root -p nextcloud > nextcloud-db-backup.sql

# Restart
docker-compose start nextcloud
```

### Restore

```bash
# Restore data
docker run --rm -v nextcloud_data:/data -v $(pwd):/backup \
  alpine tar xzf /backup/nextcloud-backup.tar.gz -C /

# Restore database
docker-compose exec -T db mysql -u root -p nextcloud < nextcloud-db-backup.sql
```

## Apps Installation

### Via Web Interface

1. Login as admin
2. Settings -> Apps
3. Browse and install apps

### Via OCC

```bash
docker-compose exec -u www-data nextcloud php occ app:install calendar
docker-compose exec -u www-data nextcloud php occ app:enable calendar
```

## Troubleshooting

### Permission Issues

```bash
# Fix permissions
docker-compose exec nextcloud chown -R www-data:www-data /var/www/html
```

### Database Connection Error

- Verify database credentials in `.env`
- Check database container is running: `docker-compose ps`
- View database logs: `docker-compose logs db`

### Trusted Domain Error

```bash
# Add trusted domain
docker-compose exec -u www-data nextcloud php occ config:system:set \
  trusted_domains 1 --value=your.domain.com
```

### Large File Uploads

Edit `docker-compose.yml`:
```yaml
environment:
  - PHP_UPLOAD_LIMIT=10G
  - PHP_MEMORY_LIMIT=512M
```

## Security Notes

**Test environment only**:
- Use strong passwords in `.env`
- Don't expose to internet without HTTPS
- Regular backups recommended
- Update images regularly
- For production, use dedicated setup

## Performance Tuning

### Enable Redis Cache

Add to `docker-compose.yml`:
```yaml
  redis:
    image: redis:alpine

  nextcloud:
    environment:
      - REDIS_HOST=redis
```

### Database Tuning

For MariaDB, add to `docker-compose.yml`:
```yaml
  db:
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
```

## Use Cases

- Test file sync
- Develop Nextcloud apps
- Test backup/restore procedures
- Evaluate Nextcloud features
- Training environment

## Resources

- [Nextcloud Documentation](https://docs.nextcloud.com/)
- [Nextcloud Docker Image](https://hub.docker.com/_/nextcloud)
- [OCC Commands](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/occ_command.html)

## License

Personal project - Test use

---

**Note**: Nextcloud is an open-source project. This is a test deployment configuration.
