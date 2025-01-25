# Project-Data-Analysis-for-B2B-Retail-Customer-Analytics-Report

## Introduction
xyz.com adalah perusahaan rintisan B2B yang menjual berbagai produk tidak langsung kepada end user tetapi ke bisnis/perusahaan lainnya. Sebagai data driven company, maka setiap pengambilan keputusan di xyz.com selalu berdasarkan data. Setaip quarter xyz.com akan mengadakan townhall dimana seluruh atau perwakilan divisi akan berkumpul untuk me-review performance perusahaan selama quarter terakhir.

Adapun beberapa hal yang akan direview adalah:
1. Bagaimana pertumbuhan penjualan saat ini?
2. Apakah jumlah customer xyz.com semakin bertambah?
3. Seberapa banyak customers tersebut sudah melakukan transaksi?
4. Category produk apa saja yang paling banyak dibeli oleh customers?
5. Seberapa banyak customers yang tetap aktif bertransaksi?

Ada 3 tabel yang akan digunakan pada project ini, yakni sebagai berikut:
* Tabel orders_1 : data transaksi penjualan periode quartar 1 (Jan - Mar 2004)
* Tabel orders_2 : data transaksi penjualan periode quartar 2 (Apr - Jun 2004)
* Tabel customer : data profil customer yang mendaftar menjadi customer xyz.com

