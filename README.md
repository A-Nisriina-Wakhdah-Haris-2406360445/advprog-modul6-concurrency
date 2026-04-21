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

<details>
<Summary><b>Commit 4 Reflection notes</b></Summary>

Perbedaan fungsi `handle_connection` saat ini dengan sebelumnya terletak pada cara program menentukan response terhadap request yang diterima dari client.

Pada kode sebelumnya, penentuan proses dilakukan menggunakan blok `if-else` yang menangani kondisi satu per satu. Pada kode saat ini, penentuan response menggunakan pattern matching (`match`) sehingga lebih fleksibel dalam menangani berbagai jenis request, meningkatkan readability, dan mantainability kode, terutama jika jumlah endpoint bertambah. Selain itu, pada kode saat ini terdapat endpoint baru bernama `"/sleep"`.

Ketika client mengirimkan request ke path `/`, server akan mengembalikan halaman utama, yaitu `hello.html` dengan status 200 OK. Namun, ketika client mengirimkan request ke path `/sleep`, server akan menunda response selama 10 detik. Hal ini dikarenakan, ketika mengirimkan request ke path tersebut, program akan menjalankan `thread::sleep(Duration::from_secs(10));`, yang memiliki arti thread saat ini "ditidurkan" selama 10 detik (tidak melakukan apa-apa). Kemudian setelah 10 detik, server akan mengembalikan halaman yang sama. Duration digunakan untuk membuat objek durasi waktu (representasi waktu saja) dan thread::sleep() berfungsi untuk memberi tahu sistem operasi untuk menangguhkan thread saat ini. Selama sleep, thread tidak menggunaka CPU dan CPU bisa dipakai oleh thread lain. Setelah 10 detik, OS akan membangunkan thread dan memindahkannya ke state `Ready` dan thread akan lanjut mengeksekusi baris setelah `sleep` dijalankan. Selain itu, jika client mengirimkan request ke path diluar path yang telah didefinisikan, maka server akan mengembalikan halaman 404.html sebagai penanda bahwa path yang diinginkan tidak tersedia.

Penambahan endpoint tersebut menunjukkan adanya simulasi proses yang memakan waktu lama, sehingga menyebabkan blocking behavior. Blocking behavior merupakan kejadian ketika selama proses sleep berlangsung, suatu thread tidak dapat menangani request lain.
</details>

<details>
<Summary><b>Commit 5 Reflection notes</b></Summary>

<h1>File lib.rs</h1>
Struktur utama untuk ThreadPool adalah pub struct ThreadPool yang berisi workers dan sender. workers merupakan kumpulan thread (worker), sedangkan sender berfungsi sebagai pintu masuk untuk mengirim job ke dalam queue.

type Job merupakan fungsi anonim (closure) yang dapat dijalankan satu kali (FnOnce), aman untuk dikirim antar thread (Send), dan tidak bergantung pada scope lokal ('static).

struct Worker merepresentasikan satu thread yang akan mengambil dan mengeksekusi job dari queue.

Berikut ini adalah langkah-langkah pembuatan ThreadPool, yaitu:
1. Validasi ukuran thread
program akan memastikan jumlah thread lebih dari 0 menggunakan `assert!(size > 0);`.
2. Membuat channel (queue) menggunakan `let (sender, receiver) = mpsc::channel();`
Channel digunakan sebagai antrian job, di mana:
- `sender` digunakan untuk mengirimkan job
- `receiver` digunakan oleh worker untuk menerima job.
3. Membagikan receiver ke banyak thread menggunakan `let receiver = Arc::new(Mutex::new(receiver));`. Beberapa komponen penting;
- `Arc` (Atomic Reference Counting): Membagikan ownership antar thread. 
- `Mutex`: Mencegah race condition dengan memastikan hanya satu thread yang mengakses receiver pada satu waktu.
- `Receiver`: Tidak bisa diakses secara bersamaan oleh banyak thread, sehingga perlu dilindungi dengan Mutex
4. Membuat worker threads `workers.push(Worker::new(id, Arc::clone(&receiver)));`, di mana setiap worker merupakan thread yang berjalan terus-menerus dan memiliki akses ke queue yang sama.
5. Mengirim job menggunakan fungsi `pub fn execute<F>(&self, f: F)` dengan cara mengubah request menjadi job dan memasukkannya ke dalam queue.
6. `Worker` thread akan mengambil job dengan cara melakukan `lock()` pada `Mutex`. lalu Worker memanggil `recv()` untuk mengambil job dari queue. Jika tidak ada job, maka thread akan blocking (menunggu) tanpa menggunakan CPU. Jika ada job, maka job dijalankan `job()`. Setelah selesai, kembali ke loop untuk mengambil job berikutnya.

<h1>File main.rs</h1>

Implementasi ThreadPool pada file main.rs dilakukan dengan cara mengirimkan job ke ThreadPool melalui method `execute()`. Pada saat terdapat koneksi masuk, program akan membuat sebuah closure yang berisi pemanggilan fungsi `handle_connection(stream)`.

Closure tersebut kemudian dibungkus menjadi sebuah job dan dikirim ke dalam queue menggunakan channel (mpsc). Setelah job masuk ke dalam queue, salah satu worker thread yang tersedia akan mengambil job tersebut melalui receiver. Worker thread akan terus berjalan dalam sebuah loop, di mana ia akan:
1. Mengambil job dari queue menggunakan `recv()`
2. Menjalankan job tersebut (yaitu `handle_connection(stream)`)
3. Kembali menunggu job berikutnya
Dengan demikian, main thread hanya bertugas menerima koneksi dan mendistribusikan pekerjaan, sementara worker thread menangani request secara paralel.
</details>