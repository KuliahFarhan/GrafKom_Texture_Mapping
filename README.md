# GrafKom Texture Mapping

## Ringkasan Proyek
Proyek ini menampilkan kubus 3D dengan texture mapping pada keenam sisi menggunakan WebGL.
Setiap sisi kubus memakai gambar yang berbeda dari folder park sehingga hasil akhirnya menyerupai cubemap manual.

Implementasi utama ada di file berikut:
- [tugas1.html](tugas1.html): versi pengembangan yang sudah diperluas.
- [tugas1-release.html](tugas1-release.html): versi awal (baseline) dari tugas.

Library matematika matriks/vector:
- [gl-matrix_2.js](gl-matrix_2.js) dipakai oleh versi pengembangan.
- [gl-matrix.js](gl-matrix.js) dipakai oleh versi release.

Catatan penting: pada proyek ini kita tidak mengubah isi library gl-matrix. Perubahan fokus di kode aplikasi WebGL pada [tugas1.html](tugas1.html).

## Tujuan Teknis
1. Menampilkan kubus 3D dengan 6 tekstur berbeda.
2. Memetakan file tekstur ke arah sumbu yang benar.
3. Menyediakan kontrol interaktif agar kubus bisa diputar untuk melihat sisi depan, belakang, kanan, kiri, atas, dan bawah.
4. Menangani kasus ukuran tekstur non-power-of-two agar tetap kompatibel di WebGL.

## Mapping Tekstur per Sisi Kubus
Mapping yang dipakai pada [tugas1.html](tugas1.html):

| File Gambar | Arah Sumbu | Sisi Kubus |
| --- | --- | --- |
| posx.jpg | +X | kanan (right) |
| negx.jpg | -X | kiri (left) |
| posy.jpg | +Y | atas (top) |
| negy.jpg | -Y | bawah (bottom) |
| posz.jpg | +Z | depan (front) |
| negz.jpg | -Z | belakang (back) |

Semua file berada di folder [park/readme.txt](park/readme.txt) dan file gambar terkait di folder park.

## Cara Kerja Program Secara Detail

### 1. Shader Pipeline
File [tugas1.html](tugas1.html) memiliki dua shader:
1. Vertex shader
	1. Menerima atribut posisi vertex dan koordinat tekstur.
	2. Menghitung warna sederhana berdasarkan normal (komponen z normal setelah transformasi normal matrix).
	3. Meneruskan texCoords ke fragment shader.
2. Fragment shader
	1. Mengambil warna piksel dari sampler texture 2D.
	2. Mengalikan warna texture dengan warna dari vertex shader.

Hasilnya: objek tetap punya nuansa pencahayaan sederhana sambil menampilkan detail gambar tekstur.

### 2. Data Geometri Kubus
Di [tugas1.html](tugas1.html), kubus didefinisikan sebagai 6 buah face kuadrat.
Setiap face menyimpan:
1. label sisi.
2. direction arah sumbu.
3. textureURL file gambar.
4. normal vector sisi.
5. vertices (4 titik, digambar TRIANGLE_FAN).
6. texCoords (koordinat UV).

Pendekatan ini membuat mapping menjadi eksplisit dan mudah di-debug karena satu face benar-benar memiliki paket data sendiri.

### 3. Matriks Transformasi
Program memakai matriks berikut:
1. projection: perspektif kamera.
2. modelview: posisi kamera dan rotasi objek.
3. modelviewProj: hasil projection * modelview.
4. normalMatrix: turunan dari modelview untuk transform normal.

Urutan render utama pada draw:
1. clear color dan depth buffer.
2. set kamera dengan lookAt.
3. terapkan rotasi rotateX/rotateY/rotateZ.
4. loop semua face dan panggil drawFace.

### 4. Render Per Face
Pada drawFace di [tugas1.html](tugas1.html):
1. Upload vertex face ke ARRAY_BUFFER.
2. Upload normal face ke uniform.
3. Hitung modelviewProj dan kirim ke shader.
4. Hitung normalMatrix dan kirim ke shader.
5. Upload texCoords face.
6. Bind texture object milik face itu.
7. Draw dengan TRIANGLE_FAN (4 vertex).

Dengan cara ini, setiap face benar-benar dirender dengan tekstur yang tepat.

### 5. Loading Tekstur Asinkron
Tekstur dimuat satu per satu menggunakan Image.onload.
Alurnya:
1. createTexture untuk index face.
2. texImage2D dari image.
3. setup filtering berdasarkan dimensi gambar.
4. increment loadedCount.
5. jika semua sudah selesai, baru draw.

Kelebihan alur ini: mencegah draw saat texture belum siap.

### 6. Power-of-Two vs Non-Power-of-Two
Fungsi setupTextureFiltering di [tugas1.html](tugas1.html):
1. Jika lebar dan tinggi power-of-two:
	1. generateMipmap.
	2. min filter LINEAR_MIPMAP_LINEAR.
