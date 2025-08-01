### В архиве получаем 2 файла:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task]
└─# ls -la                                                                   
total 52
drwxrwxrwx 1 root root     0 Mar  5 16:47 .
drwxrwxrwx 1 root root     0 Jul 27 14:36 ..
-rwxrwxrwx 1 root root 27152 Mar  5 15:36 dump
-rwxrwxrwx 1 root root 22321 Mar  5 16:32 SecretPrograms.pdf
```

### Проверяем тип файлов:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task]
└─# file *                                                                   
dump:               pcapng capture file - version 1.0
SecretPrograms.pdf: Zip archive data, at least v2.0 to extract, compression method=store
```

### Меняем названия для удобства:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task]
└─# mv dump dump.pcapng

┌──(root㉿scrock)-[~/mnt/task (17)/task]
└─# mv SecretPrograms.pdf SecretPrograms.zip

┌──(root㉿scrock)-[~/mnt/task (17)/task]
└─# ls                                                                       
dump.pcapng  SecretPrograms.zip
```

### Пробуем разархивировать:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task]
└─# unzip SecretPrograms.zip                                                 
Archive:  SecretPrograms.zip
   creating: SecretPrograms/
[SecretPrograms.zip] SecretPrograms/client password: 
   skipping: SecretPrograms/client   incorrect password
   skipping: SecretPrograms/server   incorrect password
```

### Брутим пароль:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task]
└─# fcrackzip -v -D -u -p /usr/share/wordlists/rockyou.txt SecretPrograms.zip'SecretPrograms/' is not encrypted, skipping
found file 'SecretPrograms/client', (size cp/uc  10885/ 13104, flags 1, chk 3fd3)
found file 'SecretPrograms/server', (size cp/uc  10964/ 13324, flags 1, chk 4804)


PASSWORD FOUND!!!!: pw == caramelo
```

### Разархивируем:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task]
└─# unzip SecretPrograms.zip                                                 
Archive:  SecretPrograms.zip
[SecretPrograms.zip] SecretPrograms/client password: 
  inflating: SecretPrograms/client   
  inflating: SecretPrograms/server  
```

### Смотрим:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task/SecretPrograms]
└─# ls -la                                                                   
total 32
drwxrwxrwx 1 root root     0 Jul 27 14:40 .
drwxrwxrwx 1 root root     0 Jul 27 14:39 ..
-rwxrwxrwx 1 root root 13104 Mar  5 22:58 client
-rwxrwxrwx 1 root root 13324 Mar  5 22:58 server

┌──(root㉿scrock)-[~/mnt/task (17)/task/SecretPrograms]
└─# file *                                                                   
client: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), statically linked, no section header
server: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), statically linked, no section header
```

### Тут 2 исполняемых файла, пробиваем их с помощью strings:

И видим что они запакованы:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task/SecretPrograms]
└─# strings -n10 *   

$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 4.24 Copyright (C) 1996-2024 the UPX Team. All Rights Reserved. $
```

### Гуглим упаковщик и распаковываем:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task/SecretPrograms]
└─# upx -d client                                                            
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2024
UPX 4.2.4       Markus Oberhumer, Laszlo Molnar & John Reiser    May 9th 2024

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
     27175 <-     13104   48.22%   linux/amd64   client

Unpacked 1 file.

┌──(root㉿scrock)-[~/mnt/task (17)/task/SecretPrograms]
└─# upx -d server                                                            
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2024
UPX 4.2.4       Markus Oberhumer, Laszlo Molnar & John Reiser    May 9th 2024

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
     27011 <-     13324   49.33%   linux/amd64   server

Unpacked 1 file.
```

### Снова смотрим strings:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task/SecretPrograms]
└─# strings -n10 *   
```

Находим приватные ключи

### Создаем файл приватного ключа клиента:

```
echo "Найденная base64 строка в файле клиента" | base64 -d > client_private.key
```

Создаем файл приватного ключа сервера:

```
echo "Найденная base64 строка в файле сервера" | base64 -d > server_private.key
```

### Проверяем корректность приватных ключей:

```
┌──(root㉿scrock)-[~/mnt/task (17)/task/SecretPrograms]
└─# openssl rsa -in client_private.key -check -noout
openssl rsa -in server_private.key -check -noout
RSA key ok
RSA key ok
```

Там также еще будут публичные ключи (немного короче base64 строки), но они нам для расшифровки не нужны.

### У меня уже был примерный код поэтому я отредактировал его под данный кейс

```python
from scapy.all import *
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_OAEP

def decrypt_pcap(pcap_file, private_key_path):
    packets = rdpcap(pcap_file)
    key = RSA.import_key(open(private_key_path).read())
    cipher = PKCS1_OAEP.new(key)
    
    for pkt in packets:
        if TCP in pkt and pkt[TCP].payload:
            payload = bytes(pkt[TCP].payload)
            try:
                decrypted = cipher.decrypt(payload)
                print(f"Decrypted: {decrypted.decode()}")
            except:
                continue

decrypt_pcap(r"C:\Users\scrock_\Desktop\a\iso\ctf\EHAX++\task (17)\task\dump.pcapng", r"C:\Users\scrock_\Desktop\a\iso\ctf\EHAX++\task (17)\task\SecretPrograms\server_private.key")
```

### Обратите внимание, что здесь не полный диалог, расшифрована только одна сторона, диалог:

(.venv) PS C:\Users\scrock_\Desktop\a\LABS\SEMESTR 9\BD> .\.venv\Scripts\python.exe .\ОТЧЕТЫ\ctf.py
Decrypted: привет привет
Decrypted: Как там? все в силе?
Decrypted: А диски с Невского?
Decrypted: Отлично
Decrypted: смотри
Decrypted: Володарский мост, вкуснор и точка
Decrypted: Володарский мост, вкусно и точка
Decrypted: только не пались, ладно?
Decrypted: ну ну
Decrypted: чего чего?
Decrypted: всм?
Decrypted: то есть...
Decrypted: фигасе
Decrypted: согласен

### Далее гуглим место:

![](./screens/screen1.PNG)

### Флаг:

```
caplag{НАРОДНАЯ_4_2_САНКТ-ПЕТЕРБУРГ}
```
