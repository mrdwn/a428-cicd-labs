
Latihan Membangun CI Pipeline dengan Jenkins

Latihan ini akan mengajarkan Anda bagaimana cara menggunakan Jenkins untuk mengorkestrasi aplikasi React App. Aplikasi ini menggunakan React dan Node.js untuk menampilkan halaman web dengan konten "Welcome to React" dan disertai pengujian untuk memeriksa berjalannya proses rendering pada aplikasi.

Tak masalah jika Anda bukan React atau Node.js Developer, latihan ini tetap dapat diikuti karena telah didesain untuk fokus pada praktik continuous integration, bukan pemrograman. Jika telah mengikuti DevOps Engineer Learning Path dari awal, seharusnya Anda akan lebih mudah untuk mengikuti latihan ini karena materinya banyak bersinggungan dengan konsep CI/CD, istilah jaringan, perintah Linux, dan Docker container.

Untuk mengikuti latihan ini, Anda bisa menggunakan sistem operasi Windows, Linux, ataupun macOS. Namun, pastikan beberapa tools berikut sudah terinstal di komputer Anda.

    Docker
    Git
    Visual Studio Code

Jika semua tools tersebut sudah terpasang, simak tahapan proses yang akan kita lakukan di latihan berikut ini.

    Dimulai dari menjalankan Jenkins melalui Docker.
    Kemudian, menyiapkan Jenkins wizard.
    Setelah itu, melakukan fork dan clone repository yang akan digunakan pada latihan ini.
    Lalu, membuat Pipeline project di Jenkins.
    Selanjutnya, menulis berkas Jenkinsfile untuk membuat build stage dan test stage di Jenkins pipeline.

Bagaimana, sudah siap? Kalau sudah, yuk kita mulai perjalanannya.


Menjalankan Jenkins di Docker

Di latihan ini, Anda akan menjalankan Jenkins sebagai Docker container dari jenkins/jenkins Docker image. Akan tetapi, karena image tersebut tidak bundle dengan Blue Ocean plugins (UX baru untuk Jenkins), kita juga akan menjalankan Blue Ocean sebagai Docker container. Silakan ikuti instruksi sesuai dengan sistem operasi yang Anda pakai.

    macOS dan Linux
    Windows

    Buka aplikasi Terminal pada komputer Anda.
    Buatlah sebuah bridge network di Docker menggunakan perintah berikut.

    docker network create jenkins

Kita akan menjalankan aplikasi React App menggunakan Docker container di dalam sebuah Docker container (lebih tepatnya di dalam Blue Ocean container–nanti akan dibahas). Praktik ini disebut dengan dind alias docker in docker. Jadi, silakan unduh dan jalankan docker:dind Docker image menggunakan perintah berikut.

    docker run \
      --name jenkins-docker \
      --rm \
      --detach \
      --privileged \
      --network jenkins \
      --network-alias docker \
      --env DOCKER_TLS_CERTDIR=/certs \
      --volume jenkins-docker-certs:/certs/client \
      --volume jenkins-data:/var/jenkins_home \
      --publish 2376:2376 \
      --publish 3000:3000 \
      docker:dind \
      --storage-driver overlay2

