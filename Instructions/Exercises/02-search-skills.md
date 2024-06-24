---
lab:
  title: Membuat Kemampuan Kustom untuk Pencarian Azure AI
  module: Module 12 - Creating a Knowledge Mining Solution
---

# Membuat Kemampuan Kustom untuk Pencarian Azure AI

Pencarian Azure AI menggunakan alur pengayaan kemampuan AI untuk mengekstrak bidang yang dihasilkan AI dari dokumen dan memasukkannya ke dalam indeks pencarian. Ada serangkaian keterampilan bawaan komprehensif yang dapat Anda gunakan, tetapi jika Anda memiliki persyaratan khusus yang tidak terpenuhi oleh keterampilan ini, Anda dapat membuat keterampilan kustom.

Dalam latihan ini, Anda akan membuat keterampilan khusus yang menabulasikan frekuensi kata-kata individual dalam dokumen untuk menghasilkan daftar lima kata teratas yang paling banyak digunakan, dan menambahkannya ke solusi pencarian untuk Margie's Travel - agen perjalanan fiktif.

## Bersiap untuk mengembangkan aplikasi di Visual Studio Code

Anda akan mengembangkan aplikasi pencarian menggunakan Visual Studio Code. File kode untuk aplikasi Anda telah disediakan di repositori GitHub.

> **Tips**: Jika Anda sudah mengkloning repositori **mslearn-knowledge-mining**, buka di Visual Studio Code. Atau, ikuti langkah-langkah ini untuk mengkloningnya ke lingkungan pengembangan Anda.

1. Memulai Visual Studio Code.
1. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` ke folder lokal (tidak masalah folder mana).
1. Setelah repositori dikloning, buka folder di Visual Studio Code.
1. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

## Membuat sumber daya Azure

> **Catatan**: Jika sebelumnya Anda telah menyelesaikan latihan **[Membuat solusi Pencarian Azure AI](01-azure-search.md)**, dan masih memiliki sumber daya Azure ini dalam langganan, Anda dapat melewati bagian ini dan mulai dari bagian **Membuat solusi pencarian**. Jika tidak, ikuti langkah-langkah di bawah ini untuk menyediakan sumber daya Azure yang diperlukan.

1. Di browser web, buka portal Microsoft Azure di `https://portal.azure.com`, dan masuk menggunakan akun Microsoft yang terkait dengan langganan Azure Anda.
2. Di bilah pencarian teratas, cari *layanan Azure AI*, pilih **Layanan Azure AI**, dan buat sumber daya akun multi-layanan layanan Azure AI dengan pengaturan berikut:
    - **Langganan**: *Langganan Azure Anda*
    - **Grup sumber daya**: *Pilih atau buat grup sumber daya (jika Anda menggunakan langganan terbatas, Anda mungkin tidak memiliki izin untuk membuat grup sumber daya baru - gunakan yang disediakan)*
    - **Wilayah**: *Pilih wilayah yang tersedia yang secara geografis dekat dengan Anda*
    - **Nama**: *Masukkan nama unik*
    - **Tingkat harga**: Standar S0
1. Setela disebarkan, buka sumber daya dan di halaman **Gambaran umum**, catat **ID Langganan** dan **Lokasi**. Anda akan memerlukan nilai-nilai ini, bersama dengan nama grup sumber daya di langkah selanjutnya. 
1. Di Visual Studio Code, perluas folder **Labfiles/02-search-skill** dan pilih **setup.cmd**. Anda akan menggunakan skrip batch ini untuk menjalankan perintah antarmuka baris perintah (CLI) Azure yang diperlukan untuk membuat sumber daya Azure yang Anda butuhkan.
1. Klik kanan folder **02-search-skill** dan pilih **Buka di Terminal Terintegrasi**.
1. Di panel terminal, masukkan perintah berikut untuk membuat koneksi yang diautentikasi ke langganan Azure Anda.

    ```powershell
    az login --output none
    ```

8. Saat diminta, pilih atau masuk ke langganan Azure Anda. Kemudian kembali ke Visual Studio Code dan tunggu proses masuk selesai.
9. Jalankan perintah berikut untuk membuat daftar lokasi Azure.

    ```powershell
    az account list-locations -o table
    ```

