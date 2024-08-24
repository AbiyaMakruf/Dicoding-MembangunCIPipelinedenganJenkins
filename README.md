# Dicoding-Membangun CI Pipelined engan Jenkins

## Penilaian Proyek

Proyek ini berhasil mendapatkan bintang 4/5 pada submission dicoding course Belajar Implementasi CI/CD.

<img src="https://raw.githubusercontent.com/AbiyaMakruf/Dicoding-MembangunCIPipelinedenganJenkins/main/images/nilai.png" width="500">

Kriteria tambahan yang saya kerjakan sehingga mendapat nilai terbaik:

1. Menggunakan NGINX sebagai reverse proxy server. Supaya, selain bisa diakses dari host melalui port default-nya yakni http://localhost:8080/ (atau port 49000 jika Anda menerapkan saran di poin berikutnya), halaman Jenkins juga bisa diakses melalui port 9000 (NGINX) yakni http://localhost:9000/. NGINX boleh dijalankan melalui docker container atau langsung di local server (Terminal/CMD).
2. Mengubah port default untuk menjalankan Jenkins di host, yang semula dari 8080 ke 49000.
3. Alih-alih menggunakan declarative pipeline untuk sintaks Jenkinsfile, Anda menggunakan scripted pipeline.
4. Menerapkan Poll SCM untuk Build Triggers sehingga Jenkins akan terus memeriksa git repository setiap 2 menit sekali.

Kriteria tambahan yang tidak saya kerjakan:
1. Mengerjakan submission menggunakan source code aplikasi yang berbeda (pastikan untuk mencantumkan screenshot halaman Jenkins pipeline pada submission). Anda bisa pakai salah satu proyek dari opsi berikut.

## simple-node-js-react-npm-app

This repository is for the
[Build a Node.js and React app with npm](https://jenkins.io/doc/tutorials/build-a-node-js-and-react-app-with-npm/)
tutorial in the [Jenkins User Documentation](https://jenkins.io/doc/).

The repository contains a simple Node.js and React application which generates
a web page with the content "Welcome to React" and is accompanied by a test to
check that the application renders satisfactorily.

The `jenkins` directory contains an example of the `Jenkinsfile` (i.e. Pipeline)
you'll be creating yourself during the tutorial and the `scripts` subdirectory
contains shell scripts with commands that are executed when Jenkins processes
the "Test" and "Deliver" stages of your Pipeline.


## Menjalankan jenkins dan blue ocean
1. Buatkan network di docker
    ```
    docker network create jenkins
    ```

2. Menjalankan docker:dind
    ```
    docker run --name jenkins-docker --detach ^
    --privileged --network jenkins --network-alias docker ^
    --env DOCKER_TLS_CERTDIR=/certs ^
    --volume jenkins-docker-certs:/certs/client ^
    --volume jenkins-data:/var/jenkins_home ^
    --restart always ^
    --publish 3000:3000 --publish 5000:5000 --publish 2376:2376 ^
    docker:dind
    ```

3. Buat sebuah berkas bernama **Dockerfile**

    ```
    FROM jenkins/jenkins:2.426.3-jdk17
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
    RUN jenkins-plugin-cli --plugins "blueocean:1.27.10 docker-workflow:572.v950f58993843"
    ```

4. Buat sebuah docker image
    ```
    docker build -t myjenkins-blueocean:2.426.2-1 .
    ```

5. Jalankan container jenkins 
    ```
    docker run --name jenkins-blueocean --detach ^
    --network jenkins --env DOCKER_HOST=tcp://docker:2376 ^
    --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 ^
    --volume jenkins-data:/var/jenkins_home ^
    --volume jenkins-docker-certs:/certs/client:ro ^
    --volume "%HOMEDRIVE%%HOMEPATH%":/home ^
    --restart=on-failure ^
    --env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" ^
    --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:2.426.2-1
    ```

6. Buka browser Anda dan jalankan http://localhost:8080. Tunggu hingga halaman Unlock Jenkins muncul.

7. Tampilkan jenkins console log
    ```
    docker logs jenkins-blueocean
    ```

8. Dari aplikasi Terminal/CMD, salin password yang ada di antara 2 rangkaian asterisk

9. Kembali ke halaman Unlock Jenkins di browser, paste password tersebut ke kolom Administrator password dan klik Continue.

10. Setelah itu, halaman Customize Jenkins muncul. Pilih Install suggested plugins.

11. Saat halaman Create First Admin User muncul, isilah sesuai keinginan Anda dan klik Save and Continue.

12. Pada halaman Instance Configuration, pilih Save and Finish.

13. Saat halaman Jenkins is ready muncul, klik tombol Start using Jenkins.