Berikut adalah penjelasan dari perintah di atas.

    --name: Menentukan nama Docker container yang akan digunakan untuk menjalankan image.

    --rm: Secara otomatis menghapus Docker container (yakni sebuah instance dari Docker image) saat dimatikan (shut down).

    --detach: Menjalankan Docker container di background. Meski begitu, instance ini dapat dihentikan nanti dengan menjalankan perintah docker stop jenkins-docker.

    --privileged: Menjalankan dind (docker in docker alias docker di dalam docker) saat ini memerlukan privileged access (akses istimewa) agar bisa berfungsi dengan baik. Persyaratan ini bisa jadi tak diperlukan dengan versi kernel Linux terbaru.

    --network jenkins: Ini berhubungan dengan network yang dibuat pada langkah sebelumnya.

    --network-alias docker: Membuat Docker di dalam Docker container tersedia sebagai hostname docker di dalam jenkins network.

    --env DOCKER_TLS_CERTDIR=/certs: Mengaktifkan penggunaan TLS di Docker server. Ini direkomendasikan karena kita menggunakan privileged container. Environment variable ini mengontrol root directory di mana Docker TLS certificates dikelola.

    --volume jenkins-docker-certs:/certs/client: Memetakan direktori /certs/client di dalam container ke Docker volume bernama jenkins-docker-certs.

    --volume jenkins-data:/var/jenkins_home: Memetakan direktori /var/jenkins_home di dalam container ke Docker volume bernama jenkins-data. Ini akan memungkinkan Docker container lain dikelola oleh Docker container’s Docker daemon ini untuk mount data dari Jenkins.

    --publish 2376:2376: Mengekspos Docker daemon port pada mesin host (komputer Anda). Ini berguna untuk mengeksekusi Docker command (perintah Docker) pada mesin host (komputer Anda) dalam mengontrol inner Docker daemon.

    --publish 3000:3000: Mengekspos port 3000 dari docker in docker container.

    docker:dind: Ini adalah image dari docker:dind itu sendiri. Image ini bisa diunduh sebelum dijalankan menggunakan perintah docker image pull docker:dind.

    --storage-driver overlay2: Storage driver untuk Docker volume. Lihat halaman "Docker Storage drivers” untuk berbagai opsi yang didukung.

    Apakah Anda merasa cukup pusing melihat perintah Docker di atas? Tenang. Tak masalah jika Anda tidak begitu mahir dalam hal Docker karena fokus dari latihan ini adalah pada praktik continuous integration, bukan pada Docker. Untuk saat ini, Anda bisa jalankan saja sesuai perintah saja.

Oke, sampai di sini, Docker container untuk Jenkins sudah berjalan. Namun, tugas kita belum selesai. Kita harus menjalankan Docker container lagi, kali ini untuk Blue Ocean (UX terbaru dari Jenkins). Untuk itu, buatlah sebuah Dockerfile dengan perintah berikut.

    nano Dockerfile

Catatan: Anda bisa menaruh Dockerfile di direktori yang Anda inginkan dengan pindah terlebih dahulu ke direktori tersebut menggunakan perintah cd path_directory. Misalnya, cd ~/Downloads/.
Salin konten berikut ini ke Dockerfile Anda.

    FROM jenkins/jenkins:2.346.1-jdk11
    USER root
    RUN apt-get update && apt-get install -y lsb-release
    RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
      https://download.docker.com/linux/debian/gpg
    RUN echo "deb [arch=$(dpkg --print-architecture) \
      signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
      https://download.docker.com/linux/debian \
      $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
    RUN apt-get update && apt-get install -y docker-ce-cli
    USER jenkins
    RUN jenkins-plugin-cli --plugins "blueocean:1.25.5 docker-workflow:1.28"

Simpan berkas tersebut dengan menekan CTRL+O, Enter, lalu CTRL+X.
Buat sebuah Docker image baru dari Dockerfile tadi dan berikan nama myjenkins-blueocean:2.346.1-1.

    docker build -t myjenkins-blueocean:2.346.1-1 .

