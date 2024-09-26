---
lab:
  title: Membuat solusi Pencarian Azure AI
  module: Module 12 - Creating a Knowledge Mining Solution
---

# Membuat solusi Pencarian Azure AI

Semua organisasi mengandalkan informasi untuk membuat keputusan, menjawab pertanyaan, serta berfungsi secara efisien. Masalah bagi sebagian besar organisasi bukanlah kurangnya informasi, tetapi tantangan untuk menemukan serta mengekstraksi informasi dari sekumpulan besar dokumen, database, serta sumber lain tempat informasi disimpan.

Misalnya, bayangkan *Margie's Travel* adalah agen perjalanan yang mengkhususkan diri dalam mengatur perjalanan ke kota-kota di seluruh dunia. Seiring waktu, perusahaan tersebut telah mengumpulkan sejumlah besar informasi dalam dokumen seperti brosur, serta ulasan hotel yang dikirimkan oleh pelanggan. Data ini adalah sumber wawasan yang berharga bagi agen perjalanan serta pelanggan saat mereka merencanakan perjalanan, tetapi banyaknya data dapat menyulitkan untuk menemukan informasi yang relevan untuk menjawab pertanyaan pelanggan tertentu.

Untuk mengatasi tantangan ini, Margie's Travel dapat menggunakan Pencarian Azure AI untuk menerapkan solusi di mana dokumen diindeks dan diperkaya dengan menggunakan keterampilan AI untuk membuatnya lebih mudah dicari.

## Membuat sumber daya Azure

Solusi yang akan Anda buat untuk Margie's Travel memerlukan sumber daya berikut dalam langganan Azure Anda:

- Sumber daya **Pencarian Azure AI**, yang akan mengelola pengindeksan dan kueri.
- Sumber daya **Layanan Azure AI**, yang menyediakan layanan AI untuk keterampilan yang dapat digunakan solusi pencarian Anda untuk memperkaya data di sumber data dengan wawasan yang dihasilkan AI.
- **Akun penyimpanan** dengan kontainer blob tempat dokumen yang akan dicari tersimpan.

> **Penting**: Sumber daya Pencarian Azure AI dan Layanan Azure AI Anda harus berada di lokasi yang sama!

### Membuat sumber daya  Pencarian Azure AI

1. Di browser web, buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
2. Pilih **&#65291; Buat tombol** sumber daya, cari *pencarian*, dan buat sumber daya **Pencarian Azure AI** dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Buat grup sumber daya baru (jika Anda menggunakan langganan terbatas, Anda mungkin tidak memiliki izin untuk membuat grup sumber daya baru - gunakan yang disediakan)*
    - **Nama layanan**: *Masukkan nama yang unik*
    - **Lokasi**: *Pilih lokasi - perhatikan bahwa sumber daya Pencarian Azure AI dan Layanan Azure AI Anda harus berada di lokasi yang sama*
    - **Tingkat harga**: Dasar

3. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
4. Tinjau halaman **Gambaran umum** di bilah untuk sumber daya Pencarian Azure AI Anda di portal Microsoft Azure. Di sini, Anda dapat menggunakan antarmuka visual untuk membuat, menguji, mengelola, dan memantau berbagai komponen solusi pencarian yang termasuk sumber data, indeks, pengindeks, dan set kemampuan.

### Membuat sumber daya Layanan Azure AI

Jika Belum memilikinya di langganan, Anda harus menyediakan sumber daya **Layanan Azure AI**. Solusi pencarian Anda akan menggunakan ini untuk memperkaya data di datastore dengan wawasan yang dihasilkan AI.

1. Kembali ke beranda portal Azure, lalu pilih **&#65291; Buat tombol** sumber daya, cari *Layanan Azure AI*, dan buat sumber daya **akun multi-layanan Azure AI** dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Grup sumber daya yang sama dengan sumber daya Pencarian Azure AI Anda*
    - **Wilayah**: *Lokasi yang sama dengan sumber daya Pencarian Azure AI Anda*
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Standar S0
2. Pilih kotak centang yang diperlukan dan buat sumber daya.
3. Tunggu hingga penyebaran selesai, lalu lihat detail penyebaran.

### Buat akun penyimpanan

