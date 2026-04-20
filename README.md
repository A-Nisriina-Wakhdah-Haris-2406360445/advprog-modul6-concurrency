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