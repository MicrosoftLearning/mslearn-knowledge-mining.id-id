---
lab:
  title: Menambahkan ke indeks menggunakan API pendorongan
---

# Menambahkan ke indeks menggunakan API pendorongan

Anda ingin menjelajahi cara membuat indeks Pencarian Azure AI dan mengunggah dokumen ke indeks tersebut menggunakan kode C#.

Dalam latihan ini, Anda akan mengkloning solusi C# yang ada dan menjalankannya untuk menghasilkan ukuran batch yang optimal untuk mengunggah dokumen. Anda kemudian akan menggunakan ukuran batch ini dan mengunggah dokumen secara efektif menggunakan pendekatan berutas.

> **Catatan** Untuk menyelesaikan latihan ini, Anda memerlukan langganan Microsoft Azure. Jika Anda belum memilikinya, Anda dapat mendaftar uji coba gratis di [https://azure.com/free](https://azure.com/free?azure-portal=true) .

## Siapkan sumber daya Azure Anda

Untuk menghemat waktu Anda, pilih templat Azure Resource Manager ini untuk membuat sumber daya yang Anda perlukan nanti dalam latihan:

1. [Sebarkan sumber daya ke Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F07-exercise-add-to-index-use-push-api%20lab-files%2Fazuredeploy.json) - pilih tautan ini untuk membuat sumber daya Azure AI Anda.
    ![Cuplikan layar opsi yang ditampilkan saat menyebarkan sumber daya ke Azure.](../media/07-media/deploy-azure-resources.png)
1. Di **Grup sumber daya**, pilih ** Buat baru **, beri nama **cog-search-language-exe **.
1. Di **Wilayah**, pilih [wilayah yang didukung](/azure/ai-services/language-service/custom-text-classification/service-limits#regional-availability) yang dekat dengan Anda.
1. **Awalan Sumber Daya** harus unik secara global, masukkan angka acak dan awalan karakter huruf kecil, misalnya, **acs118245**.
1. Di **Lokasi**, pilih wilayah yang sama dengan yang Anda pilih di atas.
1. Pilih **Tinjau + buat**.
1. Pilih **Buat**.
1. Setelah penyebaran selesai, pilih **Buka grup sumber daya** untuk melihat semua sumber daya yang telah Anda buat.

    ![Cuplikan layar yang menampilkan semua sumber daya Azure yang disebarkan.](../media/07-media/azure-resources-created.png)

## Menyalin informasi REST API layanan Pencarian Azure AI

1. Dalam daftar sumber daya, pilih layanan pencarian yang Anda buat. Pada contoh di atas, **acs118245-search-service**.
1. Salin nama layanan pencarian ke dalam file teks.

    ![Cuplikan layar bagian kunci dari layanan pencarian.](../media/07-media/search-api-keys-exercise-version.png)
1. Di sebelah kiri, pilih **Kunci**, lalu salin **Kunci admin utama** ke dalam file teks yang sama.

## Mengunduh kode contoh untuk digunakan di Visual Studio Code

Anda akan menjalankan kode sampel Azure menggunakan Visual Studio Code. File kode telah disediakan dalam repositori GitHub.

1. Memulai Visual Studio Code.
1. Buka palet (SHIFT+CTRL+P) dan jalankan **Git: Perintah klon** untuk mengkloning repositori `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` ke folder lokal (tidak masalah folder mana).
1. Setelah repositori dikloning, buka folder di Visual Studio Code.
1. Tunggu sementara file tambahan diinstal untuk mendukung proyek kode C# di repositori.

    > **Catatan**: Jika Anda diminta untuk menambahkan aset yang diperlukan guna membangun dan men-debug, pilih **Tidak Sekarang**.

1. Pada navigasi di sebelah kiri, perluas folder **optimize-data-indexing/v11/OptimizeDataIndexing**, lalu pilih file **appsettings.json**.

    ![Cuplikan layar yang menampilkan konten file appsettings.json.](../media/07-media/update-app-settings.png)
1. Tempelkan nama layanan pencarian dan kunci admin utama Anda.

    ```json
    {
      "SearchServiceUri": "https://acs118245-search-service.search.windows.net",
      "SearchServiceAdminApiKey": "YOUR_SEARCH_SERVICE_KEY",
      "SearchIndexName": "optimize-indexing"
    }
    ```

    File pengaturan akan terlihat seperti di atas.
1. Simpan perubahan dengan menekan **CTRL + S**.
1. Klik kanan folder **OptimizeDataIndexing**, lalu pilih **Open in Integrated Terminal (Buka di Terminal Terintegrasi)**.
1. Di terminal, masukkan `dotnet run`, lalu tekan **Enter**.

    ![Cuplikan layar yang menampilkan aplikasi yang berjalan di Visual Studio Code dengan pengecualian.](../media/07-media/debug-application.png)
Output menunjukkan bahwa ukuran batch optimal untuk kasus ini adalah 900 dokumen. Karena mencapai 6,071 MB per detik.

## Edit kode untuk menerapkan utas dan strategi backoff serta coba lagi

Ada kode yang dikomentari yang siap mengubah aplikasi untuk menggunakan utas untuk mengunggah dokumen ke indeks pencarian.

1. Pastikan Anda telah memilih **Program.cs**.

    ![Cuplikan layar Visual Studio Code yang menampilkan file Program.cs.](../media/07-media/edit-program-code.png)
1. Beri komentar pada baris 37 dan 38 seperti ini:

    ```csharp
    //Console.WriteLine("{0}", "Finding optimal batch size...\n");
    //await TestBatchSizesAsync(searchClient, numTries: 3);
    ```

1. Batalkan komentar pada baris 44 hingga 48.

    ```csharp
    Console.WriteLine("{0}", "Uploading using exponential backoff...\n");
    await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8);

    Console.WriteLine("{0}", "Validating all data was indexed...\n");
    await ValidateIndexAsync(indexClient, indexName, numDocuments);
    ```

    Kode yang mengontrol ukuran batch dan jumlah utas adalah `await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8)`. Ukuran batch adalah 1000 dan utasnya delapan.

    ![Cuplikan layar yang menampilkan semua kode yang diedit.](../media/07-media/thread-code-ready.png)
    Kode Anda akan terlihat seperti di atas.

1. Simpan perubahan Anda, tekan **CTRL**+**S**.
1. Pilih terminal Anda, lalu tekan tombol apa pun untuk mengakhiri proses yang sedang berjalan jika Anda belum melakukannya.
1. Jalankan `dotnet run` di terminal.

    Aplikasi akan memulai delapan utas, dan ketika setiap utas selesai menulis pesan baru ke konsol:

    ```powershell
    Finished a thread, kicking off another...
    Sending a batch of 1000 docs starting with doc 57000...
    ```

    Setelah 100.000 dokumen diunggah, aplikasi menulis ringkasan (mungkin perlu waktu beberapa saat untuk menyelesaikannya):

    ```powershell
    Ended at: 9/1/2023 3:25:36 PM
    
    Upload time total: 00:01:18:0220862
    Upload time per batch: 780.2209 ms
    Upload time per document: 0.7802 ms
    
    Validating all data was indexed...
    
    Waiting for service statistics to update...
    
    Document Count is 100000
    
    Waiting for service statistics to update...
    
    Index Statistics: Document Count is 100000
    Index Statistics: Storage Size is 71453102
    
    ``````

Jelajahi kode dalam prosedur `TestBatchSizesAsync` untuk melihat bagaimana kode menguji kinerja ukuran batch.

Jelajahi kode dalam prosedur `IndexDataAsync` untuk melihat bagaimana kode mengelola threading.

Jelajahi kode di `ExponentialBackoffAsync` untuk melihat bagaimana kode menerapkan strategi percobaan ulang backoff eksponensial.

Anda dapat mencari dan memverifikasi bahwa dokumen telah ditambahkan ke indeks di portal Azure.

![Cuplikan layar yang menampilkan indeks pencarian dengan 100.000 dokumen.](../media/07-media/check-search-service-index.png)

## Pembersihan

Sekarang setelah Anda menyelesaikan latihan, hapus semua sumber daya yang tidak lagi Anda perlukan. Mulailah dengan kode yang dikloning ke mesin Anda. Kemudian hapus sumber daya Azure.

1. Di portal Microsoft Azure, pilih **Grup sumber daya**.
1. Pilih grup sumber daya yang telah Anda buat untuk latihan ini.
1. Pilih **Hapus grup sumber daya**. 
1. Konfirmasikan penghapusan, lalu pilih **Hapus**.
1. Pilih sumber daya yang tidak Anda perlukan, lalu pilih **Hapus**.