1. Kembali ke beranda portal Microsoft Azure, lalu pilih tombol **&#65291;Buat sumber daya**, cari *akun penyimpanan*, dan buat sumber daya **akun penyimpanan** dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: **Grup sumber daya yang sama dengan sumber daya Pencarian Azure AI dan Layanan Azure AI Anda*
    - **Nama akun penyimpanan**: *Masukkan nama yang unik*
    - **Wilayah**: *Pilih wilayah yang tersedia*
    - **Performa**: Standar
    - **Redundansi**: Penyimpanan redundan secara lokal (LRS)
    - Pada tab **Tingkat Lanjut**, centang kotak di samping *Izinkan mengaktifkan akses anonim pada kontainer individual*
2. Tunggu hingga penyebaran selesai, lalu buka sumber daya yang disebarkan.
3. Pada halaman **Ringkasan**, perhatikan **ID Langganan** -ini menunjukkan langganan yang menyediakan akun penyimpanan.
4. Pada halaman **Kunci akses**, perhatikan bahwa dua kunci telah dibuat untuk akun penyimpanan Anda. Kemudian pilih **Tampilkan kunci** untuk melihat kunci.

    > **Tips**: Biarkan bilah **Akun Penyimpanan** tetap terbuka - Anda akan memerlukan ID langganan dan salah satu kunci dalam prosedur berikutnya.

## Bersiap untuk mengembangkan aplikasi di Visual Studio Code

Anda akan mengembangkan aplikasi pencarian menggunakan Visual Studio Code. File kode untuk aplikasi Anda telah disediakan di repositori GitHub.

