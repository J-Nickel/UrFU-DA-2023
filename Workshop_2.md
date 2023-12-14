# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил:
- Зефиров Никита Викторович
- РИ220946

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

Структура отчета

- Данные о работе: название работы, , группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.

## Цель работы
Научиться передавать в Unity данные из Google Sheets с помощью Python.

## Задание 1
### Выбрать одну из компьютерных игр, приведите скриншот её геймплея и краткое описание концепта игры.

Terraria - 2D sandbox-игра с открытым миром, исследованием, строительством и битвами. Игроки добывают ресурсы, сражаются с врагами, создают предметы, строят базы и прокачивают персонажей в случайно генерируемом мире с пиксельной графикой.

![[Terraria]](WS_2/terraria.jpg)

В Terraria здоровье (HP) является одним из ключевых показателей, который определяет выживаемость и изменение игровых механик.
Максимальное значение HP в Terraria может быть изменено в ходе игры как временно, так и перманентно.
Диапазон значений HP:
- Начало игры: от 0 до 100 
- Середина игры: от 0 до 400
- Завершающий этап игры от 0 до 500

> На каждом этапе к максимальному значению может быть добавлено до 200 ед. по средствам временных баффов.

Роль здоровья:
- HP определяет общее количество повреждений, которое игрок может выдержать, прежде чем погибнет.
- Если здоровье игрока достигает нуля, он либо заканчивает игру, либо возраждается с потерей ресурсов. (зависит от режима игры)
- Здоровье влияет на тактику ведения боя и исследования мира.

![[HP]](WS_2/Terraria_HP.png)

## Задание 2
### Генерация данных с помощью Python и их передача в google таблицу

Ход работы:
- Запустить Jupyter Notebook, создать новый .ipynb-файл.
- Добавить в директорию с новым .ipynb-файлом файл с ключом в формате json от сервисного аккаунта.
- В файле на языке Python реализовать передачу данных в google-таблицу. Реализация на Python рассчитывает темп инфляции для экономической модели:
```python
import gspread
import numpy as np
gc = gspread.service_account(filename='urfu-da-zefirovnv-d2bbef6049f6.json')
sh = gc.open("Workshop_2")

HP_value = 100
line_pos = 2

sh.sheet1.update(f'A{1}', f'Default HP')
sh.sheet1.update(f'B{1}', str(HP_value))
for i in range(15):
    HP_value += 20
    sh.sheet1.update(f'A{str(line_pos)}', f'Red_Heart_{str(i + 1)}')
    sh.sheet1.update(f'B{str(line_pos)}', str(HP_value))
    line_pos += 1
for i in range(5):
    HP_value += 20
    sh.sheet1.update(f'A{str(line_pos)}', f'LifeFruit_{str(i + 1)}')
    sh.sheet1.update(f'B{str(line_pos)}', str(HP_value))
    line_pos += 1

```

> Результат записи данных в таблицу

![[Data]](WS_2/WS_2_Table.PNG)

>"A" - Статус изменения
>"B" - Значение после изменения максимального значения HP

![[Diagram]](WS_2/WS_2_Diagram.PNG)

## Задание 3
### Настройка воспроизведения звуковых файлов, описывающих динамику изменения выбранной переменной на сцене Unity.


```c#
using System;
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
    private Dictionary<string,float> dataSet = new();
    private List<string> keys = new();
    private int i;
    private bool statusStart;

    private void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    private void Update()
    {
        if (statusStart || i == dataSet.Count) return;
        switch (dataSet[keys[i]])
        {
            case 100:
                StartCoroutine(PlayAudio(badSpeak, 4));
                break;
            case <= 400:
                StartCoroutine(PlayAudio(normalSpeak, 4));
                break;
            default:
                StartCoroutine(PlayAudio(goodSpeak, 4));
                break;
        }
        Debug.Log(dataSet[keys[i]]);
    }

    private IEnumerator GoogleSheets()
    {
        var currentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/16f8OohIkUgQvtUzEVgAQDXhFYrAKaM7IFyo5jx_AO2o/values/Лист1?key=AIzaSyBSHIxHS3qg-RwK4XBo1VwmU9ZPBStFCW4");
        yield return currentResp.SendWebRequest();
        var rawResp = currentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            Debug.Log(selectRow.ToString());
            dataSet.Add(selectRow[0], float.Parse(selectRow[1]));
            keys.Add(selectRow[0]);
        }
    }

    private IEnumerator PlayAudio(AudioClip clip, float delay)
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = clip;
        selectAudio.Play();
        yield return new WaitForSeconds(delay);
        statusStart = false;
        i++;
    }
}
```

## Выводы

- В процессе работы был опробован метод передачи данных между Google Sheets и Unity, а также визуализация этих данных. 
- Удалось заполнить данные в Google таблице с использованием Python скрипта.
- Удалось переписать и выполнить C# скрипт в Unity взаимодайствующий с данными из Google таблицы.

|Plugin|README|
|---|---|
|Dropbox|[plugins/dropbox/README.md][PlDb]|
|GitHub|[plugins/github/README.md][PlGh]|
|Google Drive|[plugins/googledrive/README.md][PlGd]|
|OneDrive|[plugins/onedrive/README.md][PlOd]|
|Medium|[plugins/medium/README.md][PlMe]|
|Google Analytics|[plugins/googleanalytics/README.md][PlGa]| 