10. Pada output, temukan nilai **Name** yang sesuai dengan lokasi grup sumber daya Anda (misalnya, untuk *East US* nama yang sesuai adalah *eastus*).
11. Dalam skrip **setup.cmd**, ubah deklarasi variabel **subscription_id**, **resource_group**, dan **location** dengan nilai yang sesuai untuk ID langganan Anda, nama grup sumber daya, dan nama lokasi. Kemudian simpan perubahan Anda.
12. Di terminal untuk folder **02-search-skill**, masukkan perintah berikut untuk menjalankan skrip:

    ```powershell
    ./setup
    ```

    > **Catatan**: Jika skrip gagal, pastikan Anda menyimpannya dengan nama variabel yang benar dan coba lagi.

13. Saat skrip selesai, tinjau output yang ditampilkan dan catat informasi berikut tentang sumber daya Azure Anda (Anda akan memerlukan nilai ini nanti):
    - Nama akun penyimpanan
    - String koneksi penyimpanan
    - Titik akhir layanan Pencarian
    - Kunci admin layanan Pencarian
    - Kunci kueri layanan Pencarian

14. Di portal Azure, refresh grup sumber daya dan pastikan grup sumber daya tersebut berisi akun Azure Storage, sumber daya Layanan Azure AI, dan sumber daya Pencarian Azure AI.

## Membuat solusi pencarian

Sekarang setelah Anda memiliki sumber daya Azure yang diperlukan, Anda dapat membuat solusi pencarian yang terdiri dari komponen berikut:

- **Sumber data** yang mereferensikan dokumen di kontainer penyimpanan Azure Anda.
- **Kumpulan keterampilan** yang menentukan alur pengayaan keterampilan untuk mengekstrak bidang yang dihasilkan AI dari dokumen.
- **Indeks** yang menentukan kumpulan rekaman dokumen yang dapat dicari.
- **Pengindeks** yang mengekstrak dokumen dari sumber data, menerapkan kumpulan keterampilan, dan mengisi indeks.

Dalam latihan ini, Anda akan menggunakan antarmuka REST Pencarian Azure AI untuk membuat komponen ini dengan mengirimkan permintaan JSON.

1. Di Visual Studio Code, di folder **02-search-skill**, luaskan folder **create-search**, dan pilih **data_source.json**. File ini berisi definisi JSON untuk sumber data bernama **margies-custom-data**.
2. Ganti tempat penampung **YOUR_CONNECTION_STRING** dengan string koneksi untuk akun Azure storage Anda, yang akan terlihat seperti berikut:

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *Anda dapat menemukan string koneksi di halaman **Kunci akses** untuk akun penyimpanan Anda di portal Azure.*

3. Simpan dan tutup file JSON yang diperbarui.
4. Dalam folder **create-search**, buka **skillset.json**. File ini berisi definisi JSON untuk kumpulan keterampilan bernama **margies-custom-skillset**.
5. Di bagian atas definisi kumpulan kemampuan, di elemen **cognitiveServices**, ganti tempat penampung **YOUR_AI_SERVICES_KEY** dengan salah satu kunci untuk sumber daya Layanan Azure AI Anda.

    *Anda dapat menemukan kunci di halaman **Kunci dan Titik Akhir** untuk sumber daya Layanan Azure AI Anda di portal Azure.*

6. Simpan dan tutup file JSON yang diperbarui.
7. Dalam folder **create-search**, buka **index.json**. File ini berisi definisi JSON untuk indeks bernama **margies-custom-index**.
8. Tinjau JSON untuk indeks, lalu tutup file tanpa membuat perubahan apa pun.
9. Dalam folder **create-search**, buka **indexer.json**. File ini berisi definisi JSON untuk pengindeks bernama **margies-custom-indexer**.
10. Tinjau JSON untuk pengindeks, lalu tutup file tanpa membuat perubahan apa pun.
11. Dalam folder **create-search**, buka **create-search.cmd**. Skrip batch ini menggunakan utilitas cURL untuk mengirimkan definisi JSON ke antarmuka REST untuk sumber daya Pencarian Azure AI Anda.
12. Ganti tempat penampung variabel **YOUR_SEARCH_URL** dan **YOUR_ADMIN_KEY** dengan **Url** dan salah satu **kunci admin** untuk sumber daya Pencarian Azure AI Anda.

    *Anda dapat menemukan nilai ini di halaman **Gambaran umum** dan **Kunci** untuk sumber daya Pencarian Azure AI di portal Azure.*

13. Simpan file batch yang diperbarui.
14. Klik kanan folder **create-search** dan pilih **Buka di Terminal Terpadu**.
15. Di panel terminal untuk folder **create-search**, masukkan perintah berikut, jalankan skrip batch.

    ```powershell
    ./create-search
    ```

