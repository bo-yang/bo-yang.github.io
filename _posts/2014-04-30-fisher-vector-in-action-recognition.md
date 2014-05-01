---
layout: post
title: Fisher Vector for Action Recognition
categories: 
- Computer Vision
tags:
- Computer Vision
- Fisher Vector
- SVM
status: publish
type: post
published: true
author: Bo Yang
---

This is a summary of doing human action recognition using Fisher Vector with (Improved) Dense Trjectory Features(DTF, http://lear.inrialpes.fr/~wang/improved_trajectories) and STIP features(http://crcv.ucf.edu/ICCV13-Action-Workshop/download.html) on UCF 101 dataset(<http://crcv.ucf.edu/data/UCF101.php>). In the STIP features, two low-level visual features HOG and HOF are integrated, with dimensions 72 and 90 respectively. The (improved) DTF employ more features(TR, HOG, HOF and MBHx/MBHy) with longer dimensions. 

You can find my Matlab code from [my GitHub Channel](https://github.com/bo-yang):
- [DTF + Fisher Vector code](https://github.com/bo-yang/dtf_fisher)
- [STIP + Fisher Vector code](https://github.com/bo-yang/stip_fisher)

### Dense Trajectory Features

For some details of DTF, please refer to [my previous post](http://bo-yang.github.io/2014/01/10/dense-trajectory-notes/).

### Pipeline

The pipeline of integrating DTF/STIP features and Fisher vectors is shown in Figure 2. The first step is subsampling a fixed number of STIP/DTF features(in my implementation, 1000) from each
video clip in training list, which will be used to do PCA and train
Gaussian Mixture Models(GMMs).

After getting the PCA coefficients and GMM parameters, treat UCF 101 video clips
action by action. For each action, first load all train videos in this
action(positive videos), and then randomly load the same number of video
clips not in this action(negative videos). All of the loaded videos are
multiplied with the saved PCA coefficients in order to reduce dimensions
and rotate matrices. Fisher vectors are computed for each loaded video
clip. Finally a binary SVM model is trained with both the positive and negative
Fisher vectors.

When dealing with the test videos, similar process is adopted. The only
difference is that the Fisher vectors are used for SVM classification,
which is based on the SVM model trained with training videos.

![Figure 2. Pipeline of UCF101 action recognition using Fisher
vector.]({{ site.url }}/assets/images/fisher_pipeline.png).

To well utilize the STIP or DTF features, features(HOG, HOF, MBH, etc.) are treated
separately and they are only combined(simple concatenation) after computing Fisher
vectors before linear SVM classification.


### Pre-processing

#### STIP Features

The offcial STIP features are stored in class, which means that all the STIP
info of all video clips in each class are mixed together in a file. To
extract STIP features for each video, I wrote a script(mk_stip_data)
to separate STIP features for each video clip. And all the following
operations are based on each video clip.


#### DTF Features

Since the DTF features are "dense"(which means a lot of data), it took me 4~5 days to exact the (improved) DTF features of UCF 101 clips with the dedault parameters on a modern Linux desktop(I used 10 threads for extraction in paralle). The installation of DTF tools was also a very tricky task.

To save space, all the DTF features were compressed. For UCF101, it would cost about 500GB after zip compression. And the required space would doubled if not compressing. If you don't want to save the DTF features, you can call the DTF tools in Matlab and discard the extracted features.


### Fisher Vector

The Fisher Vector (FV) representation of visual features is an extension of the popular bag-of-visual words (BOV)[1]. Both of them are based on an intermediate representation, the visual vocabulary built in the low level feature space. A probability density function (in most cases a Gaussian Mixture Model) is used to model the visual vocabulary, and we can compute the gradient of the log likelihood with respect to the parameters of the model to represent an image or video. The Fisher Vector is the concatenation of these partial derivatives and describes in which direction the parameters of the model should be modified to best fit the data. This representation has the advantage to give similar or even better classification performance than BOV obtained with supervised visual vocabularies.  

Following is the algorithm of computing Fisher vectors from features:
![Figure 1. Algorithm of computing Fisher vectors.]({{ site.url }}/assets/images/fisher_vector_algorithm.png).

During the subsampling of STIP features, I randomly chose 1000 HOG or
HOF features from each training video clip. For some videos, if the
total number of features were less than 1000, I would use all of their
features. All the subsampled features are square rooted after L1
normalization.

After that, the dimensions of the subsampled features were reduced to
half of their original dimensions by doing PCA. At this step, the
coefficients of PCA were recorded, which would be used in later. The
GMMs were trained with the half-sized features, and the parameters of
GMMs(i.e. weight, mean and covariance) were stored for the following
process. In my program, the GMM code implemented by Oxford Visual
Geometry
Group(VGG, http://www.robots.ox.ac.uk/~vgg/software/enceval_toolkit/)
is used, which eventually call VLFeat(http://www.vlfeat.org/). In my
code, 256 Gaussians were used.


### SVM Classification

Binary SVM classification([LIBSVM](http://www.csie.ntu.edu.tw/~cjlin/libsvm/)) is used in my implementation.
For each action, positive video clips are labeled as 1 while negative
videos are as labeled -1 during training and test. In my code, SVM cost
is set to 100. The option of SVM training is:

> -t 0 -s 0 -q -c 100 -b 1.


### Results

The action recognition accuracy of all the 101 actions was 77.95% when
using above pipeline. And the confusion matrix is shown in Figure 3.

![Figure 3. Confusion matrix of all the 101 actions with STIP features.]({{ site.url }}/assets/images/confusionmat_101.png)

The mean accurary of the fist 10 actions with DTF features was 90.6%, while the STIP was only 84.32%. The mean accuracy of the whole UCF 101 data(train/test list 1) was about 85%, about 8% higher than using BOV representation. And the best result I got with the ISA network on UCF 101 was only 58% last year.


### Conclusion

It is obvious that Fisher vector can lead to better results than Bag-of-visual words in action recognition. Compared to other low-level visual features, DTF features have more advantages in action recognition. However, in the long run I still believe deep learning methods - when deep neural networks could be trained with millions of vidios[5], they would learn more info from scratch and achieve state-of-the-art accuracy.


### References

1. Gabriela Csurka, Florent Perronnin, _Fisher Vectors: Beyond Bag-of-Visual-Words Image Representations_ , Communications in Computer and Information Science Volume 229, 2011, pp 28-42.
2. Chih-Chung Chang and Chih-Jen Lin. _Libsvm: A library for support vector machines_. ACM Trans. Intell. Syst. Technol., 2(3):27:1–27:27, May 2011.
3. Heng Wang and Cordelia Schmid. _Action Recognition with Improved Trajectories_. In ICCV 2013 - IEEE International Conference on Computer Vision, Sydney, Australia, December 2013. IEEE.
4. Jorge Sanchez, Florent Perronnin, Thomas Mensink, and Jakob Verbeek. _Image Classification with the Fisher Vector: Theory and Practice_. International Journal of Computer Vision, 105(3):222–245, December 2013.
5. Andrej Karpathy, George Toderici, Sanketh Shetty, Thomas Leung, Rahul Sukthankar, and Li Fei-Fei. _Large-scale video classification with convolutional neural networks_. In CVPR, 2014.