> **Tips**: Jika Anda sudah mengkloning repositori **mslearn-knowledge-mining**, buka di Visual Studio Code. Atau, ikuti langkah-langkah ini untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Memulai Visual Studio Code.
1. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` ke folder lokal (tidak masalah folder mana).
1. Setelah repositori dikloning, buka folder di Visual Studio Code.
1. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

## Mengunggah Dokumen ke Azure Storage

Sekarang setelah Anda memiliki sumber daya yang diperlukan, Anda dapat mengunggah beberapa dokumen ke akun Penyimpanan Azure Anda.

1. Di Visual Studio Code, di panel **Penjelajah**, perluas folder **Labfiles\01-azure-search** dan pilih **UploadDocs.cmd**.
2. Edit file batch untuk mengganti tempat penampung **YOUR_SUBSCRIPTION_ID**, **YOUR_AZURE_STORAGE_ACCOUNT_NAME**, dan **YOUR_AZURE_STORAGE_KEY** dengan ID langganan yang sesuai, nama akun penyimpanan Azure, dan nilai kunci akun penyimpanan Azure untuk akun penyimpanan yang Anda buat sebelumnya.
3. Simpan perubahan Anda, lalu klik kanan folder **01-azure-search** dan buka terminal terintegrasi.
4. Masukkan perintah berikut untuk masuk ke langganan Azure Anda menggunakan Azure CLI.

    ```powershell
    az login
    ```

    Tab browser web akan terbuka dan meminta Anda untuk masuk ke Azure. Lakukan, lalu tutup tab browser dan kembali ke Visual Studio Code.

5. Masukkan perintah berikut untuk menjalankan file batch. Perintah ini akan membuat kontainer blob di akun penyimpanan Anda dan mengunggah dokumen di folder **data** ke dalamnya.

    ```powershell
    .\UploadDocs.cmd
    ```

## Mengindeks dokumen

Sekarang setelah Anda memiliki dokumen, Anda dapat membuat solusi pencarian dengan mengindeksnya.

1. Di portal Microsoft Azure, telusuri ke sumber daya Pencarian Azure AI Anda. Kemudian, pada halaman **Ringkasan**, pilih **Impor data**.
2. Pada halaman **Hubungkan ke data Anda**, dalam daftar **Sumber Data**, pilih **Azure Blob Storage**. Kemudian lengkapi detail penyimpanan data dengan nilai berikut:
    - **Sumber Data**: Azure Blob Storage
    - **Nama sumber data**: margies-data
    - **Data yang akan diekstrak**: Konten dan metadata
    - **Mode penguraian**: Default
    - **String koneksi**: *Pilih **Pilih koneksi yang ada**. Kemudian pilih akun penyimpanan Anda, dan terakhir pilih kontainer **margies** yang dibuat oleh skrip UploadDocs.cmd.*
    - **Autentikasi identitas terkelola**: Tidak ada
    - **Nama kontainer**: margies
    - **Folder blob**: *Biarkan ini kosong*
    - **Deskripsi**: Brosur dan ulasan di situs web Perjalanan Margie.
3. Lanjutkan ke langkah berikutnya (*Tambahkan keterampilan kognitif*).
4. di bagian **Lampirkan Layanan Azure AI**, pilih sumber daya Azure AI Services Anda.
5. Di bagian **Tambahkan pengayaan**:
    - Ubah **Nama set kemampuan** menjadi **margies-skillset**.
    - Pilih opsi **Aktifkan OCR dan gabungkan semua teks ke dalam bidang merged_content**.
    - Pastikan **bidang data Sumber** diatur ke **merged_content**.
    - Biarkan **Tingkat granuralitas pengayaan** sebagai **Bidang sumber**, yang mengatur seluruh konten dokumen yang diindeks; tetapi perhatikan bahwa Anda dapat mengubah pengaturan ini untuk mengekstrak informasi pada tingkat yang lebih terperinci, seperti halaman atau kalimat.
    - Pilih bidang yang diperkaya berikut:

        | Kemampuan Kognitif | Parameter | Nama bidang |
        | --------------- | ---------- | ---------- |
        | Ekstrak nama lokasi | | locations |
        | Mengekstrak frasa kunci | | frasa kunci |
        | Mendeteksi bahasa | | bahasa |
        | Buat tag dari gambar | | imageTags |
        | Buat keterangan dari gambar | | imageCaption |

6. Periksa kembali pilihan Anda (mungkin sulit untuk mengubahnya nanti). Kemudian lanjutkan ke langkah berikutnya (*Sesuaikan indeks target*).
7. Ubah **Nama indeks** menjadi **margies-index**.
8. Pastikan **Kunci** diatur menjadi **metadata_storage_path** dan biarkan **Nama pemberi saran** kosong dan **Mode pencarian** pada defaultnya.
9. Buat perubahan berikut pada bidang indeks, biarkan semua bidang lainnya dengan pengaturan defaultnya (**PENTING**: Anda mungkin perlu menggulir ke kanan untuk melihat seluruh bagian tabel):

    | Nama bidang | Dapat diambil | Dapat difilter | Dapat disortir | Facetable | Dapat dicari |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | metadata_author | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | frasa kunci | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | bahasa | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |

10. Periksa kembali pilihan Anda, dengan memberikan perhatian khusus untuk memastikan bahwa opsi **Dapat diambil kembali**, **Dapat Difilter**, **Dapat Diurutkan**, **Dapat Dilihat**, dan **Dapat dicari** dipilih untuk setiap bidang (mungkin sulit untuk mengubahnya nanti). Kemudian lanjutkan ke langkah berikutnya (*Buat pengindeks*).
11. Ubah **Nama pengindeks** menjadi **margies-indexer**.
12. Biarkan **Jadwal** diatur ke **Sekali**.
13. Perluas opsi **Tingkat Lanjut**, dan pastikan bahwa opsi **Kunci pengodean Base-64** dipilih (umumnya kunci pengodean membuat indeks lebih efisien).
14. Pilih **Kirim** untuk membuat sumber data, set kemampuan, indeks, dan pengindeks. Pengindeks dijalankan secara otomatis dan menjalankan alur pengindeksan, yang:
    1. Mengekstrak bidang dan konten metadata dokumen dari sumber data
    2. Menjalankan set kemampuan dari kemampuan kognitif untuk menghasilkan bidang tambahan yang diperkaya
    3. Memetakan bidang yang diekstraksi ke indeks.
15. Di sisi kiri, lihat halaman **Pengindeks**, yang harus menampilkan margies-indexer ** yang baru dibuat**. Tunggu beberapa menit, dan klik **&orarr; Refresh** hingga **Status** menunjukkan keberhasilan.

## Cari indeks

Sekarang Anda memiliki indeks, Anda dapat mencarinya.

1. Di bagian atas halaman **Gambaran Umum** untuk sumber daya Pencarian Azure AI Anda, pilih **Penjelajah pencarian**.
2. Di Penjelajah pencarian, di kotak **String kueri**, masukkan `*` (tanda bintang tunggal), lalu pilih **Cari**.

    Kueri ini mengambil semua dokumen dalam indeks dalam format JSON. Periksa hasilnya dan catat bidang untuk setiap dokumen, yang berisi konten dokumen, metadata, dan data yang diperkaya yang diekstraksi oleh keterampilan kognitif yang Anda pilih.

3. Di menu **Tampilan**, pilih **tampilan JSON** dan perhatikan bahwa permintaan JSON untuk pencarian ditampilkan, seperti ini:

    ```json
    {
      "search": "*"
    }
    ```

1. Ubah permintaan JSON untuk menyertakan parameter **jumlah** seperti yang ditunjukkan di sini:

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. Kirimkan pencarian yang dimodifikasi. Kali ini, hasilnya menyertakan bidang **@odata.count** di bagian atas hasil yang menunjukkan jumlah dokumen yang dikembalikan oleh pencarian.

4. Coba kueri berikut:

    ```json
    {
      "search": "*",
      "count": true,
      "select": "metadata_storage_name,metadata_author,locations"
    }
    ```

    Kali ini hasilnya hanya menyertakan nama file, penulis, dan lokasi yang disebutkan dalam konten dokumen. Nama file dan penulisnya ada di bidang **metadata_storage_name** dan **metadata_author**, yang diambil dari dokumen sumber. Bidang **lokasi** dihasilkan oleh keterampilan kognitif.

5. Sekarang coba string kueri berikut:

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name,keyphrases"
    }
    ```

    Pencarian ini menemukan dokumen yang menyebutkan "New York" di salah satu bidang yang dapat dicari, dan mengembalikan nama file dan frasa kunci dalam dokumen.

