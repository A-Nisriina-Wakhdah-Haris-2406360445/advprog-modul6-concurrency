Nama: Nisriina Wakhdah Haris<br>
NPM: 2406360445<br>
Kelas: A<br>

<details>
<Summary><b>Commit 1 Reflection notes</b></Summary>

Fungsi `handle_connection` digunakan untuk menerima koneksi TCP dan membaca request HTTP menggunakan `BufReader`. Pembacaan dilakukan hingga ditemukan baris kosong yang menandakan akhir dari request header HTTP. Setelah itu, data yang telah dibaca disimpan dalam bentuk vector dan ditampilkan ke console. Berikut ini adalah alurnya, yaitu:

1. Parameter berupa `mut stream: TcpStream` masuk, yang artinya fungsi menerima koneksi dari client. Keyword `mut` digunakan karena stream akan dibaca (posisi read akan berubah). `TcpStream` merupakan representasi koneksi antara client dan server.
2. `stream` kemudian dibungkus dengan `BufReader` agar dapat membaca data melalui mekanisme buffering sehingga lebih efisien dan cepat. Jika tidak menggunakan `BufReader`, maka proses pembacaan data akan lebih lambat karena membaca datanya per byte.
3. Selanjutnya, membaca data perbaris (`buf_reader.lines()`) yang akan menghasilkan sebuah iterator bertipe `Iterator<Item = Result<String, Error>>`, di mana setiap elemen dapat menghasilkan Ok(String) jika sukses dan Error jika gagal.
4. Setelah itu, melakukkan mapping (`unwrap, yaitu .map(|result| result.unwrap())`) yang berfungsi untuk mengubah `Result<String, Error>` menjadi String dengan cara mengambil nilai Ok. 
5. Lalu, program akan berhenti membaca data ketika terdapat baris kosong menggunakan `take_while(|line| !line.is_empty())`. Hal ini penting karena dalam protokol HTTP, baris kosong menandai akhir dari header request.
6. Data yang telah dibaca akan di kumpulkan ke dalam sebuah vector bertipe `Vec<String>` menggunakan `.collect()`.
7. Hasil request akan ditampilkan ke console menggunakan pretty debug format `{:#?}`.
</details>

<details>
<Summary><b>Commit 2 Reflection notes</b></Summary>

Fungsi `handle_connection` digunakan untuk menerima dan membaca HTTP request dari client, menyiapkan HTTP response berupa status dan file hello.html, dan mengirim response kembali ke client.

1. Fungsi ini menerima koneksi berupa input stream yang bertipe `TcpStream`. `TcpStream` merupakan representasi koneksi TCP antara client (browser) dan server.
2. Input stream tersebut kemudian dibungkus menggunakan `BufReader`. Dengan menggunakan `BufReader`, proses pembacaan data bisa berjalan lebih efisien karena data dibuffer terlebih dahulu sebelum dibaca oleh program.
3. Kemudian, program akan membaca HTTP request dari client perbaris menggunakan `.lines()`. Setiap baris yang dibaca berupa `Result` sehingga akan di-unwrap untuk mendapatkan nilai String menggunakan `.map(|result| result.unwrap())`. Proses pembacaan dilakukan hingga menemukan baris kosong yang akan menandakan akhir dari header HTTP (`.take_while(|line| !line.is_empty())`). Seluruh baris request tersebut  kemudaian akan dikumpulkan ke dalam sebuah `Vec<String>` bernama `http_request`
4. Setelah itu, program akan menentukan response, di mana jika request berhasil, program akan mengembalikan status `"HTTP/1.1 200 OK"`
5. Lalu, program akan membaca isi file hello.html menggunakan `fs::read_to_string` dan akan menyimpannya sebagai bentuk `String`.
6. Program kemudian akan menghitung panjang konten menggunakan `contents.len()`. Nilai ini akan digunakan sebagai header `Content-Length` yang diperlukan browser untuk mengetahui jumlah byte yang harus dibaca.
7. Setelah menghitung `content_length`, program akan menyusun HTTP response yang kemudian akan dikirimkan ke client melalui `TcpStream` menggunakan `write_all`. Response tersebut akan dikonversi ke dalam bentuk bytes menggunakan `.as_bytes()`.
8. Jika response sudah diterima, browser akan memproses dan menampilkan isi hello.html ke user.

Perbedaannya dengan kode yang lama adalah:
1. Kode lama hanya menampilkan isi request ke terminal (bersifat debugging), sedangkan pada kode baru, server tidak hanya membaca request, tetapi juga mengirimkan response sehingga halaman HTML dapay ditampilkan di browser.
2. Pada kode baru, response HTTP disusun secara lengkap dibandingkan pada kode lama.
3. Pada kode lama, browser tidak menerima response sehingga akan terus loading atau timeout. Sedangkan pada kode baru, browser akan menerima response berupa isi file HTML.

Berikut ini merupakan tampilan browse ketika program dijalankan:
![Commit 2 screen capture](/images/commit2.png)
</details>

<details>
<Summary><b>Commit 3 Reflection notes</b></Summary>

Cara Split between response adalah dengan memisahkan proses penentuan response dengan proses pembuatan dan pengiriman response. Response tidak langsung dibuat sekaligus, melainkan dipecah menjadi dua tahap, yaitu:
1. Menentukan response
Pada tahap ini, program menentukan status HTTP dan file apa yang akan dikirimkan ke client sebagai response dari request yang diterima.
2. Membuat dan mengirim response
Setelah menentukan response apa yang akan dikirim, program akan membaca file, menghitung panjang data, menyusun format HTTP, lalu mengirimkannya ke client.

Refactoring ini diperlukan untuk mengurangi duplikasi kode pada proses pembuatan HTTP response dan membuat kode sesuai dengan prinsip DRY. Hal ini dikarenakan, pada kode sebelumnya, setiap kondisi memiliki blok kode yang identik untuk membaca file, menghitung panjang konten, menyusun HTTP response, dan mengirim response. Selain itu, refactoring diperlukan untuk meningkatkan readability kode, mempermudah maintanance (misalnya jika nanti ingin mengubah format response), serta memudahkan pengembangan fitur di masa depan.

Berikut ini adalah tampilan browser ketika kode dijalankan:
![Commit 3 screen capture](/images/commit3.png)
</details>