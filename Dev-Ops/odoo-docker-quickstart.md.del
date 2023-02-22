---
title:  odoo 快速开始
...

 
```
version: '2'
services:
  web:
    image: odoo
    depends_on:
      - db
    ports:
      - "80:8069"
  db:
    image: postgres:10
    volumes:
      # Mount for data directory
      - /var/lib/postgres:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_PASSWORD=odoo
      - POSTGRES_USER=odoo
```