6. Mari kita coba satu kueri lagi:

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name",
      "filter": "metadata_author eq 'Reviewer'"
    }
    ```

    Kueri ini mengembalikan nama file dokumen apa pun yang ditulis oleh *Peninjau* yang menyebutkan "New York".

## Jelajahi dan ubah definisi komponen pencarian

Komponen solusi pencarian didasarkan pada definisi JSON, yang dapat Anda lihat dan edit di portal Azure.

Meskipun Anda dapat menggunakan portal untuk membuat dan memodifikasi solusi pencarian, sering kali diinginkan untuk menentukan objek pencarian di JSON dan menggunakan antarmuka Layanan Azure AI REST untuk membuat dan memodifikasinya.

### Mendapatkan titik akhir dan kunci untuk sumber daya Pencarian Azure AI Anda

1. Di portal Microsoft Azure, kembali ke halaman **Gambaran Umum** untuk sumber daya Pencarian Azure AI Anda; dan di bagian atas halaman, temukan **Url** untuk sumber daya Anda (yang terlihat seperti **https://resource_name.search.windows.net**) dan salin ke clipboard.
2. Di Visual Studio Code, di panel Penjelajah, perluas folder **01-azure-search** dan subfolder **pencarian modifikasi**, dan pilih **modify-search.cmd** untuk membukanya. Anda akan menggunakan file skrip ini untuk menjalankan perintah *cURL* yang mengirimkan JSON ke antarmuka Layanan Azure AI REST.
3. Di **modify-search.cmd**, ganti wadah penampung **YOUR_SEARCH_URL** dengan URL yang Anda salin ke clipboard.
4. Di portal Azure, di bagian **Pengaturan** , lihat halaman **Kunci** untuk sumber daya Pencarian Azure AI Anda dan salin **Kunci admin utama** ke papan klip.
5. Di Visual Studio Code, ganti wadah penampung **YOUR_ADMIN_KEY** dengan kunci yang Anda salin ke clipboard.
6. Simpan perubahan ke **modify-search.cmd** (tetapi jangan jalankan dulu!)

### Tinjau dan modifikasi set keterampilan

1. Di Visual studio Code, di folder **modify-search**, buka **skillset.json**. Cara ini menunjukkan definisi JSON untuk **margies-skillset**.
2. Di bagian atas definisi set keterampilan, perhatikan objek **cognitiveServices**, yang digunakan untuk menyambungkan sumber daya Layanan Azure AI Anda ke set keterampilan.
3. Di portal Azure, buka sumber daya Layanan Azure AI Anda (<u>bukan</u> sumber daya Pencarian Azure AI Anda!) dan di bagian **Manajemen Sumber Daya**, lihat halaman **Kunci dan Endpoint**. Kemudian salin **KUNCI 1** ke papan klip.
4. Di Visual Studio Code, di **skillset.json**, ganti tempat penampung **YOUR_COGNITIVE_SERVICES_KEY** dengan kunci Layanan Azure AI yang Anda salin ke clipboard.
5. Gulir file JSON, perhatikan bahwa file tersebut menyertakan definisi untuk keterampilan yang Anda buat menggunakan antarmuka pengguna Pencarian Azure AI di portal Microsoft Azure. Di bagian bawah daftar keterampilan, keterampilan tambahan telah ditambahkan dengan definisi berikut:

    ```json
    {
        "@odata.type": "#Microsoft.Skills.Text.V3.SentimentSkill",
        "defaultLanguageCode": "en",
        "name": "get-sentiment",
        "description": "New skill to evaluate sentiment",
        "context": "/document",
        "inputs": [
            {
                "name": "text",
                "source": "/document/merged_content"
            },
            {
                "name": "languageCode",
                "source": "/document/language"
            }
        ],
        "outputs": [
            {
                "name": "sentiment",
                "targetName": "sentimentLabel"
            }
        ]
    }
    ```

    Keterampilan baru bernama **get-sentiment**, dan untuk setiap tingkat **dokumen** dalam dokumen, keterampilan tersebut akan mengevaluasi teks yang ditemukan di bidang **merged_content** dokumen sedang diindeks (yang mencakup konten sumber serta teks apa pun yang diekstraksi dari gambar dalam konten). Keterampilan ini menggunakan **bahasa** dokumen yang diekstraksi (dengan bahasa default bahasa Inggris), dan mengevaluasi label untuk sentimen konten. Nilai untuk label sentimen dapat berupa "positif", "negatif", "netral", atau "campuran". Label ini kemudian ditampilkan sebagai bidang baru bernama **sentimentLabel**.

6. Simpan perubahan yang Anda buat pada **skillset.json**.

### Tinjau dan ubah indeks

1. Di Visual studio Code, di folder **modify-search**, buka **index.json**. Cara ini menunjukkan definisi JSON untuk **margies-index**.
2. Gulir melalui indeks dan lihat definisi bidang. Beberapa bidang didasarkan pada metadata dan konten dalam dokumen sumber, dan yang lainnya adalah hasil keterampilan dalam kumpulan keterampilan.
3. Di akhir daftar bidang yang Anda tentukan di portal Microsoft Azure, perhatikan bahwa dua bidang tambahan telah ditambahkan:

    ```json
    {
        "name": "sentiment",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "sortable": true
    },
    {
        "name": "url",
        "type": "Edm.String",
        "facetable": false,
        "filterable": true,
        "retrievable": true,
        "searchable": false,
        "sortable": false
    }
    ```

4. Bidang **sentimen** akan digunakan untuk menambahkan output dari keterampilan **get-sentiment** yang ditambahkan ke dalam kumpulan keterampilan. Bidang **URL** akan digunakan untuk menambahkan URL untuk setiap dokumen yang diindeks ke indeks, berdasarkan nilai **metadata_storage_path** yang diambil dari sumber data. Perhatikan bahwa indeks sudah menyertakan bidang **metadata_storage_path**, tetapi digunakan sebagai kunci indeks dan dikodekan Base-64, membuatnya efisien sebagai kunci tetapi mengharuskan aplikasi klien untuk memecahkan kodenya jika mereka ingin menggunakan nilai URL yang sebenarnya sebagai lapangan. Menambahkan bidang kedua untuk nilai yang tidak dikodekan menyelesaikan masalah ini.

### Tinjau dan ubah pengindeks

1. Di Visual studio Code, di folder **modify-search**, buka **indexer.json**. Cara ini menunjukkan definisi JSON untuk **margies-indexer**, yang memetakan bidang yang diekstrak dari konten dokumen dan metadata (di bagian **fieldMappings**), dan nilai yang diekstrak oleh keterampilan dalam kumpulan keterampilan (di **outputFieldMappings**), ke bidang di indeks.
2. Dalam daftar **fieldMappings**, perhatikan pemetaan untuk nilai **metadata_storage_path** ke bidang kunci yang dikodekan base-64. Pemetaan ini dibuat saat Anda menetapkan **metadata_storage_path** sebagai kunci dan memilih opsi untuk mengodekan kunci di portal Microsoft Azure. Selain itu, pemetaan baru secara eksplisit memetakan nilai yang sama ke bidang **URL**, tetapi tanpa pengodean Base-64:

    ```json
    {
        "sourceFieldName" : "metadata_storage_path",
        "targetFieldName" : "url"
    }    
    ```

    Semua metadata dan bidang konten lainnya dalam dokumen sumber secara implisit dipetakan ke bidang dengan nama yang sama dalam indeks.

3. Tinjau bagian **ouputFieldMappings**, yang memetakan output dari keterampilan dalam kumpulan keterampilan ke bidang indeks. Sebagian besar dari bidang ini menampilkan pilihan yang Anda buat di antarmuka pengguna, namun pemetaan berikut telah ditambahkan untuk memetakan nilai **sentimentLabel** yang diekstraks oleh keterampilan sentimen Anda ke bidang **sentimen** yang Anda tambahkan ke indeks:

    ```json
    {
        "sourceFieldName": "/document/sentimentLabel",
        "targetFieldName": "sentiment"
    }
    ```

### Gunakan REST API untuk memperbarui solusi pencarian

1. Klik kanan folder **modify-search** dan buka terminal terintegrasi.
2. Di panel terminal untuk folder **modify-search**, masukkan perintah berikut untuk menjalankan skrip **modify-search.cmd**, yang mengirimkan definisi JSON ke antarmuka REST dan memulai pengindeksan .

    ```powershell
    ./modify-search
    ```

3. Setelah skrip selesai, kembali ke halaman **Gambaran Umum** untuk sumber daya Pencarian Azure AI Anda di portal Microsoft Azure dan lihat halaman **Pengindeks**. Pilih **Refresh** secara berkala untuk melacak kemajuan operasi pengindeksan. Mungkin perlu satu menit atau lebih untuk menyelesaikannya.

    *Mungkin ada beberapa peringatan untuk beberapa dokumen yang terlalu besar untuk mengevaluasi sentimen. Seringkali analisis sentimen dilakukan pada tingkat halaman atau kalimat daripada dokumen lengkap; namun dalam skenario kasus ini, sebagian besar dokumen - khususnya ulasan hotel, cukup pendek untuk dievaluasi skor sentimen tingkat dokumen yang berguna.*

### Kueri indeks yang dimodifikasi

1. Di bagian atas bilah untuk sumber daya Pencarian Azure AI Anda, pilih **Penjelajah pencarian**.
2. Di Penjelajah pencarian, dalam kotak **String kueri**, kirimkan kueri JSON berikut:

    ```json
    {
      "search": "London",
      "select": "url,sentiment,keyphrases",
      "filter": "metadata_author eq 'Reviewer' and sentiment eq 'positive'"
    }
    ```

    Kueri ini mengambil **URL**, **sentimen**, dan **frasa kunci** untuk semua dokumen yang menyebutkan *London* yang ditulis oleh *Peninjau* yang memiliki label **sentimen** positif (dengan kata lain, ulasan positif yang menyebutkan London)

3. Tutup halaman **Penjelajah pencarian** untuk kembali ke halaman **Ringkasan**.

## Buat aplikasi klien pencarian

Sekarang Anda memiliki indeks yang berguna, Anda dapat menggunakannya dari aplikasi klien. Anda dapat melakukan pencarian menggunakan antarmuka REST, mengirimkan permintaan dan menerima tanggapan dalam format JSON melalui HTTP; atau Anda dapat menggunakan kit pengembangan perangkat lunak (SDK) untuk bahasa pemrograman pilihan Anda. Dalam latihan ini, kita akan menggunakan SDK.

> **Catatan**: Anda dapat memilih untuk menggunakan SDK baik untuk **C#** atau **Python**. Dalam langkah-langkah di bawah ini, lakukan tindakan yang sesuai untuk bahasa pilihan Anda.

### Dapatkan titik akhir dan kunci untuk sumber daya pencarian Anda

1. Di portal Microsoft Azure, pada halaman **Gambaran Umum** untuk sumber daya Pencarian Azure AI Anda, perhatikan nilai **Url**, yang seharusnya mirip dengan **https://*your_resource_name*.search.windows.net**. Nilai ini adalah titik akhir untuk sumber pencarian Anda.
2. Pada halaman **Kunci**, perhatikan bahwa ada dua kunci **admin**, dan satu kunci **kueri**. Kunci *admin* digunakan untuk membuat dan mengelola sumber daya pencarian; kunci *kueri* digunakan oleh aplikasi klien yang hanya perlu melakukan kueri pencarian.

    *Anda memerlukan titik akhir dan kunci kueri untuk aplikasi klien Anda.*

### Bersiap untuk menggunakan Pencarian Azure AI SDK

1. Di Visual Studio Code, di panel **Penjelajah**, telusuri folder **01-azure-search** dan perluas folder **C-Sharp** atau **Python** tergantung pada preferensi bahasa Anda.
2. Klik kanan folder **margies-travel** dan buka terminal terintegrasi. Kemudian instal paket Pencarian Azure AI SDK dengan menjalankan perintah yang sesuai untuk preferensi bahasa Anda:

    **C#**

    ```
    dotnet add package Azure.Search.Documents --version 11.6.0
    ```

    **Python**

    ```
    pip install azure-search-documents==11.5.1
    ```

3. Lihat konten folder **margies-travel**, dan perhatikan bahwa folder tersebut berisi file untuk pengaturan konfigurasi:
    - **C#**: appsettings.json
    - **Python**: .env

    Buka file konfigurasi dan perbarui nilai konfigurasi yang dikandungnya untuk mencerminkan **titik akhir** dan **kunci kueri** untuk sumber daya Pencarian Azure AI Anda. Simpan perubahan Anda.

### Jelajahi kode untuk mencari indeks

Folder **margies-travel** berisi file kode untuk aplikasi web (aplikasi web Microsoft C# *ASP.NET Razor* atau aplikasi Python *Flask*), yang mencakup fungsionalitas pencarian.

1. Buka file kode berikut di aplikasi web, tergantung pada pilihan bahasa pemrograman Anda:
    - **C#**:Pages/Index.cshtml.cs
    - **Python**: app.py
2. Di dekat bagian atas file kode, temukan komentar **Impor namespace pencarian**, dan perhatikan namespace layanan yang telah diimpor untuk bekerja dengan Pencarian Azure AI SDK:
3. Dalam fungsi **search_query**, temukan komentar **Buat klien pencarian**, dan perhatikan bahwa kode membuat objek **SearchClient** menggunakan titik akhir dan kunci kueri untuk sumber daya Azure AI Search Anda:
4. Dalam fungsi **search_query**, temukan komentar **Kirim kueri pencarian**, dan tinjau kode untuk mengirimkan pencarian untuk teks yang ditentukan dengan opsi berikut:
    - *Mode pencarian* yang memerlukan **semua** kata individual dalam teks pencarian yang ditemukan.
    - Jumlah total dokumen yang ditemukan oleh pencarian termasuk dalam hasil.
    - Hasil disaring untuk menyertakan hanya dokumen yang cocok dengan ekspresi filter yang disediakan.
    - Hasilnya diurutkan ke dalam urutan pengurutan yang ditentukan.
    - Setiap nilai diskret bidang **metadata_author** dikembalikan sebagai *faset* yang dapat digunakan untuk menampilkan nilai yang telah ditentukan sebelumnya untuk pemfilteran.
    - Hingga tiga ekstrak bidang **merged_content** dan **imageCaption** dengan istilah pencarian yang disorot disertakan dalam hasil.
    - Hasilnya hanya mencakup bidang yang ditentukan.

### Jelajahi kode untuk merender hasil pencarian

Aplikasi web sudah menyertakan kode untuk memproses dan merender hasil pencarian.

1. Buka file kode berikut di aplikasi web, tergantung pada pilihan bahasa pemrograman Anda:
    - **C#**:Pages/Index.cshtml
    - **Python**: templates/search.html
2. Periksa kode, yang membuat halaman tempat hasil pencarian ditampilkan. Perhatikan bahwa:
    - Halaman dimulai dengan formulir pencarian yang dapat digunakan pengguna untuk mengirimkan pencarian baru (dalam versi aplikasi Python, formulir ini ditentukan dalam templat **base.html**), yang dirujuk di awal halaman.
    - Formulir kedua kemudian diberikan, memungkinkan pengguna untuk menyaring hasil pencarian. Kode untuk formulir ini:
        - Mengambil dan menampilkan jumlah dokumen dari hasil pencarian.
        - Mengambil nilai faset untuk bidang **metadata_author** dan menampilkannya sebagai daftar opsi untuk pemfilteran.
        - Membuat daftar dropdown dari opsi pengurutan untuk hasil.
    - Kode kemudian beralih melalui hasil pencarian, membuat setiap hasil sebagai berikut:
        - Menampilkan bidang **metadata_storage_name** (nama file) sebagai tautan ke alamat di bidang **URL**.
        - Menampilkan *sorotan* untuk istilah pencarian yang ditemukan di bidang **merged_content** dan **imageCaption** untuk membantu menampilkan istilah pencarian dalam konteks.
        - Menampilkan bidang **metadata_author**, **metadata_storage_size**, **metadata_storage_last_modified**, dan **language**.
        - Menampilkan label **sentimen** untuk dokumen tersebut. Bisa berupa positif, negatif, netral, atau campuran.
        - Menampilkan lima **frasa kunci** pertama (jika ada).
        - Menampilkan lima **lokasi** pertama (jika ada).
        - Menampilkan lima **imageTag** pertama (jika ada).

### Jalankan aplikasi web

 1. kembali ke terminal terintegrasi untuk folder **margies-travel**, dan masukkan perintah berikut untuk menjalankan program:

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    flask run
    ```

