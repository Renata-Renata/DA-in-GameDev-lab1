# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
# 5. Лабораторная работа #5 ML-Agent в экономической системе

Отчет по лабораторной работе #5 выполнил(а):
- Каримова Рената Ильясовна
- НМТ-213511 
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

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.

## Цель работы


## Постановка задачи.


## Задание 1. 
###  Измените параметры файла yaml-агента и определить, какие параметры и как влияют на обучение модели.

Возьмем за первичные входные данные результаты проекта преподавателя с наименовнием EconomicX.

Этап 1. "Эксперимент" , 
Идея данного этапа заключается в том, чтобы, грубо говоря, сломать всё и посмотреть, что произойдет и как повлияет на наш ход работы.
Получим такую конфигурацию:
```
behaviors:
  Economic:
    trainer_type: ppo
    hyperparameters:
      batch_size: 1024
      buffer_size: 10240
      learning_rate: 3.0e-4
      learning_rate_schedule: linear
      beta: 1.0e-2
      epsilon: 0.3
      lambd: 0.92
      num_epoch: 7      
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    checkpoint_interval: 500000
    max_steps: 750000
    time_horizon: 64
    summary_freq: 5000
    self_play:
      save_steps: 20000
      team_change: 100000
      swap_steps: 10000
      play_against_latest_model_ratio: 0.5
      window: 10
```

![image](https://user-images.githubusercontent.com/90757310/205037014-3d2997ff-8b54-4c00-8d6d-43a47a860826.png)


- Важно отметить, что параметры всё же были изменены не совсем бессознательно, так как нужно знать, что вообще можно поменять, а что нет. Но никакой общей идеи в этих изменениях не было. Проведем обучение до 25 тыс. шагов или до двух отчетов о полученном вознаграждении.

Результат:

![image](https://user-images.githubusercontent.com/90757310/205038880-86b8951f-2cb4-480a-bb2b-6decd162fdfa.png)

![image](https://user-images.githubusercontent.com/90757310/205039068-f397623a-6d05-4f2a-9b39-05ada64a4dee.png)

Итог: сломалась кривая хаотичности системы и это отразилась на результате. 

Теперь попробуем поставить идею повышения скорости обучаемости модели. Вернемся в параметры, схожие со стандартными, но с уклоном в более быстрый процесс обучения. Для этого ознакомимся с несколькими интересными параметрами тренера.

| Параметр | Описание |
| ------ | ------ |
| `beta` |  Сила регуляризации энтропии, которая делает политику «более случайной». Это гарантирует, что агенты должным образом исследуют пространство действия во время обучения. Увеличение этого параметра обеспечит выполнение большего количества случайных действий. Это должно быть скорректировано таким образом, чтобы энтропия (измеряемая с помощью TensorBoard) медленно уменьшалась вместе с увеличением вознаграждения. Если энтропия падает слишком быстро, увеличьте бета. Если энтропия падает слишком медленно, уменьшите beta. |
| `epsilon` | Влияет на скорость изменения политики во время обучения. Соответствует допустимому порогу расхождения между старой и новой политикой при обновлении градиентного спуска. Установка небольшого значения этого параметра приведет к более стабильным обновлениям, но также замедлит процесс обучения. |
| `lambd` | Параметр регуляризации лямбда, используемый при расчете обобщенной оценки преимущества. Это можно рассматривать как то, насколько агент полагается на свою текущую оценку стоимости при вычислении обновленной оценки стоимости. Низкие значения соответствуют большему полаганию на текущую оценку ценности (что может быть высоким смещением), а высокие значения соответствуют большему полаганию на фактические вознаграждения, полученные в среде (что может быть высокой дисперсией). Параметр обеспечивает компромисс между ними, и правильное значение может привести к более стабильному процессу обучения. |
| `num_epoch` |  Количество проходов через буфер опыта при выполнении оптимизации градиентного спуска. Чем больше размер партии, тем больше это допустимо. Уменьшение этого параметра обеспечит более стабильные обновления за счет более медленного обучения. |

Вот так будет выглядеть новая конфигурация тренера: 

```
behaviors:
  Economic:
    trainer_type: ppo
    hyperparameters:
      batch_size: 1024
      buffer_size: 10240
      learning_rate: 3.0e-4
      learning_rate_schedule: linear
      beta: 1.0e-3
      epsilon: 0.1
      lambd: 0.94
      num_epoch: 5      
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    checkpoint_interval: 500000
    max_steps: 750000
    time_horizon: 64
    summary_freq: 5000
    self_play:
      save_steps: 20000
      team_change: 100000
      swap_steps: 10000
      play_against_latest_model_ratio: 0.5
      window: 10
```

![image](https://user-images.githubusercontent.com/90757310/205040699-895d54f2-832e-4427-902b-1e5eb4571f54.png)

А вот и результаты: 

![image](https://user-images.githubusercontent.com/90757310/205041584-02b8a6ca-aaf7-45b9-aa2d-54c78e96ffb8.png)

![image](https://user-images.githubusercontent.com/90757310/205042316-ef4c9810-6bcd-4a1d-9da5-c2fa76562026.png)


Сразу же можно отметить более предсказуемый результат rewards и entropy, котроя даже стала снижаться на позднем этапе обучения, когда система стала дорабатывать имеющиеся результаты до идеального состояния.

## Задание 2 Анализируем графики

![image](https://user-images.githubusercontent.com/90757310/205043698-61b3dc04-a69f-421e-9a15-1c5c3a3a5cf4.png)

На диаграмме мы видим зависимость вознаграждение от совершенных действий. 

` Синяя прямая ` - вариант преподавателя. Система совершала действия, положительно влияющие на обучении все время и получала одно вознаграждение. 

` Красная ломаная ` - первый и самый хаотичный эксперимент. Плюс: система обучалась более быстро. Минус: в какой-то момент хаотичность стала мешать и вознаграждение упало.

 ` Голубая ломаная ` - второй эксперимент, который должен был бы быть оптимальным на фоне предыдущего. И действительно: награда растет, а значит система развивается, однако в самом начале количество вознаграждения не велико. 
 
Остальные ломанные - это промежуточные результаты и никакой смысловой нагрузки в них нет. 

- Идеально было бы балансировать параметры тренера таким образом, чтобы, изменяя их отдельно и поочередно, мы добивались наилучшего результата. 

## Выводы
В ходе лабораторной работы мы познакомились с инструментом tensorflow - графическим отображением поведения нейросети во время обучения, открытая программной библиотекой для машинного обучения.
Также мы повторно обратились к документации `ml-agent 19` для решения второго этапа нашей задачи. Это закрепило навыки, полученные при исполнении лбораторной работы 3 и создании RollerAgent.
Для достижения идеального результата необходимо балансировать параметры тренера отдельно и поочередно, чтобы после шага назад мы могли вернуться к предыдущему этапу и пересмотреть стартегию дальнейшего обучения, отображать поведения сети мы можем с помощью tensorflow.


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
