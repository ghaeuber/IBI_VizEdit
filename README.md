# Overview of IBI VizEdit

IBI VizEdit is a program built using RShiny. It is designed to assist in the manual editing of inter-beat interval files that are derived from photoplethysmogram (PPG) recordings. Unlike the electrocardiogram signal (EKG or ECG), PPG signals are characterized by a slow-moving waveform, which present a different set of challenges when the true signal becomes contaminated with motion artefacts and other sources of noise. 

Though increasingly popular due to their ease of use, most heart rate editing software that exists to date was designed and optimized for the detection and editing of inter-beat interval files derived from EKG signals. IBI VizEdit provides a new suite of tools for researchers who find themselves working with messy PPG files. 

Please note that IBI VizEdit is beta software. It has not been fully tested, and there are likely numerous bugs and opportunities to optimize code and performance. Any and all feedback is welcome. 

*As of right now, IBI VizEdit is only supported for use on Windows 7/8/10.* 

Please cite as: 

Barstead, M. G. (2018). IBI VizEdit v.1.0: An RShiny Application [Computer software]. University of Maryland.

## Features and Settings

### 1. Supported File Types 

IBI VizEdit can work with any raw PPG (or even EKG) signal that is saved as a tab-delimited `.txt` file. The data should exist in a single column. If header information exists, that is fine as there is an option when loading the raw file to specify the number of header rows (IBI VizEdit will discard these rows when reading in the data).

The program supports files sampled at a rate of up to 2000 Hz, though it is **strongly** recommended that you make use of the downsampling feature within IBI VizEdit to reduce the file size and overall computation time. You will definitely notice the difference in speed when attempting to use any of the advanced imputation features. You may also find some of the graphics slow to refresh when using even the basic features if raw files are particularly large. The current beta version of the program (v.0.5) has an **overall file size limit of 150MB.**

### 2. Selecting a Working Directory

One slight downside in working with RShiny is that it can be slow in navigating a system's file structure. To help speed this process up, before you go to use IBI VizEdit, place all files you would like to edit in a specific folder. This includes the timing file (more on that below). After you have selected the appropriate folder, choose the file you would like to edit. 

When you save your processed and edited files, the program will create a new output folder that will contain a `Case Processing Summary.rtf` file, a raw `.txt` file containing the downsampled version of your PPG data, a separate time series `.csv` for each epoch interval you selected on the opening screen, a overall summary `.csv` file containing heart rate and heart rate variability scores by task, both edited and raw versions of your processed IBI files (as `.txt` files), and finally, separate IBI `.txt` files separated by task. 

### 3. Selecting File Settings

When initially loading your raw file, there are several important inputs and decisions required. First, to standardize file processing and naming, IBI VizEdit requires that files are named using the following convention: `ID#_Timepoint.txt` . The ID of the participant should be the study ID used by the researcher and the `Timepoint` can be any mix of legal filename characters that specify a study time point (e.g., T1, T2, etc), a group assignment (e.g., Tx, Crtl), or a study task or condition (e.g., Task1, Condition2, etc.). 

After you have input the necessary file information, you have the option of specifying an additional identifier for your output files. For instance if your raw file contains several heart rate signals from separate individuals, you may want to specify which one you are working on using this identifier (e.g., subjA, subjB, etc.). 

Timing files are necessary for IBI VizEdit, even if you don't have any tasks you want to use to separate your output.  Absent any meaningful timing files, the researcher should create a timing file with a column of 0's and a column total with rows indexed by the ID names of the files (see above). 

Once all of the files are loaded, you should specify the column in the PPG data file with the relevant channel, the number of header rows (if any), the sampling rate of the original file, and the downsampling rate of the final file. Also be sure to select the number of different epoch lengths you would like to see in the timeseries data files created by the program. 

## Processing Your File

Once you have loaded the various settings for the program, your next task is to process the PPG file and identify the peaks. First, use the View Plot button on the second panel to ensure that the data has loaded correctly. If you are satisfied that you have the right file loaded, click the "Process File" button. The peak detection algorithm will iteratively identify an optimal processing bandwidth and return a raw interbeat interval file. The processing summary will populate on the second panel when finished. The same table will be included in the `Case_Processing_Summary.rtf` output when you have finished editing your file.

## Editing Your File 

There are two different editing panels as part of IBI VizEdit. The specific portion of the file you decide to view and edit on either the "Basic" or the "Advanced" panel is controlled by the small horizontal graphic at the bottom of the "Basic Editing Panel".  Simply use your mouse to highlight a section of the overall file and VizEdit will display that portion of the file in visual editing interface on both panels.

The basic panel allows you to perform several simple functions. 

1. With add/delete turned on you can manually **ADD** a peak with a single click at the exact point (referenced to the x-axis) you believe a beat should have been identified. 
2. **DELETE** incorrectly identified points. 
3. **COMBINE** two or more interbeat intervals that were incorrectly identified by the peak detection algorithm. 
4. **DIVIDE** a point where the peak detection program "skipped" over the identification of previous heart rate points.
5. **AVERAGE** nearby IBIs when there are issues with detection peak detections.  

The advanced panel offers a new suite of data imputation techniques that, as of yet, have not been included in previous heart rate editing packages. Chief among these techniques is the incorporation of Gaussian process models. 

