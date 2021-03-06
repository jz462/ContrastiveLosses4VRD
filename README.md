# Graphical Contrastive Losses for Scene Graph Parsing

This is a PyTorch implementation for [Graphical Contrastive Losses for Scene Graph Parsing, CVPR2019](https://arxiv.org/abs/1903.02728).

**Warning:** This is forked from [Nvidia's official repository](https://github.com/NVIDIA/ContrastiveLosses4VRD) and may be out-of-date. Please refer to that official repository for latest updates.

## Cloning
```
git clone https://github.com/NVIDIA/ContrastiveLosses4VRD.git --recurse-submodules

```

## Requirements
The requirements are the same with [Detectron.pytorch](https://github.com/roytseng-tw/Detectron.pytorch):
* Python 3
* Python packages
  * pytorch 0.4.0 or 0.4.1.post2 (not guaranteed to work on newer versions)
  * cython
  * matplotlib
  * numpy
  * scipy
  * opencv
  * pyyaml
  * packaging
  * [pycocotools](https://github.com/cocodataset/cocoapi)
  * tensorboardX
  * tqdm
  * pillow
  * scikit-image
* An NVIDIA GPU and CUDA 8.0 or higher. Some operations only have gpu implementation.

* (Optional) A dockerfile with all necessary dependencies is included in docker/Dockerfile. Requires nvidia-docker

```
# ROOT=path/to/cloned/repository
cd $ROOT/docker
# build the docker image and tag it
docker build -t myname/mydockertag:1.0
# launch an interactive session with this folder
nvidia-docker run -v $ROOT:/workspace/visual-relationship-detection:rw -it myname/mydockertag:1.0
# NOTE: you may need to mount other volumes depending on where your datasets are stored
```

## Compilation
Compile the CUDA code in the Detectron submodule and in the repo:
```
# ROOT=path/to/cloned/repository
cd $ROOT/Detectron_pytorch/lib
sh make.sh
cd $ROOT/lib
sh make.sh
```

## Annotations

Create a data folder at the top-level directory of the repository:
```
# ROOT=path/to/cloned/repository
cd $ROOT
mkdir data
```
If necessary, one may edit the `DATA_DIR` field in lib/core/config.py to change the expected path to the data directory. Be sure to update the paths in the VRD preprocessing scripts (mentioned below) if this is done.

### OpenImages/OpenImages_mini
Download it [here](https://drive.google.com/open?id=1GeUEsiS9Z3eRYnH1GPUz99wjQwjcHl6n). Unzip it under the data folder. You should see an `openimages_v4` folder unzipped there. It contains .json annotation files for both OpenImages and OpenImages_mini, which is a subset of the former created by us including 4500 train and 1000 test images. The .json files are created based on the original .csv annotations.

### Visual Genome
Download it [here](https://drive.google.com/open?id=1VDuba95vIPVhg5DiriPtwuVA6mleYGad). Unzip it under the data folder. You should see a `vg` folder unzipped there. It contains .json annotations that suit the dataloader used in this repo.

### Visual Relation Detection

See [Images:VRD](#visual-relation-detection-1)

## Images

### OpenImages
Create a folder `train/` for the training images:
```
# ROOT=path/to/cloned/repository
cd $ROOT/data/openimages_v4
mkdir train
```
Download OpenImages v4 training images from the [official page](https://storage.googleapis.com/openimages/web/download.html) (**Warning: this is a very large dataset**). **Note:** only training images are needed since our annotations will split them into a train and a validation set. Put all images in `train/`

### Visual Genome
Create a folder for all images:
```
# ROOT=path/to/cloned/repository
cd $ROOT/data/vg
mkdir VG_100K
```
Download Visual Genome images from the [official page](https://visualgenome.org/api/v0/api_home.html). Unzip all images (part 1 and part 2) into `VG_100K/`. There should be a total of 108249 files.

### Visual Relation Detection
Create the vrd folder under `data`:
```
# ROOT=path/to/cloned/repository
cd $ROOT/data/vrd
```
Download the original annotation json files from [here](https://cs.stanford.edu/people/ranjaykrishna/vrd/) and unzip `json_dataset.zip` here. The images can be downloaded from [here](http://imagenet.stanford.edu/internal/jcjohns/scene_graphs/sg_dataset.zip). Unzip `sg_dataset.zip` to create an `sg_dataset` folder in `data/vrd`. Next run the preprocessing scripts:

```
cd $ROOT
python tools/rename_vrd_with_numbers.py
python tools/convert_vrd_anno_to_coco_format.py
```
`rename_vrd_with_numbers.py` converts all non-jpg images (some images are in png or gif) to jpg, and renames them in the {:012d}.jpg format (e.g., "000000000001.jpg"). It also creates new relationship annotations other than the original ones. This is mostly to make things easier for the dataloader. The filename mapping from the original is stored in `data/vrd/*_fname_mapping.json` where "*" is either "train" or "val".

`convert_vrd_anno_to_coco_format.py` creates object detection annotations from the new annotations generated above, which are required by the dataloader during training.

## Pre-trained Object Detection Models
Download pre-trained object detection models [here](https://drive.google.com/open?id=1NrqOLbMa_RwHbG3KIXJFWLnlND2kiIpj). Unzip it under the root directory. **Note:** We do not include code for training object detectors. Please refer to  [Detectron.pytorch](https://github.com/roytseng-tw/Detectron.pytorch) for this.

## Our Trained Relationship Detection Models
Download our trained models [here](https://drive.google.com/open?id=1mVnkZXdlg1ClVF5cGrSgQm31Q3Z0ZcNX). Unzip it under the root folder and you should see a `trained_models` folder there.

## Directory Structure
The final directories for data and detection models should look like:
```
|-- detection_models
|   |-- oi_rel
|   |   |-- X-101-64x4d-FPN
|   |   |   |-- model_step599999.pth
|   |-- vg
|   |   |-- VGG16
|   |   |   |-- model_step479999.pth
|   |   |-- X-101-64x4d-FPN
|   |   |   |-- model_step119999.pth
|   |-- vrd
|   |   |-- VGG16
|   |   |   |-- model_step4499.pth
|-- data
|   |-- openimages_v4
|   |   |-- train    <-- (contains OpenImages_v4 training/validation images)
|   |   |-- rel
|   |   |   |-- rel_only_annotations_train.json
|   |   |   |-- rel_only_annotations_val.json
|   |   |   |-- ...
|   |-- vg
|   |   |-- VG_100K    <-- (contains Visual Genome all images)
|   |   |-- rel_annotations_train.json
|   |   |-- rel_annotations_val.json
|   |   |-- ...
|   |-- vrd
|   |   |-- train_images    <-- (contains Visual Relation Detection training images)
|   |   |-- val_images    <-- (contains Visual Relation Detection validation images)
|   |   |-- new_annotations_train.json
|   |   |-- new_annotations_val.json
|   |   |-- ...
|-- trained_models
|   |-- oi_mini_X-101-64x4d-FPN
|   |   |-- model_step6749.pth
|   |-- oi_X-101-64x4d-FPN
|   |   |-- model_step80929.pth
|   |-- vg_VGG16
|   |   |-- model_step62722.pth
|   |-- vg_X-101-64x4d-FPN
|   |   |-- model_step62722.pth
|   |-- vrd_VGG16_IN_pretrained
|   |   |-- model_step7559.pth
|   |-- vrd_VGG16_COCO_pretrained
|   |   |-- model_step7559.pth
```

## Evaluating Pre-trained Relationship Detection models

DO NOT CHANGE anything in the provided config files(configs/xx/xxxx.yaml) even if you want to test with less or more than 8 GPUs. Use the environment variable `CUDA_VISIBLE_DEVICES` to control how many and which GPUs to use. Remove the
`--multi-gpu-test` for single-gpu inference.

### OpenImages_mini
To test a trained model using a ResNeXt-101-64x4d-FPN backbone, run
```
python ./tools/test_net_rel.py --dataset oi_rel_mini --cfg configs/oi_rel_mini/e2e_faster_rcnn_X-101-64x4d-FPN_12_epochs_oi_rel_mini_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5.yaml --load_ckpt trained_models/oi_mini_X-101-64x4d-FPN/model_step6749.pth --output_dir Outputs/oi_mini_X-101-64x4d-FPN --multi-gpu-testing --do_val
```
This should reproduce the numbers shown at the last line of Table 1 in the paper. 

### OpenImages
To test a trained model using a ResNeXt-101-64x4d-FPN backbone, run
```
python ./tools/test_net_rel.py --dataset oi_rel --cfg configs/oi_rel/e2e_faster_rcnn_X-101-64x4d-FPN_12_epochs_oi_rel_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5.yaml --load_ckpt trained_models/oi_X-101-64x4d-FPN/model_step80929.pth --output_dir Outputs/oi_X-101-64x4d-FPN --multi-gpu-testing --do_val
```

### Visual Genome
**NOTE:** May require at least 64GB RAM to evaluate on the Visual Genome test set

We use three evaluation metrics for Visual Genome:
1. SGDET: predict all the three labels and two boxes
1. SGCLS: predict subject, object and predicate labels given ground truth subject and object boxes
1. PRDCLS: predict predicate labels given ground truth subject and object boxes and labels

To test a trained model using a VGG16 backbone with "SGDET", run
```
python ./tools/test_net_rel.py --dataset vg --cfg configs/vg/e2e_faster_rcnn_VGG16_8_epochs_vg_v3_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5_no_spt.yaml --load_ckpt trained_models/vg_VGG16/model_step62722.pth --output_dir Outputs/vg_VGG16 --multi-gpu-testing --do_val
```
Use `--use_gt_boxes` option to test it with "SGCLS"; use `--use_gt_boxes --use_gt_labels` options to test it with "PRDCLS". The results will vary slightly with the last line of Table 6 in the paper.

To test a trained model using a vg_X-101-64x4d-FPN backbone with "SGDET", run
```
python ./tools/test_net_rel.py --dataset vg --cfg configs/vg/e2e_faster_rcnn_X-101-64x4d-FPN_8_epochs_vg_v3_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5.yaml --load_ckpt trained_models/vg_X-101-64x4d-FPN/model_step62722.pth --output_dir Outputs/vg_X-101-64x4d-FPN --multi-gpu-testing --do_val
```
Use `--use_gt_boxes` option to test it with "SGCLS"; use `--use_gt_boxes --use_gt_labels` options to test it with "PRDCLS". The results will vary slightly with those at the last line of Table 1 in the supplementary.

### Visual Relation Detection
To test a trained model initialized by an ImageNet pre-trained VGG16 model, run
```
python ./tools/test_net_rel.py --dataset vrd --cfg configs/vrd/e2e_faster_rcnn_VGG16_16_epochs_vrd_v3_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5_IN_pretrained.yaml --load_ckpt trained_models/vrd_VGG16_IN_pretrained/model_step7559.pth --output_dir Outputs/vrd_VGG16_IN_pretrained --multi-gpu-testing --do_val
```
The results are slightly different with those at the second to the last line of Table 7.

To test a trained model initialized by an COCO pre-trained VGG16 model, run
```
python ./tools/test_net_rel.py --dataset vrd --cfg configs/vrd/e2e_faster_rcnn_VGG16_16_epochs_vrd_v3_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5_COCO_pretrained.yaml --load_ckpt trained_models/vrd_VGG16_COCO_pretrained/model_step7559.pth --output_dir Outputs/vrd_VGG16_COCO_pretrained --multi-gpu-testing --do_val
```
The results are slightly different with those at the last line of Table 7.

## Training Relationship Detection Models

The section provides the command-line arguments to train our relationship detection models given the pre-trained object detection models described above. **Note:** We do not train object detectors here. We only use trained object detectors (provided in `detection_models/`) to initialize our to-be-trained relationship models.

DO NOT CHANGE anything in the provided config files(configs/xx/xxxx.yaml) even if you want to train with less or more than 8 GPUs. Use the environment variable `CUDA_VISIBLE_DEVICES` to control how many and which GPUs to use.

With the following command lines, the training results (models and logs) should be in `$ROOT/Outputs/xxx/` where `xxx` is the .yaml file name used in the command without the ".yaml" extension. If you want to test with your trained models, simply run the test commands described above by setting `--load_ckpt` as the path of your trained models.

### OpenImages_mini
To train our relationship network using a ResNeXt-101-64x4d-FPN backbone, run
```
python tools/train_net_step_rel.py --dataset oi_rel_mini --cfg configs/oi_rel_mini/e2e_faster_rcnn_X-101-64x4d-FPN_12_epochs_oi_rel_mini_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5.yaml --nw 8 --use_tfboard
```

### OpenImages
To train our relationship network using a ResNeXt-101-64x4d-FPN backbone, run
```
python tools/train_net_step_rel.py --dataset oi_rel --cfg configs/oi_rel/e2e_faster_rcnn_X-101-64x4d-FPN_12_epochs_oi_rel_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5.yaml --nw 8 --use_tfboard
```

### Visual Genome
To train our relationship network using a VGG16 backbone, run
```
python tools/train_net_step_rel.py --dataset vg --cfg configs/vg/e2e_faster_rcnn_VGG16_8_epochs_vg_v3_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5_no_spt.yaml --nw 8 --use_tfboard
```

To train our relationship network using a ResNeXt-101-64x4d-FPN backbone, run
```
python tools/train_net_step_rel.py --dataset vg --cfg configs/vg/e2e_faster_rcnn_X-101-64x4d-FPN_8_epochs_vg_v3_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5.yaml --nw 8 --use_tfboard
```

### Visual Relation Detection
To train our relationship network initialized by an ImageNet pre-trained VGG16 model, run
```
python tools/train_net_step_rel.py --dataset vrd --cfg configs/vrd/e2e_faster_rcnn_VGG16_16_epochs_vrd_v3_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5_IN_pretrained.yaml --nw 8 --use_tfboard
```

To train our relationship network initialized by a COCO pre-trained VGG16 model, run
```
python tools/train_net_step_rel.py --dataset vrd --cfg configs/vrd/e2e_faster_rcnn_VGG16_16_epochs_vrd_v3_default_node_contrastive_loss_w_so_p_aware_margin_point2_so_weight_point5_COCO_pretrained.yaml --nw 8 --use_tfboard
```

## Acknowledgements
This repository uses code based on the [Neural-Motifs](https://github.com/rowanz/neural-motifs) source code from Rowan Zellers, as well as
code from the [Detectron.pytorch](https://github.com/roytseng-tw/Detectron.pytorch) repository by Roy Tseng. See LICENSES for additional details.

## Citing
If you use this code in your research, please use the following BibTeX entry.
```
@conference{zhang2019vrd,
  title={Graphical Contrastive Losses for Scene Graph Generation},
  author={Zhang, Ji and Shih, Kevin J. and Elgammal, Ahmed and Tao, Andrew and Catanzaro, Bryan},
  booktitle={CVPR},
  year={2019}
}
