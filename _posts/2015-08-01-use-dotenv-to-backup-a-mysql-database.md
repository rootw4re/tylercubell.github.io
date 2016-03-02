---
title: 'Use Dotenv To Backup a MySQL Database'
author: Tyler
layout: post
permalink: /use-dotenv-to-backup-a-mysql-database
categories:
  - Posts
---

Shell script to create a timestamped schema-only and data-only backup of a MySQL database using a dotenv file. Schedule a cron job for an automated database backup solution.

```bash
#!/bin/bash

# Backup MySQL database using dotenv file
#
# Example:
#     ./backup_database.sh /path_to_project/.env
#
# Result:
#     Schema-only and data-only .sql files

# Keys in dotenv file required to dump MySQL data
KEYS=("DB_USERNAME" "DB_PASSWORD" "DB_NAME")

# Dotenv location
FILE=$1

# Get key value in dotenv file
val() {
    KEY=$1

    if [ ! -f "$FILE" ]; then
        echo "File does not exist"
        exit 1
    elif [ -z "$KEY" ]; then
        echo "Key not specified"
        exit 1
    fi

    echo $(eval "grep -Po '(?<=$KEY=).*' $FILE")
}

# Check that all keys have values
for i in "${KEYS[@]}"
do
    if [ -z $(val $i) ]; then
        echo "Key \"$i\" has no value"
        exit 1
    fi
done

# Dump only schema
mysqldump -u $( val ${KEYS[0]} ) \
          -p$(val ${KEYS[1]} ) \
          --no-data \
          $( val ${KEYS[2]} ) > \
          $( val ${KEYS[2]} )_schema-$(date +%F).sql

# Dump only data
mysqldump -u $( val ${KEYS[0]} ) \
          -p$( val ${KEYS[1]} ) \
          --no-create-info \
          $( val ${KEYS[2]} ) > \
          $( val ${KEYS[2]} )_data-$(date +%F).sql
```