Awas, tanda titik di akhir perintah jangan terlewat. Perintah di atas akan mengunduh official Jenkins Docker image secara otomatis.
Setelah itu, jalankan myjenkins-blueocean:2.346.1-1 image sebagai container di Docker menggunakan perintah berikut.

        docker run \
          --name jenkins-blueocean \
          --detach \
          --network jenkins \
          --env DOCKER_HOST=tcp://docker:2376 \
          --env DOCKER_CERT_PATH=/certs/client \
          --env DOCKER_TLS_VERIFY=1 \
          --publish 8080:8080 \
          --publish 50000:50000 \
          --volume jenkins-data:/var/jenkins_home \
          --volume jenkins-docker-certs:/certs/client:ro \
          --volume "$HOME":/home \
          --restart=on-failure \
          --env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" \
          myjenkins-blueocean:2.346.1-1 

    Berikut adalah penjelasan dari perintah di atas.

        --name: Menentukan nama Docker container yang akan digunakan untuk menjalankan image.

        --detach: Menjalankan Docker container di background.

        --network jenkins: Menghubungkan container ini dengan jenkins network yang dibuat sebelumnya. Ini membuat Docker daemon dari langkah sebelumnya tersedia ke container ini melalui hostname docker.

        --env DOCKER_HOST=tcp://docker:2376: Menentukan environment variable yang digunakan oleh docker, docker-compose, dan Docker tools lainnya untuk terhubung ke Docker daemon dari langkah sebelumnya.

        --publish 8080:8080: Memetakan (memublikasikan/mengekspos) port 8080 dari container saat ini ke port 8080 pada mesin host (komputer Anda). Angka pertama merepresentasikan port pada host, sedangkan yang terakhir mewakili port pada container. Jadi, jika Anda menentukan -p 49000:8080 untuk opsi ini, Anda akan mengakses Jenkins di mesin host melalui port 49000.

        --publish 50000:50000: Memetakan (mengekspos) port 50000 dari container saat ini ke port 50000 pada mesin host (komputer Anda). Ini hanya diperlukan jika Anda telah menyiapkan satu atau lebih inbound Jenkins agents di mesin lain yang berinteraksi dengan jenkins-blueocean container Anda (Jenkins "controller"). Inbound Jenkins agents berkomunikasi dengan Jenkins controller melalui TCP port 50000 secara default. Anda dapat mengubah port number ini pada Jenkins controller melalui halaman Configure Global Security.

        --volume jenkins-data:/var/jenkins_home: Memetakan direktori /var/jenkins_home pada container ke Docker volume dengan nama jenkins-data.

        --volume jenkins-docker-certs:/certs/client:ro: Memetakan direktori /certs/client pada container ke volume yang dibuat sebelumnya yang bernama jenkins-docker-certs. Ini membuat client TLS certificates untuk terhubung ke Docker daemon yang tersedia di path yang ditentukan oleh DOCKER_CERT_PATH environment variable.

        --volume "$HOME":/home: Memetakan direktori $HOME pada mesin host (komputer Anda, biasanya direktori /Users/<your-username>) ke direktori /home pada container.

        --restart=on-failure: Mengonfigurasi Docker container restart policy agar memulai kembali (restart) saat fail (terjadi kegagalan).

        --env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true": Mengizinkan checkout pada local repository.

        myjenkins-blueocean:2.346.1-1: Nama Docker image yang Anda buat di langkah sebelumnya.

Oke, Anda sudah berhasil menjalankan Jenkins di Docker. Anda bisa lanjut langsung ke tahapan Menyiapkan Jenkins Wizard.


Menyiapkan Jenkins Wizard

