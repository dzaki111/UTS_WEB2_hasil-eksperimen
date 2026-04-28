

# Implementasi Autentikasi dan Manajemen Konten Dinamis pada CodeIgniter 4

## 1. Pendahuluan
Dalam ekosistem pengembangan web, perlindungan terhadap integritas data merupakan tantangan yang terus berkembang. Salah satu ancaman yang paling persisten adalah manipulasi database query atau yang dikenal sebagai SQL Injection . Penguatan mekanisme keamanan pada gerbang utama aplikasi (autentikasi) menjadi sangat krusial, mengingat kerentanan pada sektor ini dapat berakibat pada kebocoran informasi yang fatal [1] . Pemanfaatan sistem login yang kokoh bukan sekedar kebutuhan teknis, melainkan fondasi dalam menjaga aset digital dari intervensi pihak luar yang tidak berwenang [2] . Artikel ini mendokumentasikan proses teknis yang saya tempuh dalam memitigasi risiko keamanan tersebut serta langkah optimalisasi artikel manajemen menggunakan platform CodeIgniter 4.

## 2. Eksperimen Teknis: Defisit Kerentanan SQL Injection
Percobaan ini difokuskan pada pengujian dan penguatan lapisan validasi pada login admin. Tujuannya adalah memastikan bahwa gerbang masuk utama aplikasi tidak dapat ditembus oleh manipulasi instruksi database ilegal.

### A. Analisis Risiko pada Query Konvensional
Secara teori, sistem autentikasi yang hanya mengandalkan pencocokan teks sederhana tanpa adanya filter tambahan sangatlah rawan terhadap serangan siber [2]. Masalah utama biasanya muncul ketika kita menggunakan teknik penggabungan variabel (konkatenasi) langsung ke dalam perintah SQL.

Penyerang dapat dengan mudah menyisipkan fragmen kode berbahaya, seperti ' OR '1'='1. Logika dibalik serangan ini adalah memaksa database untuk selalu menghasilkan nilai "Benar" (True), sehingga penyerang bisa masuk ke akun administratif meskipun mereka tidak memiliki kata sandi yang valid.

### B. Mitigasi Menggunakan Mekanisme Query Builder
Untuk menangkal ancaman tersebut, saya mengadopsi fitur Query Builder yang terintegrasi dalam CodeIgniter 4. Pendekatan ini secara mendasar memisahkan antara struktur logika perintah dengan data mentah yang diinput oleh pengguna.

Berikut adalah implementasi kode yang diterapkan:

```php
fungsi 
publik login_aman ( ) { 
    $model = UserModel baru (); 
    $namapengguna = $ini ->permintaan-> getPost ( 'nama pengguna' ); 
    $kata sandi = $ini ->permintaan-> getPost ( 'kata sandi' ); 
    // PENTING: Menggunakan 'userpassword' sesuai dengan kolom di database 
    // Menggunakan array dalam Where() otomatis mengaktifkan Prepared Statements (Anti SQL Injection) 
    $user = $model -> Where ([ 
        'username'     => $username , 
        'userpassword' => $password 
    ])-> first (); 
    
    if ( $user ) { 
        // Jika data cocok, simpan session dan masuk ke dashboard 
        $session = session (); 
        $sesi -> set ([ 
            'user_id'    => $user [ 'id' ], 
            'username'   => $user [ 'username' ], 
            'logged_in' => true 
        ]); 
        session ()-> setFlashdata ( 'success' , 'Login Berhasil! Sistem aman dari SQL Injection.' ); 
        return redirect ()-> ke ( '/admin/artikel' ); 
    } else { 
        // Jika gagal, kembali ke halaman login dengan pesan error 
        session ()-> setFlashdata ( 'error' , 'Username atau Password salah!' ); 
        return redirect ()-> back (); 
    }
}
```