16. Saat skrip selesai, di portal Azure pada halaman untuk sumber daya Pencarian Azure AI, pilih halaman **Pengindeks** dan tunggu hingga proses pengindeksan selesai.

    *Anda dapat memilih **Refresh** untuk melacak kemajuan operasi pengindeksan. Mungkin perlu satu menit atau lebih untuk menyelesaikannya.*

## Cari indeks

Sekarang Anda memiliki indeks, Anda dapat mencarinya.

1. Di bagian atas blade untuk sumber daya Pencarian Azure AI Anda, pilih **Penjelajah pencarian**.
2. Di Penjelajah pencarian, di kotak **String kueri**, masukkan string kueri berikut, lalu pilih **Cari**.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    Kueri ini mengambil **URL**, **sentimen**, dan **frasa kunci** untuk semua dokumen yang menyebutkan *London* yang ditulis oleh *Peninjau* yang memiliki label **sentimen** positif (dengan kata lain, ulasan positif yang menyebutkan London)

## Membuat Azure Function untuk keterampilan kustom

Solusi pencarian mencakup sejumlah kemampuan AI bawaan yang memperkaya indeks dengan informasi dari dokumen, seperti skor sentimen dan daftar frasa kunci yang terlihat dalam tugas sebelumnya.

Anda dapat meningkatkan indeks lebih lanjut dengan membuat keterampilan kustom. Misalnya, mungkin berguna untuk mengidentifikasi kata-kata yang paling sering digunakan di setiap dokumen, tetapi tidak ada keterampilan bawaan yang menawarkan fungsionalitas ini.

Untuk menerapkan fungsionalitas hitungan kata sebagai keterampilan kustom, Anda akan membuat Azure Function dalam bahasa pilihan Anda.