Gaussian process models are incredibly flexible, albeit computationally demanding. In IBI VizEdit, data imputation via Gaussian process models is implemented using Stan with R's Stan interface package `rstan`. Use of this feature requires separate installation of Stan and `rstan` (for more information visit this site).  

Depending on the downsampling rate, the size of your file, the amount of time that requires imputation, and the consistency of the surrounding data will determine how long the model takes to run. See the IBI VizEdit manual for more information about the Gaussian process models implemented in IBI VizEdit. 

IBI VizEdit also implements a simpler seasonal decomposition approach in order to aid researchers in identifying peaks. While useful for small sections of data, this technique will underestimate variability in heart rate relative to the more involved Gaussian process technique outlined above. 

## What is in Your Output

Once you have finished editing your document and you have selected the "Save" button on the main panel, a separate output directory will be created in your working directory folder that includes: 

1. `ID#_Optional(ID)_Timepoint_Case_Processing_Summary.rtf`: This file includes summary information about the your file and editing choices: 
2. - The output of the peak detection algorithm and the final bandwidth used for identifying peaks. 
   - Average heart period and heart rate variability for the entire file as well as split by task/condition. 
   - An editing summary that identifies every unique edit made including the value of the edited IBI, the time point of the edit, and the type of edit made. 
   - A summary of the results from each Gaussian Process used to impute data, including the final output for parameters estimated by the models.


2. `ID#_Optional(ID)_Timepoint_Hz_PPG.txt`: The downsampled file of the original signal. 
3. `ID#_Optional(ID)_Timepoint_raw_IBI.txt`: The unedited version of the original IBI file as produced by the peak detection algorithm. 
4. `ID#_Optional(ID)_Timepoint_Xs_Epochs.csv`: A time series file with summary statistics computed for heart period (HP), root mean square of successive differences (rmssd), and standard deviation (sd) by user-specified epoch length. If the researcher selects multiple epoch lengths when setting up the editing session, multiple separate files will be included in the output. 
5. `ID#_Optional(ID)_Timepoint_edited_IBI.txt`: The complete edited file. Edited IBI files are also output by by task. 
6. `ID#_Optional(ID)_Timepoint_edited_Task.csv`: Summary values (e.g., HP, rmssd, and sd) separated out by task/condition.  

## Advice on Editing Choices

There are more detailed instructions for using the program in the IBI VizEdit manual. Briefly, it is worth highlighting that in granting researchers more freedom to edit their PPG heart rate files, even greater responsibility is placed on the researcher to check his or her work. 

To ensure responsible and effective use of the program you should ensure that: 

1. You have successfully edited the practice files that are included in this repository. On average, your error rate should be no greater than ___. [The actual value to be determined based on incomplete pilot work.]
2. When editing your own data, be sure to perform post-editing checks. As part of your editing, you can see a running plot in the RStudio plot viewer that plots the number of edits along with the rmssd. In an ideal situation, you would see no correspondence between the number of edits and heart rate variability. This is useful, though probably not a sufficient check to ensure that your edits are not adding any systematic source of variation from the choices you make. 

## Stan and `rstan` in IBI VizEdit

Stan is a program external to R that allows researchers to easily and quickly implement a variety of Bayesian models.  As opposed to other Bayesian modeling software packages such as JAGS or BUGS, Stan runs its models in compiled C++.  This means that the first time a model is run and (requiring the relevant Stan program is compiled) can be a bit slower than subsequent runs. The increased speed in running a compiled program more than makes up for this minor inconvenience. 

Stan has an active [developer community on GitHub](https://github.com/stan-dev), and more information can be found at [mc-stan.org](http://mc-stan.org/). Its incorporation into IBI VizEdit does require some additional setup, however. Detailed instructions for setting up Stan can be found [here](https://github.com/stan-dev/rstan/wiki/Installing-RStan-on-Windows). Be sure to follow the instructions precisely in order to guarantee a clean setup. 

As a reminder, the current version of IBI VizEdit is only supported on Windows 7/8/10. 

## Gaussian Process Modeling

A key novel feature incorporated into IBI VizEdit is the use of Gaussian process models for imputing PPG data in regions where the signal has become corrupted by motion artefact or some other similar source of noise. 

![](C:\Users\Mbars\Documents\GitHub\IBI_VizEdit\Supporting_Documents\Artefact_GP.PNG)

The result of running the program, which admittedly takes up to 5 minutes or so on a 7th generation i7 processor with 16 GB of RAM (and more importantly a maximum processing speed of 3.50 GHz). 

Using the default settings for Gaussian process imputation in IBI VizEdit, on the section of messy PPG data depicted above results in the following estimated waveform: 

*[TO BE ADDED]*

Gaussian process models utilize different covariance functions to model the covariance between timepoints in a univariate time series. More details on the specific features of the covariance functions incorporated in IBI VizEdit's Bayesian GP feature can be found in the manual (which is not finished yet... but then again neither is the program)

Briefly, three different covariance functions are included: 

1. A squared exponential covariance function (a common covariance function - to the point that the Stan Development Team has incorporated a specific function for it in their base code). 
2. A quasi-periodic covariance function based on the individual's heart rate before and after the corrupted section of data. 
3. A periodic covariance function again based on proximal heart rate, but with the added feature that the heart rate can vary as a function of the participants' average respiration rate. 

Details on the specific formula, code and rationale for these functions can be found in the forthcoming IBI VizEdit manual. 



