# Disease classification on real-world data

## Statement of work

I base on classical image classification pipeline

---
1. Input image
    * Visualization
    * Categorization
2. Data Preprocessing
    * Standartization
    * Augmentation (if needed)
3. Features extracion
4. Classification model
    * Choosing 
        * network arhitecture 
        * loss functions
        * evaluation metrics
    * Train, vlidation, test
6. Testing the results with statistical methods 
---

## Week 1. Tasks to do

- [x] Writing technical requirements for the internship
- [x] Get familiar with working data
- [x] Read about histogram matching


# Report (in progress)

## Content

```
1. Problem statement
2. Working data
    a. ChestX-ray 
    b. CheXpert
3. Methodology adopted
4. Results
5. References
```

## Working data

There are two datasets to deal with chest x-ray images:
* ChestX-ray
* CheXpert

Eight visual examples of common thorax diseases

<img src="https://i.imgur.com/5YEXAxG.png" width="600">

### ChestX-ray dataset 

ChestX-ray dataset comprises 112,120 frontal-view X-ray images of 30,805 unique patients from with the text-mined fourteen disease image labels:

*  Atelectasis
*  Cardiomegaly 
*  Consolidation 
*  Edema
*  Effusion 
*  Emphysema
*  Fibrosis 
*  Hernia
*  Infiltration
*  Nodule 
*  Mass 
*  Pleural_thickening
*  Pneumonia
*  Pneumothorax 

The dataset were presented by the National Institutes of Health Clinical Center with the next material:

* 112.120 frontal-view chest X-ray PNG images in 1024x1024 resolution
* Meta data for all images (Data_Entry_2017.csv): 
    * Image Index 
    * Finding Labels 
    * Follow-up number
    * Patient ID
    * Patient Age
    * Patient Gender
    * View Position
    * Original Image
    * Pixel Spacing
* Bounding boxes for ~1000 images (BBox_List_2017.csv): 
    * Image Index, Finding Label,
    * Bbox [x, y, w, h] 
        * [x y] are coordinates of each box's topleft corner. 
        * [w h] represent the width and height of each box.
* Two data split files. All studies from the same patient will only appear in either training/validation or testing set:
    * train_val_list.txt 
    * test_list.txt 

### CheXpert dataset

CheXpert is a large public dataset for chest radiograph interpretation, consisting of 224,316 chest radiographs of 65,240 patients with fourteen labels:

* Atelectasis
* Cardiomegaly
* Consolidation
* Edema
* Enlarged Cardiomyopathy
* Fracture
* Lung Lesion
* Lung Opacity
* No Finding
* Pleural Effusion
* Pleural Other
* Pneumonia
* Pneumothorax
* Support Devices

The dataset were presented by Stanford Hospital with the next material:

* 224,316 images in 320x320 resolution
* Data split files:
    * train.csv 
    * valid.csv 
    * test.csv

## Methodology adopted

### Histogram matching

Histogram matching or histogram specification is the transformation of an image so that its histogram matches a specified histogram

Example (later there will be a chest x-ray image example)

Source image               | Image to be adjusted
:-------------------------:|:-------------------------:
<img src="https://i.imgur.com/Xk8cokH.png" width="300"> | <img src="https://i.imgur.com/JJDOg0U.png" width="300">


Source image grey levels   | Image to be adjusted grey levels
:-------------------------:|:-------------------------:
<img src="https://i.imgur.com/RtrxLvM.png" width="300"> | <img src="https://i.imgur.com/0aP2XBU.png" width="300">

Source image

| Grey levels | 0 | 1  | 2  | 3 | 4  | 5  | 6 | 7 |
|-------------|---|----|----|---|----|----|---|---|
|Pixels number| 8 | 10 | 10 | 2 | 12 | 16 | 4 | 2 |

| r_k | p_k | Cumulative value | Cumulative value / Total x L_max | Round off to nearest grey |
|-----|-----|------------------|----------------------------------|---------------------------|
| 0   | 8   | 0 + 8 = 8        | 8/64 x 7 = 0.875                 | 1                         |
| 1   | 10  | 8 + 10 = 18      | 18/64 x 7 = 1.968                | 2                         |
| 2   | 10  | 18 + 10 = 28     | 28/64 x 7 = 3.0625               | 3                         |
| 3   | 2   | 28 + 2 = 30      | 30/64 x 7 = 3.2812               | 3                         |
| 4   | 12  | 30 + 12 = 42     | 42/64 x 7 = 4.5937               | 5                         |
| 5   | 16  | 42 + 16 = 58     | 58/64 x 7 = 6.3437        
| 7   | 2   | 62 + 2 = 64      | 64/64 x 7 = 7                    | 7                         |


Image to be adjusted

| Grey levels | 0 | 1 | 2 | 3 | 4  | 5  | 6  | 7 |
|-------------|---|---|---|---|----|----|----|---|
|Pixels number| 0 | 0 | 0 | 0 | 20 | 20 | 16 | 8 |

| r_k | p_k | Cumulative value | Cumullative value / Total x L_max | Round off to nearest grey |
|-----|-----|------------------|-----------------------------------|---------------------------|
| 0   | 0   | 0                | 0                                 | 0                         |
| 1   | 0   | 0                | 0                                 | 0                         |
| 2   | 0   | 0                | 0                                 | 0                         |
| 3   | 0   | 0                | 0                                 | 0                         |
| 4   | 20  | 0 + 20 = 20      | 20/64 x 7 = 2.1875                | 2                         |
| 5   | 20  | 20 + 20 = 40     | 40/64 x 7 = 4.375                 | 4                         |
| 6   | 16  | 40 + 16 = 56     | 56/64 x 7 = 6.125                 | 6                         |
| 7   | 8   | 56 + 8 = 64      | 64/64 x 7 = 7                     | 7                         |

Source image histogram     | Image to be adjusted histogram
:-------------------------:|:-------------------------:
<img src="https://i.imgur.com/b0kmnxa.png" width="300"> | <img src="https://i.imgur.com/HwTia51.png" width="300">

Finally:


| Grey levels | Source image equalization | Image to be  adjusted equalization | Final mapping |
|-------------|---------------------------|------------------------------------|---------------|
| 0 [A]       | 1 - closest is E          | 0 [A]                              | 4             |
| 1 [B]       | 2 - closest is E          | 0 [B]                              | 4             |
| 2 [C]       | 3 - closest is F          | 0 [C]                              | 5             |
| 3 [D]       | 3 - closest is F          | 0 [D]                              | 5             |
| 4 [E]       | 5 - closest is G          | 2 [E]                              | 6             |
| 5 [F]       | 6 - closest is G          | 4 [F]                              | 6             |
| 6 [G]       | 7 - closest is H          | 6 [G]                              | 7             |
| 7 [H]       | 7 - closest is H          | 7 [H]                              | 7             |

Source image               | Matched image
:-------------------------:|:-------------------------:
<img src="https://i.imgur.com/Xk8cokH.png" width="300"> | <img src="https://i.imgur.com/TA1MfX0.png" width="300">

## Results 

## References

