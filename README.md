# GFM Gap-Filling Training Data Generation
This repository contains training data generation pipeline for gap filling task of the GFM downstream model evaluation task.
<br />

## __Introduction__
<br />

The current pipeline rely on NASA's STAC-API access to query the data, a geojson that contains each chips' geometry, with the projection of EPSG:4326 is required to process the query for tiles. Another geojson that is in EPSG:5070 projection is also needed for percise chipping of the tiles. A CDL tile is required to match the HLS projection to CDL's grid and projection. 

More information is in the notebooks. 

There are two parts in this repo:
1. Generating the geojson that will be used in the query/download and chipping pipeline
    - The dockerfile and notebook is located in the bbox_generate folder.
2. Whole pipeline of the query/download/chipping process
    - The dockerfile and notebooks are located in the root of the repo.
<br />

## __Instructions on Generating GeoJSON__
<br />

### Build/Run Docker Environment:
<br />

Build the Docker image as following:
```
docker build -t gfm-gap .
```

Run the Docker as following (this should run after changing folder to the current folder of the repo):
```
 docker run -it -p 8888:8888 -v "$(pwd)":/home/workdir gfm-gap
```
The IP to jupyterlab would be displayed automatically.

*Notice: If running from EC2 you should replace the ip address by the public DNS of the EC2*
<br />

### Generating Chip Bboxes:
To generate the chip bboxes run the notebook `gen_chip_bbox.ipynb`. This requires an `aoi.geojson` files that is located in `data/` and has been generated manually. 
<br />

## __Running the HLS V2.0 Data Pipeline__
<br />

### Build/Run Docker Environment:
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

### General workflow
- Run the cdl_generate.ipynb to generate the cdl example tif file that needed to do the reprojection (This only need to be run Once)

- For HLS image chips

1. Run the hls_v2_pipeline.ipynb till the part of downloading tiles. All tiles should now be downloaded, and we need to reproject them to match the CDL's projection, column and rows.

2. Run the hls_reprojecting.ipynb to reproject all downloaded HLS tiles and its bands.

3. Run the rest of hls_reprojecting.ipynb to chip and write the chips.

- For HLS realistic cloud masks.

1. Run the hls_v2_cloudmask_pipeline.ipynb till the part of downloading tiles.

2. Run the hls_reprojecting.ipynb to reproject all the fmask tif.

3. Run the rest of hls_v2_cloudmask_pipeline.ipynb to chip the fmasks. The fmasks are reclassified to a binary format, and a csv is created to track the could coverage for each cloud mask.

## __Other Information__
- The current pipeline works.
- The pipeline for cloud mask and img chips are similar, check the img pipeline for more information about functions and such.
## __Potential Updates__
- Complete: Adding retry for quetry process to prevent from failing due to internect connection issues.
- WIP: Adding multi-processing to query and download of tiles.
