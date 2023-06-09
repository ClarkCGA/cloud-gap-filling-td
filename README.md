# gfm-gap-filling-td
Training data for gap filling task of the GFM downstream model evaluation 


### Instructions
Build the Docker image as following:
```
docker build -t gfm-gap .
```

Run the Docker as following (this should run after changing folder to the current folder of the repo):
```
 docker run -it -p 8888:8888 -v "$(pwd)":/home/workdir gfm-gap
```

### Generating Chip Bboxes
To generate the chip bboxes run the notebook `gen_chip_bbox.ipynb`. This requires an `aoi.geojson` files that is located in `data/` and has been generated manually. 