2. Jika bukan power-of-two:
	1. wrap S/T di CLAMP_TO_EDGE.
	2. min/mag filter LINEAR.

Ini penting karena aturan WebGL untuk NPOT texture lebih ketat.

### 7. Kontrol Interaktif
Versi pengembangan mendukung:
1. Keyboard
	1. Panah: putar X/Y.
	2. PageUp/PageDown: putar Z.
	3. Home/Enter: reset rotasi.
2. Mouse drag di canvas
	1. Drag horizontal mengubah rotateY.
	2. Drag vertikal mengubah rotateX.
3. Tombol preset view
	1. Depan, Belakang, Kanan, Kiri, Atas, Bawah, Reset.

Preset view membantu verifikasi cepat apakah mapping sisi sudah benar.

## Perbedaan dengan Kode Awal (tugas1-release)

### Gambaran Singkat
Versi release di [tugas1-release.html](tugas1-release.html) masih tahap awal:
1. Baru menggambar satu face depan.
2. Baru memuat satu tekstur.
3. Belum ada struktur data per-face.

Versi pengembangan di [tugas1.html](tugas1.html):
1. Menggambar semua 6 sisi kubus.
2. Memuat 6 tekstur sesuai arah sumbu.
3. Menambahkan interaksi mouse dan tombol preset.

### Tabel Perbandingan Detail
| Aspek | tugas1-release | tugas1 (versi pengembangan) |
| --- | --- | --- |
| File library | gl-matrix.js | gl-matrix_2.js |
| Jumlah tekstur aktif | 1 (index 0) | 6 (semua sisi) |
| Objek yang dirender | 1 square (front face) | 6 face membentuk kubus penuh |
| Struktur data sisi | Tidak ada objek per sisi | Ada cubeFaces berisi label, arah, normal, vertices, texCoords, textureURL |
| Fungsi render | drawSquare tunggal | draw + drawFace per sisi |
| Loading texture | Satu gambar onload | loadAllTextures + loadTexture(index) |
| Kompatibilitas NPOT | Tidak ditangani spesifik | Ditangani lewat setupTextureFiltering |
| Input interaktif | Keyboard saja | Keyboard + mouse drag + tombol preset |
| UI halaman | Pesan instruksi sederhana | Ada panel tombol kontrol sisi |
| Guard render | Tidak ada guard loadedCount | Ada guard agar draw saat texture lengkap |

## Apakah Ada Perubahan di JS?
Ya, ada perubahan signifikan pada JavaScript aplikasi di [tugas1.html](tugas1.html).

Ringkasan perubahan JS:
1. Model data kubus diubah dari pendekatan satu-face menjadi data-driven 6 face.
2. Proses render diubah menjadi loop per face.
3. Ditambah sistem loading multi-texture asynchronous.
4. Ditambah pengecekan power-of-two untuk konfigurasi texture filtering.
5. Ditambah event handling mouse drag.
6. Ditambah fungsi preset sudut pandang (front/back/right/left/top/bottom/reset).
7. Ditambah elemen kontrol tombol di HTML dan listener-nya di JS.

Yang tidak berubah:
1. Konsep dasar shader (attribute, varying, sampler2D).
2. Pemakaian matriks projection/modelview/normal matrix.
3. Mekanisme rotasi keyboard inti.

## Langkah Menjalankan Proyek
1. Buka folder proyek di browser/editor yang mendukung static file.
2. Pastikan file HTML dan folder park berada di struktur yang benar.
3. Jalankan [tugas1.html](tugas1.html).
4. Tunggu status memuat tekstur selesai.
5. Interaksi:
	1. Drag mouse pada canvas untuk rotasi bebas.
	2. Gunakan tombol preset untuk langsung melihat sisi tertentu.
	3. Gunakan keyboard untuk rotasi incremental.

## Checklist Validasi Hasil
Gunakan checklist ini setelah menjalankan [tugas1.html](tugas1.html):
1. Semua 6 tekstur berhasil tampil (tidak hitam/blank).
2. Tombol Depan menampilkan sisi +Z (posz).
3. Tombol Belakang menampilkan sisi -Z (negz).
4. Tombol Kanan menampilkan sisi +X (posx).
5. Tombol Kiri menampilkan sisi -X (negx).
6. Tombol Atas menampilkan sisi +Y (posy).
7. Tombol Bawah menampilkan sisi -Y (negy).
8. Drag mouse merespons rotasi tanpa glitch besar.
9. Keyboard dan tombol reset tetap bekerja.

## Penutup
Inti pengembangan dari kode awal adalah perubahan dari demo satu bidang bertekstur menjadi kubus utuh enam sisi dengan kontrol eksplorasi sudut pandang yang lengkap.
Semua perubahan utama berada di JavaScript aplikasi [tugas1.html](tugas1.html), bukan di isi library gl-matrix.