Sebelum bisa mengakses Jenkins, ada beberapa langkah yang harus kita lakukan terlebih dahulu. Saat pertama kali mengakses Jenkins instance baru, kita diminta untuk unlock menggunakan password yang dibuat secara otomatis.

    Buka browser Anda dan jalankan http://localhost:8080. Tunggu hingga halaman Unlock Jenkins muncul.

    Sebagaimana yang tertulis di sana, Anda diminta untuk menyalin password dari Jenkins log untuk memastikan bahwa Jenkins diakses dengan aman oleh administrator. 
    Tampilkan Jenkins console log dengan perintah berikut di Terminal/CMD.

        docker logs jenkins-blueocean

    Dari aplikasi Terminal/CMD, salin password yang ada di antara 2 rangkaian asterisk.dos:34f687ec8216f3fa64461e73728a4dcb20220715161514.jpeg

    Kembali ke halaman Unlock Jenkins di browser, paste password tersebut ke kolom Administrator password dan klik Continue.

    Setelah itu, halaman Customize Jenkins muncul. Pilih Install suggested plugins. Setup wizard menunjukkan progres bahwa Jenkins sedang dikonfigurasi dan plugin yang disarankan sedang diinstal. Proses ini mungkin dapat memakan waktu hingga beberapa menit.
    Catatan: Jika terdapat kegagalan saat proses instalasi plugin, Anda bisa klik Retry (untuk mengulangi proses instal plugin hingga berhasil) atau klik Continue (untuk melewati dan langsung melanjutkan ke langkah berikutnya).

    Setelah beres, Jenkins akan meminta Anda untuk membuat administrator user. Saat halaman Create First Admin User muncul, isilah sesuai keinginan Anda dan klik Save and Continue.

    Pada halaman Instance Configuration, pilih Save and Finish. Itu artinya, kita akan mengakses Jenkins dari url http://localhost:8080/. 

    Saat halaman Jenkins is ready muncul, klik tombol Start using Jenkins.
    Catatan: Halaman ini bisa jadi menunjukkan Jenkins is almost ready! Bila demikian, klik Restart. Jika halaman tersebut tidak refresh otomatis setelah beberapa menit, klik ikon refresh pada browser Anda secara manual.

    Jika perlu, Anda bisa log in ulang ke Jenkins menggunakan kredensial yang tadi dibuat.

Mantap! Jenkins sudah dapat dibuka dengan baik. Pasti sudah nggak sabar ‘kan ingin segera membuat CI Pipeline? Let’s go!

dos:3e655bf53a1d8ebefe77905223c92f9c20220715161514.jpeg


Fork dan Clone React App Repository

Dapatkan aplikasi React App dari Dicoding Academy GitHub repository dengan melakukan forking repository ke akun GitHub Anda sendiri dan kemudian clone ke local environment.

    Pastikan Anda telah login ke GitHub menggunakan akun pribadi. Apabila Anda belum memiliki akun GitHub, silakan daftar secara gratis melalui GitHub website.

    Fork React App repository di Dicoding Academy ke akun GitHub pribadi Anda. Ketika melakukan Fork, jangan lupa untuk tidak mencentang opsi copy the main branch only. Jika memerlukan bantuan dengan proses ini, lihatlah dokumentasi Fork A Repo di GitHub website untuk informasi lebih lanjut.

    Clone React App yang telah di-fork ke akun GitHub Anda ke local environment Anda. 

    Pertama, buka Terminal/CMD, pindah ke direktori yang Anda inginkan dengan perintah cd, misal cd ~/Downloads/.
    Kemudian, jalankan perintah berikut.

        git clone -b react-app https://github.com/USERNAME-AKUN-GITHUB-ANDA/a428-cicd-labs.git

    Setelah berhasil di-clone, buka folder a428-cicd-labs menggunakan Visual Studio Code. Buka Visual Studio Code -> File -> Open Folder -> Pilih Folder a428-cicd-labs -> Open.

Oke, kini local development environment Anda sudah siap.

dos:6456a1b914e288907c23254c676fe15520220715162210.jpeg

Mari kita lanjutkan!


Membuat Pipeline Project di Jenkins

