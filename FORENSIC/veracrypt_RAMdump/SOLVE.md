### Расшифровываем подсказку из задания:

Вставляем на https://www.dcode.fr/cipher-identifier

![](./screens/screen1.PNG)

Узнаем, что это base32

### Декодируем base32:

https://cyberchef.org/

![](./screens/screen2.PNG)

"Le mot de passe est le nom de l'école dans laquelle j'ai étudié. Sans espaces, chaque mot avec une lettre majuscule"

### Переводится как:

"Пароль — название школы, в которой я учился. Без пробелов, каждое слово с заглавной буквы."

### Не уверен, что я к концу представил интендед решение, но после проверки дампов всех процессов которые только можно и всего куда можно запрятать, я поступил следующим образом:

1) Закинул дамп в AutoPsy

2) Перевёл "школа" на французский язык - "école"

3) Далее keyword search в AutoPsy по "ecole"

И получаем пароль:

![](./screens/screen3.PNG)

"EcoleMaternellePubliqueRaymondTeisseire"

### Далее монтируем VeraCrypt контейнер (я использовал VeraCrypt 1.26.24 Windows), но для наглядности покажу в терминале:

```
┌──(root㉿scrock)-[~/mnt/crypta]
└─# mkdir /mnt/vera

┌──(root㉿scrock)-[~/mnt/crypta]
└─# veracrypt --mount Crypted /mnt/vera --password="EcoleMaternellePubliqueRaymondTeisseire"
Enter PIM for /root/mnt/crypta/Crypted: 
Enter keyfile [none]: 
Protect hidden volume (if any)? (y=Yes/n=No) [No]: 
```

Прочитаем полученный файл:

```
┌──(root㉿scrock)-[~/mnt/crypta]
└─# cat /mnt/vera/Message.txt                                                
Cher caplag{L4ff41r33sTd4nsL3s4C}!
Nous voulons vous féliciter pour le succès du piratage du système de LolKek. Tu as montré que tu étais un vrai professionnel et tu as prouvé qu'on t'avait choisi pour une bonne raison.
Nous savons que vous n'êtes pas seulement un hacker, mais un vrai maître de votre métier. Vous êtes toujours prêt pour de nouveaux défis et n'avez pas peur des difficultés. Votre courage et votre détermination nous inspirent pour de nouvelles réalisations.
Nous voulons que tu saches que nous apprécions toi et ton travail. Vous êtes une partie importante de notre équipe et nous sommes prêts à vous soutenir dans toute entreprise.
```

### Флаг: 

caplag{L4ff41r33sTd4nsL3s4C}
