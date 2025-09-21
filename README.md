# Microservice-based Application
> Aplikasi ini adalah aplikasi yang digunakan untuk menjelaskan komunikasi yang event-driven di antara 2 service. Satu service menangani request terkait product, sementara service yang lain menangani order. 

## order-service
Service ini menerima order dari user, menyimpan order ke database, dan mengirim notifikasi order ke product-service melalui RabbitMQ

### Komponen Utama
- `main.go`: Inisialisasi server dan dependensi
- `handlers/order.go`: Handler HTTP untuk endpoint order (misal: POST /order)
- `models/order.go`: Model data untuk order
- `repository/order_repository.go`: Layer akses data ke database untuk order
- `services/product_client.go`: Client untuk komunikasi ke product-service (jika ada kebutuhan direct call).
- `queue/rabbitmq.go`: Implementasi publish event ke RabbitMQ (misal: order.created).
- `cache/redis.go`: Implementasi cache menggunakan Redis.
- `order_schema.sql`: Skema database untuk tabel order.

## product-service
Service ini mendengarkan event order dari RabbitMQ, mengurangi stok produk jika order diterima, dan menyediakan API produk.

### Komponen Utama
- `src/main.ts`: Entry point
- `src/app.module.ts`: Modul utama aplikasi.
- `src/products/`: Modul produk, berisi:
`products.controller.ts` yang berupa Endpoint HTTP untuk produk (misal: GET/POST produk). <br>
`products.service.ts`: Bisnis logic produk (misal: update stok). <br>
`products.listener.ts`: Listener RabbitMQ untuk event order (misal: order.created). <br>
`product.entity.ts`: Entity/model produk untuk ORM/database. <br>
`products.module.ts`: Modul NestJS untuk produk.
- `src/redis/redis.service.ts`: Service untuk caching produk dengan Redis (opsional).
- `product_schema.sql`: Skema database untuk tabel produk.

## Komunikasi Antar Service
order-service -> RabbitMQ -> product-service
order-service publish event order ke RabbitMQ.
product-service subscribe event order dari RabbitMQ, lalu update stok produk.

## How to Run Locally
You must have installed docker on your computer. Here is how you run the program
```
docker-compose up
```
and
```
docker-compose down -v
```
to stop the program

After the services and the dependencies already started, you can do the request with command

```
Invoke-RestMethod -Uri http://localhost:3000/products -Method POST -Body <<product>> -ContentType "application/json"  # to create a product

curl http://localhost:3000/products/:id #to see detail of product

curl.exe -X POST http://localhost:3001/orders -H "Content-Type: application/json" -d <<request body>> # to create an order

curl.exe http://localhost:3001/orders/product/:id #to see detail of order
```

## Example API Request
```
Invoke-RestMethod -Uri http://localhost:3000/products -Method POST -Body '{"name":"Laptop Lenovo","price":15000000,"qty":10}' -ContentType "application/json"  

curl http://localhost:3000/products/1 

curl.exe -X POST http://localhost:3001/orders -H "Content-Type: application/json" -d '{\"productId\":1}' 

curl.exe http://localhost:3001/orders/product/1 
```
