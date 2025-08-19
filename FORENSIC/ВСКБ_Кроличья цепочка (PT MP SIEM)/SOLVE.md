### Переходим в PT MP SIEM и выставляем время инцидента согласно заданию

![](./screens/screen1.PNG)

### Далее необходимо выставить фильтр также исходя из задания (хост и id процесса)

```
event_src.host = "itwrksn08.innorange.ru" and object.process.id = "13916"
```

![](./screens/screen2.PNG)