> **Catatan**: Dalam latihan ini, Anda akan membuat fungsi Node.JS sederhana menggunakan kemampuan pengeditan kode di portal Microsoft Azure. Dalam solusi produksi, Anda biasanya akan menggunakan lingkungan pengembangan seperti Visual Studio Code untuk membuat aplikasi fungsi dalam bahasa pilihan Anda (misalnya C#, Python, Node.JS, atau Java) dan menerbitkannya ke Azure sebagai bagian dari proses DevOps.

1. Di Portal Microsoft Azure, pada halaman **Beranda**, buat sumber daya **Aplikasi Fungsi** baru dengan pengaturan berikut:
    - **Langganan**: *Langganan Anda*
    - **Grup sumber daya**: *Grup sumber daya yang sama dengan sumber daya Pencarian Azure AI Anda*
    - **Nama Aplikasi Fungsi**: *Nama yang unik*
    - **Terbitkan**: Kode
    - **Tumpukan runtime bahasa umum**: Node.js
    - **Versi**: 18 LTS
    - **Wilayah**: *Wilayah yang sama dengan sumber daya Pencarian Azure AI Anda*

2. Tunggu hingga penyebaran selesai, lalu buka sumber daya Aplikasi Fungsi yang disebarkan.
3. Di halaman **Gambaran umum** pilih opsi **Buat di portal Azure** untuk membuat fungsi baru dengan pengaturan berikut:
    - **Menyiapkan lingkungan pengembangan**"
        - **Lingkungan pengembangan**: Kembangkan di portal
    - **Pilih templat**"
        - **Templat**: Pemicu HTTP
    - **Detail templat**:
        - **Fungsi Baru**: wordcount
        - **Tingkat otorisasi**: Fungsi
4. Tunggu hingga fungsi *wordcount* dibuat. Kemudian di halamannya, pilih tab **Kode + Uji**.
5. Ganti kode fungsi default dengan kode berikut:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. Simpan fungsi lalu buka panel **Uji/Jalankan**.
7. Di panel **Uji/Jalankan**, ganti **Isi** yang ada dengan JSON berikut, yang mencerminkan skema yang diharapkan oleh keterampilan Pencarian Azure AI di mana catatan yang berisi data untuk satu atau beberapa dokumen dikirimkan untuk diproses:

    ```json
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```

8. Klik **Jalankan** dan lihat konten respons HTTP yang dikembalikan oleh fungsi Anda. Ini mencerminkan skema yang diharapkan oleh Pencarian Azure AI saat menggunakan keterampilan, di mana respons untuk setiap dokumen dikembalikan. Dalam hal ini, respons terdiri dari hingga 10 istilah di setiap dokumen dalam urutan turun seberapa sering mereka muncul:

    ```json
    {
        "values": [
        {
            "recordId": "a1",
            "data": {
                "text": [
                "tiger",
                "burning",
                "bright",
                "darkness",
                "night"
                ]
            }
        },
        {
            "recordId": "a2",
            "data": {
                "text": [
                    "rain",
                    "spain",
                    "stays",
                    "mainly",
                    "plains",
                    "thats",
                    "youll",
                    "find"
                ]
            }
        }
        ]
    }
    ```

9. Tutup panel **Uji/Jalankan** dan di panel fungsi **wordcount**, klik **Dapatkan URL fungsi**. Kemudian salin URL untuk kunci default ke clipboard. Anda akan membutuhkan ini di prosedur berikutnya.

## Menambahkan keterampilan kustom ke solusi pencarian

Sekarang Anda perlu menyertakan fungsi Anda sebagai keterampilan kustom dalam set keterampilan solusi pencarian, dan memetakan hasil yang dihasilkannya ke bidang dalam indeks. 

1. Di Visual Studio Code, di folder **02-search-skill/update-search**, buka file **update-skillset.json**. Ini berisi definisi JSON dari set keterampilan.
2. Tinjau definisi set keterampilan. Ini termasuk keterampilan yang sama seperti sebelumnya, serta keterampilan **WebApiSkill** baru bernama **get-top-words**.
3. Edit definisi keterampilan **get-top-words** untuk mengatur nilai **uri** ke URL untuk fungsi Azure Anda (yang Anda salin ke clipboard di prosedur sebelumnya), menggantikan **YOUR-FUNCTION-APP-URL**.
4. Di bagian atas definisi kumpulan kemampuan, di elemen **cognitiveServices**, ganti tempat penampung **YOUR_AI_SERVICES_KEY** dengan salah satu kunci untuk sumber daya Layanan Azure AI Anda.

    *Anda dapat menemukan kunci di halaman **Kunci dan Titik Akhir** untuk sumber daya Layanan Azure AI Anda di portal Azure.*

5. Simpan dan tutup file JSON yang diperbarui.
6. Di folder **update-search**, buka **update-index.json**. File ini berisi definisi JSON untuk indeks **margies-custom-index**, dengan bidang tambahan bernama **top_words** di bagian bawah definisi indeks.
7. Tinjau JSON untuk indeks, lalu tutup file tanpa membuat perubahan apa pun.
8. Di folder **update-search**, buka **update-index.json**. File ini berisi definisi JSON untuk **margies-custom-indexer**, dengan pemetaan tambahan untuk bidang **top_words**.
9. Tinjau JSON untuk pengindeks, lalu tutup file tanpa membuat perubahan apa pun.
10. Di folder **update-search**, buka **update-search.cmd**. Skrip batch ini menggunakan utilitas cURL untuk mengirimkan definisi JSON yang diperbarui ke antarmuka REST untuk sumber daya Pencarian Azure AI Anda.
11. Ganti tempat penampung variabel **YOUR_SEARCH_URL** dan **YOUR_ADMIN_KEY** dengan **Url** dan salah satu **kunci admin** untuk sumber daya Pencarian Azure AI Anda.

    *Anda dapat menemukan nilai ini di halaman **Gambaran umum** dan **Kunci** untuk sumber daya Pencarian Azure AI di portal Azure.*

12. Simpan file batch yang diperbarui.
13. Klik kanan folder **update-search** dan pilih **Buka di Terminal Terpadu**.
14. Di panel terminal untuk folder **update-search**, masukkan perintah berikut, jalankan skrip batch.

    ```powershell
    ./update-search
    ```

15. Saat skrip selesai, di portal Azure pada halaman untuk sumber daya Pencarian Azure AI, pilih halaman **Pengindeks** dan tunggu hingga proses pengindeksan selesai.

    *Anda dapat memilih **Refresh** untuk melacak kemajuan operasi pengindeksan. Mungkin perlu satu menit atau lebih untuk menyelesaikannya.*

## Cari indeks

Sekarang Anda memiliki indeks, Anda dapat mencarinya.

1. Di bagian atas blade untuk sumber daya Pencarian Azure AI Anda, pilih **Penjelajah pencarian**.
2. Di Penjelajah pencarian, ubah tampilan menjadi **tampilan JSON**, lalu kirimkan kueri pencarian berikut:

    ```json
    {
      "search": "Las Vegas",
      "select": "url,top_words"
    }
    ```

    Kueri ini mengambil bidang **url** dan **top_words** untuk semua dokumen yang menyebutkan *Las Vegas*.

## Informasi selengkapnya

Untuk mempelajari selengkapnya tentang membuat keterampilan kustom untuk Pencarian Azure AI, lihat [dokumentasi Pencarian Azure AI](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface).
