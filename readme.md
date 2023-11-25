# BundleSDF: Neural 6-DoF Tracking and 3D Reconstruction of Unknown Objects
# Data download
- Download pretrained [weights of segmentation network](https://drive.google.com/file/d/1MEZvjbBdNAOF7pXcq6XPQduHeXB50VTc/view?usp=share_link), and put it under
`./BundleTrack/XMem/saves/XMem-s012.pth`

- Download pretrained [weights of LoFTR outdoor_ds.ckpt](https://drive.google.com/drive/folders/1xu2Pq6mZT5hmFgiYMBT9Zt8h1yO-3SIp), and put it under
`./BundleTrack/LoFTR/weights/outdoor_ds.ckpt`

- Download HO3D data. We provide the augmented data that you can download [here](https://drive.google.com/drive/folders/1Wk-HZDvUExyUrRn7us4WWEbHnnFHgOAX?usp=share_link). Then download YCB-Video object models from [here](https://drive.google.com/file/d/1-1m7qMMyUHYLhaRiQBbsSRMt5dMRX4jD/view?usp=share_link). Finally, make sure the structure is like below, and update your root path of `HO3D_ROOT` at the top of `BundleTrack/scripts/data_reader.py`
  ```
  HO3D_v3
    ├── evaluation
    ├── models
    └── masks_XMem
  ```

# Docker/Environment setup
- Clone the repository.
```
git clone https://github.com/ArghyaChatterjee/BundleSDF.git
```
- Build the docker image (this only needs to do once and can take some time).
```
cd ~/BundleSDF/docker
docker build --network host -t nvcr.io/nvidian/bundlesdf .
```

- Start a docker container
```
cd docker && bash run_container.sh
```
- Inside docker container, compile the packages which are machine dependent
```
bash build.sh
```
- Do some changes inside your setup:
```
# Update all packages in the environment
conda update --all   
# Reinstall scipy:
conda install --force-reinstall scipy 
# Check the version of libstdc++.so.6 in your environment to see if it has been updated
strings /opt/conda/envs/py38/lib/libstdc++.so.6 | grep GLIBCXX
# Source the path
export LD_LIBRARY_PATH=/opt/conda/envs/py38/lib:$LD_LIBRARY_PATH
```

# Run on example HO3D dataset
```
# Run BundleSDF to get the pose and reconstruction results
python run_ho3d.py --video_dirs /home/arghya/BundleSDF/HO3D_v3/evaluation/SM1 --out_dir /home/arghya/BundleSDF/ho3d_v3_generated_mesh

# Benchmark the output results
python benchmark_ho3d.py --video_dirs /home/arghya/BundleSDF/HO3D_v3/evaluation/SM1 --out_dir /home/arghya/BundleSDF/ho3d_v3_generated_mesh
```

# Run on your custom data
- Prepare your RGBD video folder as below (also refer to the example milk data). You can find an [example milk data here](https://drive.google.com/file/d/1akutk_Vay5zJRMr3hVzZ7s69GT4gxuWN/view?usp=share_link) for testing.
```
root
  ├──rgb/    (PNG files)
  ├──depth/  (PNG files, stored in mm, uint16 format. Filename same as rgb)
  ├──masks/       (PNG files. Filename same as rgb. 0 is background. Else is foreground)
  └──cam_K.txt   (3x3 intrinsic matrix, use space and enter to delimit)
```
Due to license issues, we are not able to include [XMem](https://github.com/hkchengrex/XMem) in this codebase for running segmentation online. If you are interested in doing so, please download the code separately and add a wrapper in `segmentation_utils.py`.

- Run your RGBD video (specify the video_dir and your desired output path). There are 3 steps.
```
# 1) Run joint tracking and reconstruction
python run_custom.py --mode run_video --video_dir /home/bowen/debug/2022-11-18-15-10-24_milk --out_folder /home/bowen/debug/bundlesdf_2022-11-18-15-10-24_milk --use_segmenter 1 --use_gui 1 --debug_level 2

# 2) Run global refinement post-processing to refine the mesh
python run_custom.py --mode global_refine --video_dir /home/bowen/debug/2022-11-18-15-10-24_milk --out_folder /home/bowen/debug/bundlesdf_2022-11-18-15-10-24_milk   # Change the path to your video_directory

# 3) (Optional) If you want to draw the oriented bounding box to visualize the pose, similar to our demo
python run_custom.py --mode draw_pose --out_folder /home/bowen/debug/bundlesdf_2022-11-18-15-10-24_milk
```

- Finally the results will be dumped in the `out_folder`, including the tracked poses stored in `ob_in_cam/` and reconstructed mesh with texture `textured_mesh.obj`.

<img src="./media/milk_jug.gif" height="400">
<img src="./media/problem_setup_c.gif" width="80%">

<img src="./media/preview_results_c.gif" width="80%">

<img src="./media/driller.gif" width="80%">

# Contact
For questions, please contact Bowen Wen (bowenw@nvidia.com)
