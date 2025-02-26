﻿# Contrastive Self-supervised Learning for ship detection in Sentinel 2 images

This work allow to perform ship detection using self-supervised learning (SSL). A Sentinel-2 dataset for ship detection is provided. Check our article for more details

## Environment
### Install requirements
```bash
pip3 install -r requirements.txt
```

### Docker
A docker with a correctly configured environment can be built using the Dockerfile.

## S2-SHIPS Dataset
Our S2-SHIPS dataset can be downloaded here : https://drive.google.com/file/d/1zDgz6wr5kxikPR7o9nJ2IjMcaqwtiLLu/view?usp=sharing. It contains the COCO annotations files, the 12 spectral bands for each S2-SHIPS tile in a tif or numpy array version, the S2-SHIPS segmentation masks and the water masks.

## Usage

### SSL pretext task pretraining
To pretrain a ResNet-50 backbone via SSL on a non labeled dataset having 6 channels, use the following command :

```python
python3 train_pretext.py --run_name test_pretext --dataset /path/to/dataset --model moco_sat --channels 6 --save_dir /saving/path
```

You will need to modify the data loading part in the script if you use another dataset than the one used in the script.

If a MLflow backend is used, add the experiment ID :

```python
python3 train_pretext.py --dataset /path/to/dataset --model moco_sat --channels 6 --exp_ID 1
```
### Downstream task - Classification
To use SSL pretrained weights to initialize a ResNet classifier for classifying ship/non ship patches, use this command :

```python
python3 downstream_classification.py --run_name test_downstream_class --dataset /path/to/dataset --model moco_sat --channels 6 --save_dir /saving/path --weights /path/to/weights --nb_class 2 --mode ft --test y
```
You will need to modify the data loading part in the script if you use another dataset than the AIS ship detection dataset from Agenium Space.

If you prefer to do transfer learning instead of fine-tuning, change the "mode" argument to "tf". If a MLflow backend is used, add the experiment ID option.

### Downstream task - Segmentation using U-Net
To use SSL pretrained weights to initialize a U-Net classifier to perform ship/non ship segmentation, use this command for training in a leave-one-out setting:

```python
python3 U_net_ship.py --run_name test_unet --train-path /path/to/training/set --test-path /path/to/test/set --in-channels 6 --save_results /saving/path --weights /path/to/weights --num-classes 2 --mode ft
```

If you want to vary the number of training sample and test on only 3 images, use the following command :


```python
python3 U_net_ship.py --run_name test_unet --train-path /path/to/training/set --test-path /path/to/test/set --in-channels 6 --save_results /saving/path --weights /path/to/weights --num-classes 2 --mode ft --vary_nb_img y
```

If you prefer to do transfer learning instead of fine-tuning, change the "mode" argument to "tf". If a MLflow backend is used, add the experiment ID option. If you want to perform several runs, add the "iter" option with the desired number of runs.

### Post-processing - Filtering the predictions obtained on S2-SHIPS dataset
To apply sea/land filtering on the results obtained by the segmentation, use this command :
```python
python3 filter.py --filters /path/to/sea-land-masks/ --json_dir /path/to/coco-annotations --dataset /path/to/predictions/to/filter
```

### Baseline BL-NDWI
To try the baseline BL_NDWI, do as follows :
```python
python3 s2ships_gen_data.py --save_dir /saving/path 
```

## Citing this work

Please cite using the bibtex generated from [the MDPI article page](https://www.mdpi.com/2072-4292/13/21/4255)

for example as:

```
@Article{rs13214255,
AUTHOR = {Ciocarlan, Alina and Stoian, Andrei},
TITLE = {Ship Detection in Sentinel 2 Multi-Spectral Images with Self-Supervised Learning},
JOURNAL = {Remote Sensing},
VOLUME = {13},
YEAR = {2021},
NUMBER = {21},
ARTICLE-NUMBER = {4255},
URL = {https://www.mdpi.com/2072-4292/13/21/4255},
ISSN = {2072-4292},
DOI = {10.3390/rs13214255}
}
```

## License
[MIT](https://choosealicense.com/licenses/mit/)
