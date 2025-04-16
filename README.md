# Laporan Latihan Handson Praktikum 2

### Nama     : Akmal Yusuf
### NRP      : 5025241212
### Kelas    : E
### Kelompok : 14


## 1. Process

```
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdlib.h>

int main(){
    pid_t ch_id1, ch_id2;
    int status;

    ch_id1 = fork();
    char *username = getenv("USER");
    char destination[100];
    sprintf(destination, "/home/%s/Sisop/hand2/halo/halo.txt", username);

    if (ch_id1 < 0) exit(EXIT_FAILURE);
       
    // child 1
    if(ch_id1 == 0){
        char *argv1[] = {"mkdir", "-p", "halo", NULL};
        execv("/usr/bin/mkdir", argv1);
        exit(0);
    } 
    // brach program
    else if (ch_id1 > 0) {
        wait(NULL);
        ch_id2 = fork();

        if (ch_id2 < 0) exit(EXIT_FAILURE);

        // child 2
        if(ch_id2 == 0) {
            char *argv2[] = {"touch", destination, NULL};
            execv("/usr/bin/touch", argv2);
        } 

        // parent      
        else {
            while ((wait(&status)) > 0);
            char *argv3[] = {"cp", destination, ".", NULL};
            execv("/usr/bin/cp", argv3);
        }
        exit(0);
    }

    return 0;
}
```
Untuk menyelesaikan soal ini, cukup membuat 3 program (2 child dan 1 parent), Sehingga fork dilakukan 2 kali namun fork terahir diletakkan di dalam program parent.

Child 1 akan membuat folder halo -> Child 2 akan membuat file halo.txt didalam folder halo -> Parent akan mencopy halo.txt kedalam current folder.

