# api-gateway
bash ```
version: "3.5"
services:
  kong-database:
    image: postgres:9.6.8-alpine
    hostname: kong-database
    container_name: kong-database
    restart: always
    environment:
      POSTGRES_USER: "kong"
      POSTGRES_DB: "kong"
    networks:
      - "kong-net"

  kong-migration:
    image: kong:1.1.2
    container_name: kong-migration
    restart: on-failure
    depends_on:
      - kong-database
    environment:
      KONG_DATABASE: "postgres"
      KONG_PG_HOST: "kong-database"
      KONG_CASSANDRA_CONTACT_POINTS: "kong-database"
      KONG_PG_USER: "kong"
      KONG_PG_PASSWORD: "kong"
    command: kong migrations bootstrap
    networks:
      - "kong-net"
  kong:
    image: kong:1.1.2
    hostname: kong
    container_name: kong
    restart: always
    ports:
      - "80:8000"
      - "443:8443"
      - "8001:8001"
      - "8444:8444"
    environment:
      KONG_DATABASE: "postgres"
      KONG_PG_HOST: "kong-database"
      KONG_CASSANDRA_CONTACT_POINTS: "kong-database"
      KONG_PROXY_ACCESS_LOG: "/dev/stdout"
      KONG_ADMIN_ACCESS_LOG: "/dev/stdout"
      KONG_PROXY_ERROR_LOG: "/dev/stderr"
      KONG_ADMIN_ERROR_LOG: "/dev/stderr"
      KONG_ADMIN_LISTEN: "0.0.0.0:8001, 0.0.0.0:8444 ssl"
    depends_on:
      - kong-migration
      - kong-database
    networks:
      - "kong-net"

  konga:
    image: pantsel/konga:0.14.1
    hostname: konga
    container_name: konga
    restart: always
    ports:
      - "1337:1337"
    environment:
      TOKEN_SECRET: "abc123"
      NODE_ENV: "production"
    networks:
      - "kong-net"

networks:
  kong-net:
    name: kong-net
    driver: bridge

```