2. Dalam pesan yang ditampilkan saat aplikasi berhasil dimulai, ikuti tautan ke aplikasi web yang sedang berjalan (*http://localhost:5000/* atau *http://127.0.0.1:5000/*) untuk membuka situs Margies Travel di browser web.
3. Di situs web Margie's Travel, masukkan **Hotel London** ke dalam kotak pencarian dan klik **Cari**.
4. Tinjau hasil pencarian. Hasil pencarian menyertakan nama file (dengan hyperlink ke URL file), ekstrak konten file dengan istilah pencarian (*London* dan *hotel*) yang ditekankan, dan atribut file lainnya dari bidang indeks.
5. Perhatikan bahwa halaman hasil menyertakan beberapa elemen antarmuka pengguna yang memungkinkan Anda menyaring hasil. Ini termasuk:
    - *Filter* berdasarkan nilai faset untuk bidang **metadata_author**. Cara ini menunjukkan cara Anda dapat menggunakan bidang *facetable* untuk mengembalikan daftar *faset* - bidang dengan kumpulan kecil nilai diskret yang dapat ditampilkan sebagai nilai filter potensial di antarmuka pengguna.
    - Kemampuan untuk *mengurutkan* hasil berdasarkan bidang tertentu dan arah pengurutan (naik atau turun). Urutan default didasarkan pada *relevansi*, yang dihitung sebagai nilai **search.score()** berdasarkan *profil penilaian* yang mengevaluasi frekuensi dan pentingnya pencarian istilah di bidang indeks.
6. Pilih filter **Peninjau** dan opsi pengurutan **Positif ke negatif**, lalu pilih **Persempit Hasil**.
7. Perhatikan bahwa hasil difilter untuk hanya menyertakan ulasan, dan diurutkan berdasarkan label sentimen.
8. Di kotak **Cari**, masukkan pencarian baru untuk **hotel tenang di New York** dan tinjau hasilnya.
9. Coba istilah pencarian berikut:
    - **Menara London** (perhatikan bahwa istilah ini diidentifikasi sebagai *frasa kunci* dalam beberapa dokumen).
    - **pencakar langit** (perhatikan bahwa kata ini tidak muncul dalam konten sebenarnya dari dokumen apa pun, tetapi ditemukan dalam *teks gambar* dan *tag gambar* yang dibuat untuk gambar dalam beberapa dokumen).
    - **Gurun Mojave** (perhatikan bahwa istilah ini diidentifikasi sebagai *lokasi* di beberapa dokumen).
10. Tutup tab browser yang berisi situs web Margie's Travel dan kembali ke Visual Studio Code. Kemudian di terminal Python untuk folder **margies-travel** (tempat aplikasi dotnet atau flask berjalan), masukkan Ctrl+C untuk menghentikan aplikasi.

## Pembersihan

Sekarang setelah Anda menyelesaikan latihan, hapus semua sumber daya yang tidak lagi Anda perlukan. Menghapus sumber daya Azure:

1. Di portal Microsoft Azure, pilih **Grup sumber daya**.
1. Pilih grup sumber daya yang tidak Anda perlukan lagi, lalu pilih **Hapus grup sumber daya**.

## Informasi selengkapnya

Untuk mempelajari selengkapnya tentang Azure AI Search, lihat [Dokumentasi Pencarian Azure AI](https://docs.microsoft.com/azure/search/search-what-is-azure-search).