![image](https://github.com/user-attachments/assets/1b9881e4-cf16-429f-8b14-c0c1c5d2c27a)

Langkah pertama yang dilakukan adalah memahami tabel untuk menentukan kolom mana yang sekiranya berkaitan dengan problem yang akan di analisa dan proses manipulasi data yang sekiranya perlu dilakukan karena tidak semua kolom akan digunakan.

```
SELECT * FROM orders_1 LIMIT 5;
SELECT * FROM orders_2 LIMIT 5;
SELECT * FROM customer LIMIT 5;
```
Output:

![image](https://github.com/user-attachments/assets/3e5e8b5c-105b-4334-965d-321a499a5480)

## Pertumbuhan Penjualan
1. **Total Penjualan dan Revenue pada Quarter 1 dan Quarter 2**

   Dari tabel orders_1 dan orders_2 dilakukan:
   * penjumlahan pada kolom _quantity_ dengan fungsi _aggregate sum()_ dan diberi nama "total_penjualan"
   * mengalikan kolom _quantity_ dengan kolom _priceeach_ kemudian menjumlahkan hasil perkalian kedua kolom dan diberi nama _"revenue"_
   * mem-filter kolom 'status' hanya dengan menampilkan order dengan status _"Shipped"_

     ```
     SELECT
       SUM(quantity) AS total_penjualan,
       SUM((quantity*priceeach)) AS revenue
     FROM orders_1
     WHERE status = "Shipped";
     SELECT
       SUM(quantity) AS total_penjualan,
       SUM((quantity*priceeach)) AS revenue
     FROM orders_2
     WHERE status = "Shipped";
     ```
     Output:

     ![image](https://github.com/user-attachments/assets/6b11801f-30cf-4cbd-9715-ca63454f00b5)
     
2. **Persentasi Keseluruhan Penjualan**

   Karena tabel orders_1 dan orders_2 masih terpisah, untuk menghitung persentasi keseluruhan penjualan dari kedua tabel tersebut perlu digabungkan
   * Pada tabel orders_1 dan orders_2, pilih kolom _"orderNumber"_, _"status"_, _"quantity"_, dan _"priceeach"_ lalu tambahkan kolom baru dengan nama _"quarter"_ dan isikan dengan  _value_ "1" untuk tabel orders_1 dan _value_ "2" untuk tabel orders_2
   * Gabungkan kedua tabel sebagai _subquery_ dengan nama alias "tabel_a"
   * Dari "tabel_a", jumlahkan kolom _"quantity"_ dengan fungsi _"aggregate sum()"_ dan diberi nama "total_penjualan"
   * Dari "tabel_a", kalikan kolom _"quantity"_ dengan kolom _"priceeach"_ kemudian jumlahkan hasil perkalian kedua kolom tersebut dan diberi nama _"revenue"_
   * _Filter_ kolom 'status' dengan hanya menampilkan order status _"Shipped"_
   * Kelompokkan "total_penjualan" berdasarkan _"quarter"_

     ```
     SELECT
       quarter,
       SUM(quantity) AS total_penjualan,
       SUM(quantity*priceeach) AS revenue
     FROM
       (SELECT orderNumber, status, quantity, priceEach, 1 AS quarter FROM orders_1
     UNION
       SELECT orderNumber, status, quantity, priceEach, 2 AS quarter FROM orders_2) AS tabel_a
     WHERE status = 'Shipped'
     GROUP BY quarter;
     ```
     Output:

     ![image](https://github.com/user-attachments/assets/1f3681e1-4758-4d64-afc1-727f1500c1b1)

3. **Growth Penjualan dan Revenue**

   Dari persentasi keseluruhan penjualan, dapat dihitung _growth_ penjualan dan _revenue_ sebagai berikut:
     * _Growth_ Penjualan

       = (total_penjualan quarter2 - total_penjualan quarter1)/total_penjualan quarter1

       = (6717 - 8694)/8694 = -22%
     * _Growth Revenue_

       = _(revenue quarter2 - revenue quarter1)/revenue quarter1_

       = (607548320 - 799579310)/799579310 = -24%

## Customer Analytics
4. **Apakah jumlah customer xyz.com semakin bertambah?**

   Penambahan jumlah customer dapat diketahui dengan membandingkan total jumlah _customer_ di periode saat ini dengan total jumlah _customer_ di akhir periode sebelumnya
   * Dari tabel customer, pilih kolom _customerID_, _createDate_, dan tambahkan kolom baru dengan fungsi _QUARTER()_ untuk mengekstrak nilai quarter dari _createDate_ dan beri nama _quarter_
   * _Filter_ kolom _"createDate"_ dengan menampilkan antara 1 Januari 2004 dan 30 Juni 2004
   * Gunakan langkah tersebut sebagai _subquery_ dengan nama alias tabel_b
   * Hitung jumlah unik _customer_ dan pastikan tidak ada duplikasi dan beri nama "total_customers"
   * Kelompokkan "total_customers" berdasarkan kolom _"quarter"_
     ```
     SELECT
       quarter,
       COUNT(DISTINCT(customerID)) AS total_customers
     FROM
       (SELECT customerID, createDate, QUARTER(createDate) AS quarter
       FROM customer
       WHERE createDate BETWEEN '2004-01-01' AND '2004-07-01') AS tabel_b
     GROUP BY quarter;
     ```
     Output:

     ![image](https://github.com/user-attachments/assets/2e6b4d52-db77-40d4-8985-49b483f69813)
	 	
5. **Seberapa banyak customers tersebut sudah melakukan transaksi?**

   * Dari tabel customer, pilih kolom _customerID_, _createDate_, dan tambahkan kolom baru dengan fungsi _QUARTER()_ untuk mengekstrak nilai quarter dari _createDate_ dan beri nama _quarter_
   * _Filter_ kolom _"createDate"_ dengan menampilkan antara 1 Januari 2004 dan 30 Juni 2004
   * Gunakan langkah tersebut sebagai _subquery_ dengan nama alias tabel_b
   * Dari tabel orders_1 dan orders_2, pilih kolom _customerID_, gunakan _DISTINCT_ untuk menghilangkan duplikasi kemudian gabungkan kedua tabel dengan _UNION_
   * Hitung jumlah unik _customers_ dan pastikan tidak ada duplikasi dan beri nama "total_customers"
   * Kelompokkan "total_customers" berdasarkan kolom _"quarter"_
     ```
     SELECT
       quarter,
       COUNT(DISTINCT(customerID)) AS total_customers
     FROM
       (SELECT customerID, createDate, QUARTER(createDate) AS quarter
         FROM customer
         WHERE createDate BETWEEN '2004-01-01' AND '2004-06-30' AND customerID IN
           (SELECT DISTINCT customerID FROM orders_1
             UNION
           SELECT DISTINCT customerID FROM orders_2)) AS tabel_b
     GROUP BY quarter;
     ```
     Output:

     ![image](https://github.com/user-attachments/assets/e59d000f-7ef9-4e2b-9ee7-ba4797a548dd)
  
6. **Category produk apa saja yang paling banyak dibeli oleh customers?**

   Untuk mengetahui produk paling banyak dibeli, dapat dilakukan dengan menghitung total order dan jumlah penjualan dari setiap kategori produk
   * Dari tabel orders_2 pilih _productCode_, _orderNumber_, _quantity_, dan _status_ kemudian tambahkan kolom baru dengan mengekstrak 3 karakter awal dari _productCode_ yang merupakan ID untuk kategori produk dan beri nama _categoryID_
   * _Filter_ kolom 'status' dengan menampilkan status "Shipped"
   * Gunakan langkah tersebut sebagai _subquery_ dengan nama alias tabel_c
   * Hitung total order dari kolom _"orderNumber"_ dan beri nama "total_order" dan jumlah penjualan dari kolom _"quantity"_ dan beri nama "total_penjualan"
   * Kelompokkan berdasarkan _categoryID_
   * Urutkan berdasrkan "total_order"
     ```
     SELECT
       LEFT(productCode,3) AS categoryID,
       COUNT(DISTINCH orderNumber) AS total_order,
       SUM(quantity) AS total_penjualan
     FROM
       (SELECT productCode, orderNumber, quantity, status
         FROM orders_2
         WHERE status = "Shipped") AS tabel_c
     GROUP BY categoryID
     ORDER BY total_order DESC
     ```
     Output:

     ![image](https://github.com/user-attachments/assets/e3fd6b04-44f3-4bfb-a43a-9f47c385c961)

8. **Seberapa banyak customers yang tetap aktif bertransaksi?**

   Mengetahui seberapa banyak _customers_ yang tetap aktif menunjukkan apakah xyz.com tetap digemari oleh _customers_ untuk memesan kebutuhan bisnis mereka. Hal ini juga dapat menjadi dasar bagi tim _product_ dan _business_ untuk pengembangan kedepannya.
   Adapun metrik yang digunakan disebut _retention cohort_. _Retention_ yang dapat dihitung adalah _retention_ dari _customers_ yang berbelanja di Quarter 1 dan kembali berbelanja di Quarter 2.
   * Dari tabel orders_1, tambahkan kolom baru dengan _value_ "1" dan beri nama _"quarter"_
   * Dari tabel orders_2, pilih kolom _customerID_, gunakan _distinct_ untuk menghilangkan duplikasi
   * _Filter_ tabel orders_1 dengan operator IN() menggunakan sehingga hanya _customerID_ yang pernah bertransaksi di quarter 2 yang diperhitungkan.
   * Hitung jumlah unik _customers_ dibagi dengan total_customers dalam _percentage_, pada Select statement dan beri nama "Q2"
   ```
   #Menghitung total unik customers yang transaksi di quarter_1
   SELECT COUNT(DISTINCT customerID) AS total_customers FROM orders_1;

   SELECT
     "1" AS quarter,
     COUNT(DISTINCT customerID)*100/25 AS Q2
   FROM orders_1
   WHERE customerID IN (SELECT DISTINCT customerID FROM orders_2);
   ```
Output:

![image](https://github.com/user-attachments/assets/f1f525c9-1831-4e7c-975f-ba721c3c37fb)


## Kesimpulan
Berdasarkan data yang telah dianalisa, dapat ditarik kesimpulan bahwa :
1. _Performance_ xyz.com menurun signifikan di _quarter_ ke-2, terlihat dari nilai penjualan dan revenue yang drop hingga 20% dan 24%.
2. Perolehan _customer_ baru juga tidak terlalu baik, dan sedikit menurun dibandingkan _quarter_ sebelumnya.
3. Ketertarikan _customer_ baru untuk berbelanja di xyz.com masih kurang, hanya sekitar 56% saja yang sudah bertransaksi. Sehingga, disarankan tim Produk untuk perlu mempelajari _behaviour customer_ dan melakukan _product improvement_, sehingga _conversion rate (register to transaction)_ dapat meningkat.
4. Produk kategori S18 dan S24 berkontribusi sekitar 50% dari total order dan 60% dari total penjualan, sehingga xyz.com sebaiknya fokus untuk pengembangan kategori S18 dan S24.
5. _Retention rate customer_ xyz.com juga sangat rendah yaitu hanya 24%, artinya banyak _customer_ yang sudah bertransaksi di _quarter-1_ tidak kembali melakukan order di _quarter ke-2 (no repeat order)_.
6. xyz.com mengalami pertumbuhan negatif di _quarter_ ke-2 dan perlu melakukan banyak _improvement_ baik itu di sisi produk dan bisnis marketing, jika ingin mencapai target dan positif _growth_ di quarter ke-3. Rendahnya _retention rate_ dan _conversion rate_ bisa menjadi diagnosa awal bahwa _customer_ tidak tertarik/kurang puas/kecewa berbelanja di xyz.com.