### Bentuk program akan menjadi seperti ini:
![alt text](![Schema](https://github.com/user-attachments/assets/79abaa1c-dd11-4ca7-ac3d-41449ac025e9))

```
    pid_t ch_id1, ch_id2;
    int status;
```

- Menginisiasikan pid dari child 1 dan 2
- Menginisiasikan `status` untuk menyimpan status dari proses yang sedang berjalan

```
    ch_id1 = fork();
    char *username = getenv("USER");
    char destination[100];
    sprintf(destination, "/home/%s/Sisop/hand2/halo/halo.txt", username);
```

- Membuat proses anak pertama
- Mengisi variabel username dengan nama user pengguna
- Menyimpan path menuju halo.txt ke dalam variabel destination untuk memudahkan penulisan

```
    if (ch_id1 < 0) exit(EXIT_FAILURE);
    if (ch_id2 < 0) exit(EXIT_FAILURE);
```
- Error handler

### Child 1
```
    if(ch_id1 == 0){
        char *argv1[] = {"mkdir", "-p", "halo", NULL};
        execv("/usr/bin/mkdir", argv1);
        exit(0);
    } 
```
- membuat folder dengan mkdir dan menamainya halo
- menjalankan command dengan execv

```
    // brach program
    else if (ch_id1 > 0) {
        wait(NULL);
        ch_id2 = fork();
```
- Sebelum membuat child 2, wait proses child pertama
- Kemudian membuat child 2.


### Child 2
```
        // child 2
        if(ch_id2 == 0) {
            char *argv2[] = {"touch", destination, NULL};
            execv("/usr/bin/touch", argv2);
        } 
```

- membuat file halo.txt di destinasi yang sudah ditentukan
- mengeksekusinya menggunakan execv

### Parent Process
```
        // parent      
        else {
            while ((wait(&status)) > 0);
            char *argv3[] = {"cp", destination, ".", NULL};
            execv("/usr/bin/cp", argv3);
        }
```
- Menunggu proses child selesai
- Mencopy menggunakan `cp` dari path destinasi (yang merujuk pada halo.txt) kedalam current folder
- Mengeksekusinya menggunakan execv

### Before run
![alt text]()

### Compilation
![alt text]()

### After run
![alt text]()



## 2. Threads

```
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>

typedef struct arg {
    int end[3];
    int idx;
} arg;

void *routine1(void *args){
    arg *data = (arg *)args;


    FILE *fp = fopen("count.txt", "w");
    for(int i = 1; i <= 100; i++){
        fprintf(fp, "%d\n", i);
    }
    fclose(fp);

    data->end[data->idx++] = 1;
}

void *routine2(void *args){
    arg *data = (arg *)args;

    FILE *fp = fopen("print.txt", "w");
    fprintf(fp, "Saya pintar mengerjakan thread");
    fclose(fp);

    data->end[data->idx++] = 2;
}

void *routine3(void *args){
    arg *data = (arg *)args;

    FILE *fp = fopen("count_2.txt", "w");
    for(int i = 2; i <= 100; i+=2){
        fprintf(fp, "%d\n", i);
    }
    fclose(fp);

    data->end[data->idx++] = 3;
}

int main(){
    pthread_t t[3];
    arg a1;
    a1.idx = 0;

    pthread_create(&t[0], NULL ,&routine1 ,(void *)&a1);
    pthread_create(&t[1], NULL ,&routine2 ,(void *)&a1);
    pthread_create(&t[2], NULL ,&routine3 ,(void *)&a1);

    for(int i = 0; i < 3; i++){
        pthread_join(t[i], NULL);
    }

    FILE *f = fopen("log.txt", "a");
    fprintf(f, "Threads %d\n", a1.end[0]);
    fclose(f);

    return 0;
}
```

- Threads dibuat terpisah di 3 fungsi
- struct sebagai argumen yang di passing kedalam Threads

```
typedef struct arg {
    int end[3];
    int idx;
} arg;
```
- `end[3]` untuk menampung urutan threads yang selesai terlebih dahulu
- `idx` sebagai index yang akan di increment setiap suatu threads selesai

### Threads 1
```
void *routine1(void *args){
    arg *data = (arg *)args;


    FILE *fp = fopen("count.txt", "w");
    for(int i = 1; i <= 100; i++){
        fprintf(fp, "%d\n", i);
    }
    fclose(fp);

    data->end[data->idx++] = 1;
}

```
- Menerima passing argument dari main
- Membuka dan menulis angka dari 1-100 kedalam count.txt
- Memasukkan informasi threads nomor `1` kedalam urutan theads yang selesai duluan

### Threads 2
```
void *routine2(void *args){
    arg *data = (arg *)args;

    FILE *fp = fopen("print.txt", "w");
    fprintf(fp, "Saya pintar mengerjakan thread");
    fclose(fp);

    data->end[data->idx++] = 2;
}
```

- Threads kedua untuk mengeprint kata `Saya pintar mengerjakan thread` kedalam print.txt
- Memasukkan informasi threads nomor `2` kedalam urutan theads yang selesai duluan

### Threads 3
```
void *routine3(void *args){
    arg *data = (arg *)args;

    FILE *fp = fopen("count_2.txt", "w");
    for(int i = 2; i <= 100; i+=2){
        fprintf(fp, "%d\n", i);
    }
    fclose(fp);

    data->end[data->idx++] = 3;
}
```
- Fungsi untuk menjalankan threads ketiga
- Menulis angka genap kedalam count_2.txt
- Memasukkan informasi threads nomor `3` kedalam urutan theads yang selesai duluan

```
    pthread_t t[3];
    arg a1;
    a1.idx = 0;
```
- Membuat 3 threads
- Inisiasi struct

```
    pthread_create(&t[0], NULL ,&routine1 ,(void *)&a1);
    pthread_create(&t[1], NULL ,&routine2 ,(void *)&a1);
    pthread_create(&t[2], NULL ,&routine3 ,(void *)&a1);
```
- Membuat Threads dan mem-passing struct
- Memanggil function sesuai dengan kegunaanya

```
    for(int i = 0; i < 3; i++){
        pthread_join(t[i], NULL);
    }
```
- Memulai threads
```
    FILE *f = fopen("log.txt", "a");
    fprintf(f, "Threads %d\n", a1.end[0]);
    fclose(f);
```
- Menyimpan threads yang selesai pertama kedalam file log.txt

### Before run
![alt text](https://drive.google.com/file/d/1ED26hmFdX6bB-CUaLFDvfsByV5eeJomr/view?usp=drive_link)

### Compilation
![alt text](https://drive.google.com/file/d/1oQ_TcjTYH45UA0HZ1H8bTgDTqLVm8MMu/view?usp=drive_link)

### After run
![alt text](https://drive.google.com/file/d/13FPuK2EvZYbWL2yjgNCsMqSVhl1GuHB5/view?usp=drive_link)

### Output
![alt text](https://drive.google.com/file/d/12DZgJdPzidr-LktoKJKgliiIzRGbGbIs/view?usp=sharing)

## 3. Threads Mutex
```
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdlib.h>
#include <pthread.h>

pthread_mutex_t lock; 

void *routine(void *argv){
    int data = *((int *)argv);
    FILE *f = fopen("log.txt", "a");

        if(data == 0){
            pthread_mutex_lock(&lock);
            for(int j = 1; j <= 3; j++) fprintf(f, "thread 1 count %d\n", j);
            pthread_mutex_unlock(&lock);
        } else if (data == 1){
            pthread_mutex_lock(&lock);
            for(int j = 1; j <= 3; j++) fprintf(f, "thread 2 count %d\n", j);
            pthread_mutex_unlock(&lock);
        } else if (data == 2) {
            pthread_mutex_lock(&lock);
            for(int j = 1; j <= 3; j++) fprintf(f, "thread 3 count %d\n", j);
            pthread_mutex_unlock(&lock);
        } else if (data == 3){
            pthread_mutex_lock(&lock);
            for(int j = 1; j <= 3; j++) fprintf(f, "thread 4 count %d\n", j);
            pthread_mutex_unlock(&lock);
        } else {
            pthread_mutex_lock(&lock);
            for(int j = 1; j <= 3; j++) fprintf(f, "thread 5 count %d\n", j);
            pthread_mutex_unlock(&lock);
        }

    fclose(f);
}

int main(){
    pthread_t t[5];
    int type[5] = {0,1,2,3,4};
    
    for(int i = 0; i < 5; i++){
        pthread_create(&t[i], NULL, &routine, (void*)&type[i]);
    }

    for(int i = 0; i < 5; i++){
        pthread_join(t[i], NULL);
    }

    return 0;
}
```

- Jika sebelumnya menggunakan 3 fungsi yang berbeda untuk menjalankan 3 threads, Kali ini saya menggunakan pendekatan berbeda, yaitu dengan menjalankan fungsi sesuai dengan id yang berbeda.

```
pthread_mutex_t lock;
```
- inisiasi kunci untuk mutex

```
void *routine(void *argv){
    int data = *((int *)argv);
    FILE *f = fopen("log.txt", "a");
```
- function untuk threads
- menerima passing argumen berupa array int
- membuka file log.txt

```
            pthread_mutex_lock(&lock);
            for(int j = 1; j <= 3; j++) fprintf(f, "thread 1 count %d\n", j);
            pthread_mutex_unlock(&lock);

        fclose(f);
```

- Bagian dalam threads memiliki kode yang sama yaitu menghitung angka dari 1-3, hanya berbeda di angka thread
- `pthread_mutex_lock(&lock)` memulai kunci program
- `pthread_mutex_unlock(&lock)` membuka kunci
- Terakhir menutup file

```
    pthread_t t[5];
    int type[5] = {0,1,2,3,4};
```
- inisiasi banyak threads dan tipe threads untuk di lempar kedalam fungsi routine

```
    for(int i = 0; i < 5; i++){
        pthread_create(&t[i], NULL, &routine, (void*)&type[i]);
    }

    for(int i = 0; i < 5; i++){
        pthread_join(t[i], NULL);
    }
```

- Membuat thread dan menjalankanya

### Before run
![alt text]()

### Compilation
![alt text]()

### After run
![alt text]()

### Output
![alt text]()

## 4. IPC
## 4a Shared Memory

### File Sender 
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/ipc.h>
#include <sys/shm.h>

int main(){
    key_t key = 1234;
    int shmid = shmget(key, 200, IPC_CREAT | 0666);

    char *str = (char *) shmat(shmid, NULL, 0);
    strcpy(str, "aku lagi belajar ipc\n");
    shmdt(str);
}
```
Pada file sender, program akan fokus untuk mengirimkan string

```
    key_t key = 1234;
    int shmid = shmget(key, 200, IPC_CREAT | 0666);
```
- `key_t key` menginisiasi kunci untuk shared memory
-  `int shmid` untuk menampung id share memory yang mereserve 200 byte dengan file permission 0666 (read & write untuk user, group, dan others)

```
    char *str = (char *) shmat(shmid, NULL, 0);
    strcpy(str, "aku lagi belajar ipc\n");
    shmdt(str);
```
- `str` menunjuk pada id di shared memory
- Mengcopy "aku lagi belajar pc" kedalam `str`
- detach `str`


### File Receiver
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/ipc.h>
#include <sys/shm.h>

int main(){
    key_t key = 1234;
    int shmid = shmget(key, 200, IPC_CREAT | 0666);
    char *str = (char *) shmat(shmid, NULL, 0);

    printf("%s", str);

    shmdt(str);
    shmctl(shmid, IPC_RMID, NULL);
}
```
Pada file receiver fokus pada menerima pesan dari file sender dan mencetak hasilnya
```
    key_t key = 1234;
    int shmid = shmget(key, 200, IPC_CREAT | 0666);
    char *str = (char *) shmat(shmid, NULL, 0);
```
- key untuk menyimpan key unik
- `str` menunjuk pada id di shared memory
- Mengcopy "aku lagi belajar pc" kedalam `str`
- detach `str`

### Output
![alt text](https://drive.google.com/file/d/1P2r0JVZZbUqkRiTQE2eiEizKaE0PYGL9/view?usp=sharing)


## 4b Message Queue
### File Sender
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/ipc.h>
#include <sys/msg.h>

typedef struct mesg_buffer {
	long mesg_type;
	char mesg_text[100];
} message;


int main(){
	key_t key;
	int msgid;
	message ms;

	key = ftok("progfile", 69);
	msgid = msgget(key, 0666 | IPC_CREAT);
	ms.mesg_type = 1;

	strcpy(ms.mesg_text, "yah belajar ipc mulu\n");

	msgsnd(msgid, &ms, sizeof(message), 0);
	
	return 0;
}
```
Mengirim text dari sender ke dalam receiver dengan queue

```
typedef struct mesg_buffer {
	long mesg_type;
	char mesg_text[100];
} message;
```
- struct untuk menyimpan variabel dan message yang ingin dikirim

```
	key_t key;
	int msgid;
	message ms;
```
- menginisiasi variabel key
- menginisiasi msgid untuk menampung id message
- menginisasi struct

```
	key = ftok("progfile", 69);
	msgid = msgget(key, 0666 | IPC_CREAT);
	ms.mesg_type = 1;
```
- ftok untuk membuat kunci yang lebih unik
- msgget untuk membuat id / menggambil id dari message queue yang dimasukkan kedalam msgid
- Message type digunakan sebagai tag untuk mengelompokkan pesan, yang disini saya isi dengan 1

```
	strcpy(ms.mesg_text, "yah belajar ipc mulu\n");
```
- memasukkan message kedalam variabel

```
	msgsnd(msgid, &ms, sizeof(message), 0);
```
- mengirim kedalam queue

### File receiver
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/ipc.h>
#include <sys/msg.h>

typedef struct mesg_buffer {
	long mesg_type;
	char mesg_text[100];
} message;


int main(){
	key_t key;
	int msgid;
	message ms;

	key = ftok("progfile", 69);
	msgid = msgget(key, 0666 | IPC_CREAT);

	msgrcv(msgid, &ms, sizeof(message), 1, 0);

	printf("%s",ms.mesg_text);

	msgctl(msgid, IPC_RMID, NULL);

	return 0;
}
```
memiliki struktur yang sama dengan sender.


## 4c Pipe With Fork




