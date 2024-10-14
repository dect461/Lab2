# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил:
- Шашурин Алексей Иванович
- РИ230914
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)


## Цель работы
Научиться передавать в Unity данные из Google Sheets с помощью Python.

## Задание 1
### Выберите одну из игровых переменных в игре СПАСТИ РТФ: Выживание (HP, SP, игровая валюта, здоровье и т.д.), опишите её роль в игре, условия изменения / появления и диапазон допустимых значений. Постройте схему экономической модели в игре и укажите место выбранного ресурса в ней.

Ход работы:
- Мой выбор пал на такую переменную из игры СПАСТИ РТФ как количество зомби в каждой новой волне. В начале новой волны будет произноситься слова, подсказывающие увеличилось количество зомби сильно, средне и слабо. Пределы добавочного количества зомби к старой волне варьируются каждый раз с 10 до 100, начало игры стартует с 10 зомби. Если изменилось с 10 до 30 произнесется звук "Волна слегка увеличилась", с 31 до 50 - "Волна умеренно увеличилась", с 51 до 100 "Волна сильно увеличилась". 
![shema](Shema.png)


## Задание 2
### С помощью скрипта на языке Python заполните google-таблицу данными, описывающими выбранную игровую переменную в игре “СПАСТИ РТФ:Выживание”. Средствами google-sheets визуализируйте данные в google-таблице (постройте график / диаграмму и пр.) для наглядного представления выбранной игровой величины. Опишите характер изменения этой величины, опишите недостатки в реализации этой величины (например, в игре может произойти условие наступления эксплойта) и предложите до 3-х вариантов модификации условий работы с переменной, чтобы сделать игровой опыт лучше.

- Ссылка на гугл таблицу https://docs.google.com/spreadsheets/d/1M0mgnJnXmGClC_21qhuVUFNaMYJyvuEvUXW2tp6ewLI/edit?gid=0#gid=0
- Первая диаграмма показывает рост количества зомби с каждой волной
![Diag1](Diag1.png)
- Вторая диаграмма показывает какие случайные значение были добавлены каждой новой волне
![Diag2](Diag2.png)
Предлагаемые модификации:
-Сделать рост добавочного количества зомби постепенным. Например в начале от 5-10, на 5 волне 10-25 и т.д., чтобы в начале не было прибавки 100 зомби и в следующей волне так же, что сделает победу почти нереальной на первых этапах.
-Сделать вероятность волны без увеличения количества зомби
-Сделать вероятность того чтобы в новой волне количество зомби уменьшалось
```py
import gspread
import numpy as np
gc = gspread.service_account(filename='unitydatascience-400014-da50a8398902.json')
sh = gc.open("UnitySheets")
zombies = np.random.randint(10, 100, 11)
mon = list(range(1,11))
start = 10
i = 0
while i <= len(mon):
    i += 1
    if i == 0:
        continue
    else:
        start = (int(start) + zombies[i-1])
        start = str(start)
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(start))
        sh.sheet1.update(('C' + str(i)), str(zombies[i-1]))
        print(zombiesProsent)

```

## Задание 3
### Настройте на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной. Например, если выбрано здоровье главного персонажа вы можете выводить сообщения, связанные с его состоянием.

- Для описания я изменил звуковые файлы, которые теперь говорят какая волна наступает.


```C#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string, float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet["Mon_" + i.ToString()] <= 30 & statusStart == false & i != dataSet.Count + 1)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 31 & dataSet["Mon_" + i.ToString()] < 50 & statusStart == false & i != dataSet.Count + 1)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 51 & statusStart == false & i != dataSet.Count + 1)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1M0mgnJnXmGClC_21qhuVUFNaMYJyvuEvUXW2tp6ewLI/values/Лист1?key=AIzaSyASnu9QXtSW-cHLv__q0mYZNbbhJUFUgdM");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}

```

## Выводы

Из этой лабалоторной работы я научился передавать в Unity данные из Google Sheets с помощью Python и C#. 

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
