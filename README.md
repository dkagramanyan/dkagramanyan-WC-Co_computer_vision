### Расположение снимков
Расположение исходных снимков и предобработанных снимков должно выглядеть следующим образом

```
project
│
└───images_folder
   │
   └───class1_images
   │       image1
   │       image2
   │       ...
   └───class2_images
   │       image1
   │       image2
   │       ...
   └───class3_images
   │       image1
   │       image2
   │       ...

```

### Предобработка изображений

Для явного выделения границ фаз WC/Co используется последовательное применение следующиих алгоритом

1) выбирается сторона снимка. По умолчанию использвуется снимок в отраженных электронах

2) изображение сглаживается медианным фильтром для подавления шумов и выравнивания 
цветового распределения пикселей

3) слаженное изображение бинаризуется при помощи метода Отсу

4) от бинаризованного изображения берется градиент. Он явно показывает переходы вас WC/Co

5) бинаризованное изобржение инвертируется и к нему добавляется карта градиентов, полученная в п.4

Полная обработка изображения выглядит следующим образом:

        preproc_image=1-Otsu(median_filter(image))+grad(Otsu(median_filter(image)))


Значения пикселей по классам:

* 0 - зерно WC

* 1 - регион Co

* 2 - граница региона Co, смежного с зернами WC. Толщина границы - 1 пиксель

Обработка одного снимка

        img=io.imread(img_path)
        img=grainPreprocess.image_preprocess(img,h,k)

Обработка всего датасета снимков 

        all_images=grainPreprocess.read_preprocess_data(images_folder_path,
                                                        images_num_per_class=150,
                                                        preprocess=True,
                                                        save=True)

### Определение углов у регионов Co

Для определения углов последовательно используется 3 инстурмента:

1) поиск границ Кенни. Он находит на изображении все возможные границы и затем выделяет их
        
        
2) метод поиска конутров Suzuki. Он нужен для разметки (нумерования) пикселей найденных в п.1 контуров. 
Направление обхода контура - против часовой стрелки. Проверим направление. Точка - начало отрисовки контура
![sizuki](plots/suzuki.jpg)

        grainMark.get_row_contours(image)

3) полученные масивы пикселей контуров аппроксимируются методом Дугласа-Пекера. Он из массива точек оставляет только те
точки, которые описывают характер полигона, например: точки в углах, перегибах и тд

В итоге для каждого контура мы имеем массив точек, которые с заданной точностью описывают периметр контура. 

Для определения границы воспользуемся определителем тройкой векторов. Если определитель det меньше 0,
то угол phi меньше  180 градусов, иначе больше 180.

### Вписывание регионов Co в эллипс

Задача вписывания фигуры в эллипс называется minimal volume enclosing ellipsoid. Параметры эллипсоида подбираются так, 
чтобы заданные точки находились внутри фигуры, а ее геометрические характеристики были наименьшими.

Для решения задачи используется алгоритм Хачуяна из библиотеки radio-beam.

![ellipsoid](/plots/enclosed-ellipse.png "miv vall ellipsoid")

### Машинное обучение

Для использования снимков нужно создать модель автоэнкодер, которая из снимков будет вытягивать вектор признаков.
 Затем этот вектор вместе с вычисленными характеристиками будет отправлться на вход другой нейронной сети, 
 которая будет искать зависимости между входными значениями и физическими характеристиками сплавов
 
 Архитектура:
 
 ![pipeline](https://drive.google.com/file/d/14-KxRV8ZdljLHcCqkZpT_N7x3Or1ZB8B/view?usp=sharing)
 
 ### Генерация снимков на Blender
 Для генерации снимков на Windows необходимо установить Blender и загрузить файл GrainGenerator_pobedit.blend. 
 В разделе Scripting в строке 92 указать путь к папке, в которой необходимо сохранить снимки (всё, что до "'blender'+" и так далее), в том же формате и запустить скрипт.
 Будут созданы три образца, и с каждого будут сделаны три снимка и сохранены по указанному пути.
 Если не сработает, нужно открыть терминал Blender (в левом верхнем углу Window-Toggle System Console), там будут указаны ошибки
 
 Для запуска в Linux предположительно нужно использовать команду:
      blender yourblendfilenameorpath --python drawcar.py
   
При запуске на Харизме вставить части кода, помеченные "для генерации на харизме", и удалить строки, помеченные "для генерации локально"
