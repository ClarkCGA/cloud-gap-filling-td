# Cloud Gap Imputation Training Data Generation
This repository contains the training data generation pipeline for Cloud Gap Imputation fine-tuning of the Prithvi Geospatial Foundation Model. 

You can access the Prithvi model on [Hugging Face](https://huggingface.co/ibm-nasa-geospatial/Prithvi-100M), and read the pre-print paper on [arXiv](https://arxiv.org/abs/2310.18660). 
<br />

## __Introduction__
<br />

This repo uses the NASA's STAC-API to query for HLS imagery given a set of image chip bboxes. The bboxes are generated across the CONUS using code provided in the repo. The bboxes are sampled using the USDA CDL data to ensure a diverse representation of land cover classes in the final dataset. Since the chips are defined based on CDL projection (`EPSG:5070`), each HLS tile is also projected to this CRS during data processing, and the final chips are all in `EPSG:5070`.

There are two parts in this repo:
1. Generating chip bbox in GeoJSON format:
    - The notebook is located under `bbox_generate/`.
    - This process will generate two GeoJSON files one in `EPSG:4326` and one in `EPSG:5070` for chip bboxes. The first one is required to process the query for tiles, and the second one is needed for precise chipping of the tiles. 
2. Querying/downloading/chipping process:
    - The Dockerfile and notebooks are located in the root of the repo.

More information is provided in the notebooks. 
<br />

## Build/Run Docker Environment:
<br />

Build the Docker image as following:
```
docker build -t gfm-gap-data .
```

Run the Docker as following (this should run after changing folder to the current folder of the repo):
```
 docker run -it -v "$(pwd)":/home/ -v <full path of the data folder>:/data/ -p 8888:8888 gfm-gap-data
```
The IP to jupyterlab would be displayed automatically.

*Notice: If running from EC2 you should replace the ip address by the public DNS of the EC2*

<br />

## Generating Chip Bboxes:
To generate the chip bboxes run the notebook `bbox_generate/gen_chip_bbox.ipynb`. This requires the USDA CDL .tiff file to be located at `/data/<name of cdl>`. For the current version of the dataset, we use `2022_30m_cdls.tif`. which can be retrieved from [here](https://www.nass.usda.gov/Research_and_Science/Cropland/SARS1a.php).

<br />

## Running the HLS Chipping Data Pipeline
<br />

### General workflow

The GeoJSON files containing chip bboxes in `EPSG:4326` and `EPSG:5070` should be included under `data/` for the rest of the code to run.  

#### Generating HLS image chips

1. Run the `hls_imgData_pipeline.ipynb` till it says `Run hls_reprojecting.ipynb first`. All tiles should now be downloaded, and we need to reproject them to match the CDL's projection, column and rows.

2. Run the `hls_reprojecting.ipynb` to reproject all downloaded HLS tiles and its bands.

3. Run the rest of `hls_reprojecting.ipynb` to chip and write the chips.

#### Genrating HLS realistic cloud masks

1. Run the `hls_cloudmask_pipeline.ipynb` till it says `Run hls_reprojecting.ipynb first`.

2. Run the `hls_reprojecting.ipynb` to reproject all the fmask tif.

3. Run the rest of `hls_cloudmask_pipeline.ipynb` to chip the fmasks. The fmasks are reclassified to a binary format, and a csv is created to track the could coverage for each cloud mask.

## __Other Information__
- The pipeline for cloud mask and image chips are similar, check the image pipeline for more information about functions and such.
- In case the downloading and reprojecting process failed or stopped due to internet connection issue or memory issue, simply rerun the same code chunk as the pipeline would check if the targeting file exists or not.
- Potentially there will be corrupted or missing tiles when downloading or reprojecting, and most likely due to internet or memory issue. You may end up with less than expected chips, but the loss would be minor.
