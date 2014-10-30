ulavalSFM
=========

Version : 3.0

Author : Émile Robitaille @ LERobot

Last update : 10/05/2014

What is ulavalSFM ?
-------------------

ulavalSFM is a free software manager to prepare and do structure from motion in a parallel way. It's in development. The structure from motion will be based on bundlerSFM : https://github.com/snavely/bundler_sfm. My version of BundlerSFM will though recognize the "ulavalSFM.txt" file and the new "matches.init.txt" file. Your images have to be in JPEG only .jpg ext. Check my gists <a href=https://gist.github.com/LERobot/dbcc6385d83442013592>ext</a> to quickly change the extensions and <a href=https://gist.github.com/LERobot/3985e40fb8e941f90c5c>checkJPG</a> to find hidden PNG file to delete.

Python script usage (multithreading)
------------------------------------

```Bash
mv <your_image_dir_path> <your_ulavalsfm_bin_path>
cd <your_ulavalsfm_bin_path>/<your_image_dir>
python ../bundler.py --no-parallel --verbose --number-cores <number_cores_u_want>
```

You can change some bundlerSFM options if you want. Refer to BundlerSFM repo : https://github.com/snavely/bundler_sfm.

Python script usage (multicores)
--------------------------------

--Unstable now--

Tested on CentOS6 with a Lustre file system. The file system I've been using is named scratch/ and his path is hard coded in the python script. Change the code to make it work on your cluster should not be too difficult.

```Bash
mv <your_image_dir_path> <your_ulavalsfm_bin_path>
cd <your_ulavalsfm_bin_path>/<your_image_dir>
python ../bundler.py --no-parallel --verbose --number-cores <number_cores_u_want> --cluster --walltime <walltime_u_want>
```
You can change some bundlerSFM options if you want. Refer to BundlerSFM repo : https://github.com/snavely/bundler_sfm.

ulavalSFM Manager Usage
-----------------------

#### Displayed on terminal

* -h  ---      : Print this menu
* -v  ---      : Print the software version
* -l [dir]     : Print information about the directory
* -n [1-*]     : Specify the number of core(s) wanted (default 1, means no mpi)
* -s [dir]     : To find sift features of the directory images
* -m [dir]     : To match sift features of the directory images

Don't use these options for now, a simplier python script will follow to do all the algorithms :

* -c [0-1]     : On cluster or not. If 1, a script .sh file will be generated (default 0)
* -b [dir]     : To run bundler on the given directory
* -a [dir]     : Do something alike to "-s dir", "-m dir" and then "-b dir"

#### More details :

-l [dir] : Will give the directory name, number of images, .key files and .mat files.

-c [0-1] : The software will use Torque msub to submit the .sh file. Not implemented yet. You will have the possibility to change the dispatcher in a configuration file.

-n [1-*] : It uses OpenMPI to launch the extern program cDoSift on multiple cores.

-s [dir] : Will do sift detection using OpenCV 2.4.9 implementation and write the features in a Lowe's binairy format.

-m [dir] : Will do match using OpenCV 2.4.9. It uses knn search to find the two best matches and uses a ratio test of 0.6 to eliminate most of bad maches. It will then pruned the double matches, pruned the outliers with a fundamental matrix found using RANSAC and compute geometric constraints needed by bundlerSFM to begin the structure from motion with a homographic matrix found using RANSAC as well. I use OpenCV to compute the matrix and the inliers.

-b [dir] : Run bundlerSFM with the options found in options.txt, automatically generated by the manager.

Notes
-----

#### Sift format

The four numbers before the descriptor in Lowe's sift file format are : 

| X coordinate | Y coordinate | scale | angle |

Note that OpenCV does not give scale so I used size instead to make the format compatible with program like Changchang Wu's visualSFM which natively use the Lowe's format. This will not influence my program because we do not use scale in structure from motion, but keep it in mind if you want to use my sifts for other purposes. 

#### Parallel design

For now, I used a parallel design based on a relation between root, workers and secretary. It is built using MPI.

###### Sift parallel design

There is a root which compute a distribution of all the image. Thanks to the distribution, the root gives a start point and a end point relative to a common loop to each worker (It becomes a worker too). Each worker find sift points and then write it in a \<name\>.key file.

###### Matches parallel design

There is a root which compute a distribution of all the pairs. Thanks to the distribution, the root gives a start point and a end point relative to a common double loop. The root then becomes a secretary. Since only one file will be written, it is easier and, I think, more efficient to handle one writter than handle the synchronisation needed with multi-writter. When a worker finishes a pair, it serializes the information and sends it to the secretary. The secretary quickly push back the information in a c++ vector. I do so, because the secretary have to be always free to receive information. The opposite would block some of the workers. When all the pairs are computed, the secretary write down the information in the file : "matches.init.txt". The design have a limit of images because at a certain point, the secretary could run out of memory. In the worst case, if we consider that each pairs have 8 Kb (~ 2000 matches) and the maximum RAM of the root is 3 Gb, then, it is possible to match a maximum of approximately 850 images. But, because certain pairs are discarted or because pairs usually have a lot less than 2000 matches it is possible to have much more pairs.

Questions ? / Comments ? 
------------------------

Don't hesitate, send me an email
emile.courriel@gmail.com









