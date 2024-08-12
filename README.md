## Script SQL untuk Laporan Penjualan Bulanan

Kode SQL ini digunakan untuk membuat laporan penjualan bulanan, yang menggabungkan data penjualan berdasarkan produk. Laporan ini mengidentifikasi produk terlaris setiap bulannya.

```sql
WITH monthly_sales AS (
    SELECT
        EXTRACT(YEAR FROM orders.created_at) AS year,
        EXTRACT(MONTH FROM orders.created_at) AS month,
        products.id AS product_id,
        products.name AS product_name,
        products.category AS product_category,
        products.brand AS product_brand,
        COUNT(order_items.id) AS total_quantity_sold,
        SUM(order_items.sale_price) AS total_sales_amount
    FROM
        `bigquery-public-data.thelook_ecommerce.orders` AS orders
    JOIN
        `bigquery-public-data.thelook_ecommerce.order_items` AS order_items
        ON orders.order_id = order_items.order_id
    JOIN
        `bigquery-public-data.thelook_ecommerce.products` AS products
        ON order_items.product_id = products.id
    WHERE
        orders.status = 'Complete'
    GROUP BY
        year, month, product_id, product_name, product_category, product_brand
)
SELECT
    year,
    month,
    product_id,
    product_name,
    product_category,
    product_brand,
    total_quantity_sold,
    total_sales_amount
FROM
    monthly_sales
ORDER BY
    year, month, total_sales_amount DESC;
```

## Penjelasan

- **WITH `monthly_sales` AS:** Bagian ini mendefinisikan sebuah Common Table Expression (CTE) bernama `monthly_sales`. CTE ini berfungsi sebagai subquery yang menyimpan hasil sementara dari query yang berada di dalamnya. CTE ini akan digunakan dalam query utama untuk melakukan analisis penjualan bulanan berdasarkan produk.
  
- **SELECT:**
  - **`EXTRACT(YEAR FROM orders.created_at) AS year:`** Query ini mengekstrak tahun dari kolom `created_at` pada tabel `orders` dan memberikan alias `year`. Ini digunakan untuk mengelompokkan data penjualan berdasarkan tahun.
  - **`EXTRACT(MONTH FROM orders.created_at) AS month:`** Query ini mengekstrak bulan dari kolom `created_at` pada tabel `orders` dan memberikan alias `month`. Ini digunakan untuk mengelompokkan data penjualan berdasarkan bulan.
  - **`products.id AS product_id:`** Query ini memilih kolom `id` dari tabel `products` dan memberikan alias `product_id`. Kolom ini digunakan untuk mengidentifikasi produk yang terjual.
  - **`products.name AS product_name:`** Query ini memilih kolom `name` dari tabel `products` dan memberikan alias `product_name`. Kolom ini menunjukkan nama dari produk yang terjual.
  - **`products.category AS product_category:`** Query ini memilih kolom `category` dari tabel `products` dan memberikan alias `product_category`. Ini menunjukkan kategori dari produk yang terjual.
  - **`products.brand AS product_brand:`** Query ini memilih kolom `brand` dari tabel `products` dan memberikan alias `product_brand`. Ini menunjukkan merek dari produk yang terjual.
  - **`COUNT(order_items.id) AS total_quantity_sold:`** Query ini menghitung jumlah baris (produk yang terjual) dalam tabel `order_items` untuk setiap kombinasi produk dan bulan, kemudian memberikan alias `total_quantity_sold`. Ini mengindikasikan total kuantitas produk yang terjual dalam satu bulan.
  - **`SUM(order_items.sale_price) AS total_sales_amount:`** Query ini menjumlahkan harga jual (`sale_price`) dari setiap item yang terjual untuk setiap kombinasi produk dan bulan, kemudian memberikan alias `total_sales_amount`. Ini mengindikasikan total nilai penjualan dari produk tersebut dalam satu bulan.

- **FROM:**
  - **`bigquery-public-data.thelook_ecommerce.orders AS orders:`** Query ini mengakses data dari tabel `orders` yang berada dalam dataset `thelook_ecommerce` di koleksi dataset publik `bigquery-public-data` milik Google BigQuery. Tabel ini berisi informasi tentang setiap pesanan yang dilakukan di platform e-commerce.
  - **`JOIN bigquery-public-data.thelook_ecommerce.order_items AS order_items ON orders.order_id = order_items.order_id:`** Query ini menggabungkan tabel `orders` dengan tabel `order_items` berdasarkan `order_id` yang sama. Tabel `order_items` berisi detail dari setiap item yang ada dalam pesanan.
  - **`JOIN bigquery-public-data.thelook_ecommerce.products AS products ON order_items.product_id = products.id:`** Query ini menggabungkan tabel `order_items` dengan tabel `products` berdasarkan `product_id` yang sama. Tabel `products` berisi informasi mengenai produk yang dijual.

- **WHERE:**
  - **`orders.status = 'Complete':`** Query ini memfilter hanya pesanan yang memiliki status 'Complete', artinya hanya pesanan yang telah selesai diproses yang akan dimasukkan dalam perhitungan.

- **GROUP BY:**
  - **`year, month, product_id, product_name, product_category, product_brand:`** Query ini mengelompokkan hasil berdasarkan kombinasi dari `year`, `month`, `product_id`, `product_name`, `product_category`, dan `product_brand`. Hal ini memungkinkan untuk mendapatkan total penjualan dan kuantitas terjual per produk per bulan.

- **SELECT:**
  - **`year, month, product_id, product_name, product_category, product_brand, total_quantity_sold, total_sales_amount:`** Query utama memilih semua kolom dari CTE `monthly_sales`. Hasilnya adalah daftar produk yang dijual beserta detail penjualan untuk setiap bulan.

- **ORDER BY:**
  - **`year, month, total_sales_amount DESC:`** Query ini mengurutkan hasil akhir berdasarkan `year`, `month`, dan `total_sales_amount` dalam urutan menurun. Ini berarti produk dengan penjualan tertinggi dalam setiap bulan akan ditampilkan paling atas.