Langkah pertama dalam membangun pipeline di Jenkins adalah dengan membuat Pipeline Project.

    Buka halaman Jenkins. Bila perlu, buka http://localhost:8080/ dan login kembali dengan kredensial Anda.
    Di bagian Welcome to Jenkins!, klik Create a job. Bila Anda tak melihatnya, klik New Item di sebelah kiri atas.
    Pada kolom Enter an item name, isikan dengan nama pipeline yang Anda inginkan, misal react-app. Kemudian, pilih Pipeline dan klik OK.
    Pada halaman berikutnya, isikan deskripsi singkat untuk pipeline Anda di kolom Description, misalnya Sebuah pipeline sederhana untuk proyek React App.
    Setelah itu, buka tab bertuliskan Pipeline di atas kolom Description yang akan membawa Anda scroll ke bawah ke bagian Pipeline.
    Pada bagian Definition, pilih opsi Pipeline script from SCM. Opsi ini menginstruksikan Jenkins untuk membuat Pipeline dari Source Control Management (SCM), yang mana berarti Git repository yang Anda clone sebelumnya ke local environment.
    Pada kolom SCM, pilih Git.
    Pada bagian Repository URL, isikan directory path (lokasi direktori) dari local repository untuk React App yang sebelumnya Anda clone, yakni dari user account atau home directory pada mesin host (komputer) Anda yang dipetakan ke direktori /home pada Jenkins container. Contohnya (sesuaikan dengan struktur direktori di komputer Anda).
        macOS
        Linux
        Windows

        /home/Documents/Belajar_Implementasi_CICD/Jenkins/a428-cicd-labs

    Catatan: Anda tak perlu menyertakan hostname komputer (/home/fikrihelmi17), melainkan cukup /home saja dan langsung ke direktori yang dituju.
    Setelah itu, scroll ke bawah. Pada bagian Branch Specifier (blank for 'any'), ubah menjadi */react-app. Ini mengartikan bahwa kita ingin fokus ke branch bernama react app (sesuai yang kita clone sebelumnya).
    Kemudian, klik tombol Save untuk menyimpan Pipeline project Anda.

Anda berhasil membuat Pipeline Project di Jenkins. Proses selanjutnya adalah menulis Jenkinsfile untuk membuat CI Pipeline. Bersiaplah!


Membuat Jenkins Pipeline dengan Jenkinsfile

Kini Anda sudah siap membuat Pipeline untuk mengotomatisasi continuous integration aplikasi React App di Jenkins. Jenkins Pipeline akan dibuat menggunakan Jenkinsfile, kemudian di-commit ke Git local repository. 

Jenkinsfile adalah sebuah text file yang berisi definisi dari Jenkins Pipeline. Ini merupakan fondasi dari "Pipeline-as-Code" yang memperlakukan CI/CD pipeline sebagai bagian dari source code aplikasi. 

Jenkins Pipeline mendukung 2 bentuk sintaks, yakni Declarative (diperkenalkan di Pipeline 2.5) dan Scripted Pipeline. Berikut contoh perbedaan antara keduanya.

dos:a68388bebef6dc2c6bdc842466be3eae20220808155925.png

Oke, cukup perkenalan dengan Jenkinsfile. Akan lebih baik jika kita langsung praktik. Melalui Jenkinsfile, pertama Anda akan mengunduh Node Docker image dan menjalankannya sebagai Docker container, ialah yang akan mem-build aplikasi React App Anda. Setelahnya, tambahkan “Build” stage, yakni tahapan untuk memulai proses Build aplikasi.

    Buka Visual Studio Code yang telah Anda buka sebelumnya untuk folder a428-cicd-labs (React App). Kemudian, buka Terminal window di dalam VS Code dengan memilih Terminal -> New Terminal. Pastikan terminal menunjukkan direktori yang sesuai dengan source code aplikasi React App.
    dos:08d76790fbc7a9c307690ef98d89761420220715163311.png
    Pada panel Explorer di sebelah kiri, arahkan mouse pointer Anda ke nama folder A428-CICD-LABS, dan klik tombol New File.
    dos:d8d6e8917f6abdfc9396e477db9b492e20220715163354.png
    Berikan nama Jenkinsfile (tanpa ekstensi), lalu tekan Enter.
    dos:4a18d854b2af9c805982625a4de4644420220715163425.png
    Setelah berkas Jenkinsfile terbuka, salinlah kode Declarative Pipeline di bawah ini ke berkas tersebut.

    pipeline {
        agent {
            docker {
                image 'node:16-buster-slim' 
                args '-p 3000:3000' 
            }
        }
        stages {
            stage('Build') { 
                steps {
                    sh 'npm install' 
                }
            }
        }
    }