## Peta Kode dan Fungsi Tiap Bagian

Bagian ini merangkum "apa saja kode yang ada" dan "fungsi tiap kode" pada implementasi utama di [tugas1.html](tugas1.html).

### A. Struktur Umum File
1. HTML dasar
	1. Menyediakan elemen canvas tempat WebGL merender kubus.
	2. Menyediakan panel tombol kontrol sudut pandang.
2. CSS
	1. Mengatur gaya halaman dan tombol kontrol agar interaksi lebih nyaman.
3. Script library
	1. Memuat gl-matrix untuk operasi matriks/vector 3D.
4. Script utama aplikasi WebGL
	1. Berisi shader, data geometri, inisialisasi GPU, loading texture, render loop, dan event handler.

### B. Konstanta Shader
1. vertexShaderSource
	1. Input: posisi vertex, texture coordinates.
	2. Uniform: transform matrix, normal matrix, normal sisi, diffuse color.
	3. Output: posisi clip-space dan data interpolasi ke fragment shader.
2. fragmentShaderSource
	1. Input: interpolated color dan texCoords.
	2. Uniform: sampler2D.
	3. Output: warna akhir fragmen = warna texture x warna pencahayaan sederhana.

### C. Variabel Global Rendering
1. gl
	1. Context WebGL aktif.
2. uTransformMatrixLoc, uNormalMatrixLoc, uNormalLoc
	1. Handle lokasi uniform pada shader program.
3. projection, modelview, modelviewProj, normalMatrix
	1. Kumpulan matriks untuk kamera, transform objek, dan transform normal.
4. rotateX, rotateY, rotateZ
	1. State rotasi objek dari input user.
5. isDragging, lastMouseX, lastMouseY
	1. State interaksi drag mouse.
6. cubeFaces
	1. Data inti tiap sisi kubus (arah, normal, vertices, texCoords, textureURL).
7. textureObjects, loadedCount
	1. Menyimpan objek tekstur GPU dan progres loading.
8. coordsBuf, tCoordsBuf
	1. Buffer GPU untuk koordinat vertex dan UV.

### D. Fungsi Utama dan Perannya
1. draw
	1. Fungsi render frame utama.
	2. Menolak render bila tekstur belum lengkap.
	3. Membangun modelview dari kamera + rotasi.
	4. Loop semua face lalu memanggil drawFace.
2. drawFace(face, texObj)
	1. Upload vertices face.
	2. Set normal face.
	3. Hitung dan kirim transform matrices ke shader.
	4. Upload texCoords face.
	5. Bind texture face.
	6. Draw face dengan TRIANGLE_FAN.
3. initGL
	1. Compile/link shader program.
	2. Setup atribut buffer posisi dan texCoords.
	3. Ambil lokasi uniform dan set nilai awal.
	4. Aktifkan DEPTH_TEST.
	5. Set proyeksi perspektif.
	6. Atur flip Y untuk texture upload.
	7. Mulai load semua texture.
4. loadAllTextures
	1. Memanggil loadTexture untuk semua index face.
5. loadTexture(index)
	1. Membuat objek Image dan menunggu onload.
	2. Setelah gambar siap: buat texture GPU, upload piksel, setup filtering.
	3. Update loadedCount dan status teks.
	4. Jika semua selesai, panggil draw.
6. isPowerOfTwo(value)
	1. Mengecek apakah ukuran sisi texture termasuk power-of-two.
7. setupTextureFiltering(width, height)
	1. Jika POT: generate mipmap + filter mipmap.
	2. Jika NPOT: clamp-to-edge + linear filtering (kompatibel WebGL).
8. updateLoadingStatus
	1. Menampilkan progres loading dan instruksi interaksi.
9. createProgram(gl, vShader, fShader)
	1. Compile vertex shader.
	2. Compile fragment shader.
	3. Link jadi shader program.
	4. Throw error jika compile/link gagal.
10. doKey(evt)
	1. Memproses rotasi via keyboard.
	2. Men-trigger redraw ketika rotasi berubah.
11. setViewPreset(direction)
	1. Set rotasi ke orientasi standar (front/back/right/left/top/bottom/reset).
	2. Memudahkan verifikasi mapping sisi.
12. setupMouseControls(canvas)
	1. Menangani mousedown/mousemove/mouseup/mouseleave.
	2. Mengubah rotateX/rotateY berdasarkan delta drag.
13. init
	1. Ambil context WebGL.
	2. Panggil initGL.
	3. Pasang event listener keyboard, mouse, dan tombol preset.

### E. Bagian yang Tidak Diubah
1. Library gl-matrix di [gl-matrix_2.js](gl-matrix_2.js) tidak diubah.
2. Seluruh logic pengembangan dilakukan di [tugas1.html](tugas1.html).