**Penjelasan Proses:** Metode keamanan ini terletak pada penggunaan Pernyataan yang Disiapkan . Ketika data dikirimkan dalam format array ke fungsi Where(), framework secara otomatis melakukan sterilisasi input melalui proses binding parameter . Dengan cara ini, karakter berbahaya seperti tanda kutip tunggal tidak lagi dianggap sebagai perintah eksekusi, melainkan diperlakukan sebagai teks statistik murni yang tidak memiliki efek terhadap struktur database kueri.

## 3. Transformasi Manajemen Konten (Implementasi Praktikum 6)
Peningkatan fungsionalitas juga dilakukan pada sisi antarmuka pengelola untuk menyajikan penyajian data yang lebih informatif.

### A. Integrasi Relasional Melalui Join Table
Berdasarkan standar pengembangan pada Praktikum 6, saya melakukan rekonstruksi pada tampilan tabel artikel. Jika sistem sebelumnya hanya menampilkan kode numerik untuk kategori, kini saya menerapkan operasi JOIN untuk menghubungkan tabel artikel dan kategori. Hasilnya, dashboard mampu menyajikan nama kategori secara deskriptif dan dinamis.

```php
$ini -> artikelModel -> pilih ( 'artikel .*, kategori.nama_kategori'); 
$ini -> artikelModel -> gabung ( 'kategori ', 'kategori .id_kategori = artikel.id_kategori', 'l
```

### B. Pengaturan Navigasi dan Proteksi Jalur Akses
Guna memastikan privasi data, sistem dilengkapi dengan Filter Auth . Namun, penyesuaian khusus dilakukan pada konfigurasi Routes.php dengan menempatkan jalur login di luar jangkauan filter. Hal ini bertujuan untuk mencegah terjadinya infinite loop atau kegagalan akses saat proses autentikasi berlangsung.

## 4. Analisis Validasi dan Hasil Uji Coba
Melalui pemeriksaan yang ditentukan yang diselaraskan dengan kebutuhan sistem operasional, diperoleh hasil sebagai berikut:

1.  **Validasi Login** : Sistem berhasil melakukan otorisasi penuh saat kredensial yang tepat dimasukkan (Username: admin, Password: admin123).
   <img width="1919" height="881" alt="Screenshot 2026-04-28 142243" src="https://github.com/user-attachments/assets/2e5b6a57-c73b-4993-b183-a7c2a4fe34f3" />

2.  **Uji Coba SQL Injection** : Saat mencoba memasukkan kode ' OR '1'='1 pada kolom username, sistem tetap menolak akses dan memberikan pesan error "Username atau Password salah". Ini membuktikan bahwa input berbahaya telah berhasil "dijinakkan".
   <img width="1919" height="955" alt="Screenshot 2026-04-28 135333" src="https://github.com/user-attachments/assets/6532f815-dfd3-4704-a6b7-610f1dc9c459" />
3.  **Efisiensi Visual** : Dashboard admin kini lebih informatif dengan kategori tampilan yang kontekstual, mempermudah admin dalam mengelola data.

## 5. Kesimpulan
Implementasi ini membuktikan bahwa kerangka kerja modern seperti CodeIgniter 4 menyediakan instrumen yang efektif dalam menangkal serangan siber fundamental. Sinkronan antara struktur tabel database, logika pada controller, dan pemetaan rute menjadi faktor penentu dalam membangun aplikasi yang aman. Untuk pengembangan selanjutnya, penerapan sistem hashing pada kata sandi sangat disarankan untuk memberikan perlindungan ekstra pada data sensitif pengguna.

## Referensi
[1] E. Adicandra, Yulindon, R. Dewi, dan S. Rifka, “Implementasi Autentikasi dan Otorisasi pada Sistem Informasi Berbasis Web,” Jurnal Ilmiah Penelitian Mahasiswa (JIPM) , vol. 4, tidak. 2, hlm.289–298, April 2026.

[2] EB Zega, W. Ginting, dan R. Damanik, “Sistem Keamanan Login Menggunakan Metode Autentikasi Dua Faktor dengan Verifikasi Gambar,” International Multidiciplinary Journal , vol. 1, tidak. 2 Agustus 2026.
