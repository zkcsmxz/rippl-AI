# rippl-AI

__rippl-AI__ is an open toolbox of several artificial intelligence (AI) approaches for sharp-wave ripple (SWR) detection. In particular, this toolbox offers multiple successful plug-and-play models from 5 AI architectures (1D-CNN, 2D-CNN, LSTM, SVM and XGBoost) that are ready to use to detect SWRs in hippocampal recordings. Moreover, there is an additional package that allows easy re-training, so that models are updated to better detect particular features of your own recordings. 

# Description

## Sharp-wave ripples

Sharp-wave ripples (SWRs) are transient fast oscillatory events (100-250Hz) of around 50ms that appear in the hippocampus, that had been associated with memory consolidation. During SWRs, sequential firing of ensembles of neurons are _replayed_, reactivating memory traces of previously encoded experiences. SWR-related interventions can influence hippocampal-dependent cognitive function, making their detection crucial to understand underlying mechanisms. However, existing SWR identification tools mostly rely on using spectral methods, which remain suboptimal.

Because of the micro-circuit properties of the hippocampus, CA1 SWRs share a common profile, consisting of a _ripple_ in the _stratum pyramidale_, and a _sharp-wave_ deflection in _stratum radiatum_ that reflects the large excitatory input that comes from CA3. Yet, SWRs can extremely differ depending on the underlying reactivated circuit. This continuous recording shows this variability:

![Example of several SWRs](https://github.com/PridaLab/rippl-AI/blob/main/figures/ripple-variability.png)

## Artificial intelligence architectures

In this project, we take advantage of supervised machine learning approaches to train different AI architectures so they can unbiasedly learn to identify signature SWR features on raw Local Field Potential (LFP) recordings. These are the explored architectures:

### Convolutional Neural Networks (CNNs)

![Convolutional Neural Networks](https://github.com/PridaLab/rippl-AI/blob/main/figures/CNNs.png)

### Long-Short Term Memory Recurrent Neural Networks (LSTM)

![Long-Short Term Memory Recurrent Neural Networks](https://github.com/PridaLab/rippl-AI/blob/main/figures/LSTM.png)

### Support Vector Machine (SVM)

![Support Vector Machine](https://github.com/PridaLab/rippl-AI/blob/main/figures/SVM.png)

### Extreme-Gradient Boosting (XGBoost)

![Extreme-Gradient Boosting](https://github.com/PridaLab/rippl-AI/blob/main/figures/XGBoost.png)



# The toolbox

This toolbox contains three main blocks: **detection**, **re-training** and **exploration**. These three packages can be used jointly or separatedly. We will proceed to describe each of their purpose and usage.

## Detection

In previous works ([Navas-Olive, Amaducci et al, 2022](https://elifesciences.org/articles/77772)), we demonstrated that using feature-based algorithms to detect electrophysiological events, such as SWRs, had several advantages:
* Performance lies within the expert range
* It is more stable and less biased than spectral methods
* It can detect a wider variety of SWRs
* It can be used as an interpretation tool
All this is available in our [cnn-ripple repository](https://github.com/PridaLab/cnn-ripple).

In this toolbox, we widen the machine learning spectrum, by offering multiple plug-and-play models, from very different AI architectures: 1D-CNN, 2D-CNN, LSTM, SVM and XGBoost. We performed an exhaustive parametric search to find different `architecture` solutions (i.e. `model`s) that achieve:
* **High performance**, so detections were as similar as manually labeled SWRs
* **High stability**, so performance does not depend on threshold selection
* **High generability**, so performance remains good on very different contexts

This respository contains the best five `model`s from each of these five `architecture`s. These `model`s are already trained using mice data, and can be found in the [optimized_models/](https://github.com/PridaLab/rippl-AI/blob/main/optimized_models/) folder. 

The [rippl_AI](https://github.com/PridaLab/rippl-AI/blob/main/rippl_AI.py) python module contains all the necessary functions to easily use any `model` to detect SWRs. Additionally, we also provide some auxiliary functions in the [aux](https://github.com/PridaLab/rippl-AI/blob/main/aux.py) module, that contains useful code to process LFP and evaluate performance detection.

Moreover, several usage examples of all functions can be found in the [examples_detection/](https://github.com/PridaLab/rippl-AI/blob/main/examples_detection/) folder.



### rippl_AI.predict()

The python function `predict()` of the `rippl_AI` module computes the SWR probability for a give LFP. 

In the figure below, you can see an example of a high-density LFP recording (top) with manually labeled data (gray). The objective of these `model`s is to generate an output signal that most similarly matches the manually labeled signal. The output of the uploaded optimized models can be seen in the bottom, where outputs go from 0 (low probability of SWR) to 1 (high probability of SWR).

![Detection method](https://github.com/PridaLab/rippl-AI/blob/main/figures/output-probabilities.png)

The `rippl_AI.predict()` input and output variables are:

* Mandatory inputs:

* Optional inputs:

* Output:



### rippl_AI.get_intervals()

The python function `get_intervals()` of the `rippl_AI` module takes the output of `rippl_AI.predict()` (i.e. the SWR probability), and identifies SWR beginnings and ends by stablishing a threshold. In the figure below, you can see how the threshold can decisevely determine what events are being detected. For example, lowering the threshold to 0.5 would have result in XGBoost correctly detecting the first SWR, and the 1D-CNN detecting the sharp-wave that has no ripple.

![Detection method](https://github.com/PridaLab/rippl-AI/blob/main/figures/detection-method.png)

* **Setting the threshold**

	- `get_intervals(SWR_prob)`: displays a histogram of all SWR probability values, and a draggable threshold to set a threshold based on the values of this particular session. When 'Done' button is pressed, the GUI takes the value of the draggable as the threshold.

	- `get_intervals(SWR_prob, LFP=LFP, sf=sf, win_size=win_size)`: if `LFP` is also added as an input, then the GUI adds up to 50 examples of SWR detections. If the 'Update' button is pressed, another 50 random detections are shown. When 'Done' button is pressed, the GUI takes the value of the draggable as the threshold. Sampling frequency `sf` (in Hz) and window size `win_size` (in seconds) can be used to set the window length of the displayed examples. It automatically discards false positives due to drifts, but if you want to set it off, you can set `discard_drift` to `false`. By default, it discards noises whose mean LFP is above `std_discard` times the standard deviation, which by default is 1SD. This parameter can also be changed. ![Detection method](https://github.com/PridaLab/rippl-AI/blob/main/figures/threshold-selection.png)

	- `get_intervals(SWR_prob, 'threshold', threshold)`: if a threshold is given, then it takes that threshold without displaying any GUI.



* Additional optional inputs are: `handle_overlap` and `verbose`. In order to handle prediction of overlapping windows, use `handle_overlap` to choose to do the 'mean' (by default) or 'max'. `verbose` prints the description of internal processes (false by default).

* **Getting the intervals**
	Once the threshold is set, it automatically finds the intervals where `SWR_prob` is over that threshold. The output of `get_intervals()` is a `n_ripples` x 2 array of beginnings and ends. If `sf` has been given, the output is in seconds; if it's not given, then the output is in timestamps.






### aux.process_LFP()

The python function `process_LFP()` of the `aux` module processes the LFP before it is input to the algorithm. It downsamples LFP to 1250 Hz, and normalizes each channel separately by z-scoring them.

* Mandatory inputs:

* Optional inputs:

* Output:


### aux.interpolate_channels()

The python function `interpolate_channels()` of the `aux` module...

* Mandatory inputs:

* Optional inputs:

* Output:


### aux.get_performance()

The python function `get_performance()` of the `aux` module...

* Mandatory inputs:

* Optional inputs:

* Output:




## Re-training




## Exploration





# Enviroment setup

On Windows 10:
  Using the enviroment file 'env_PublicBCG_d.yml':
1. Install miniconda, following the tutorial: https://docs.conda.io/en/latest/miniconda.html
2. Launch the anaconda console, typing anaconda promp in the windows search bar.
3. Navigate to the project directory, using 'cd' to change folders, for example:
  cd C:\Myprojects\Public_BCG_models
4. Type in the anaconda prompt: 
   conda env create -f rippl_AI_env.yml
5. This will create a enviroment in your miniconda3 enviroments folder, usually:
  C:\Users\<your_user>\miniconda3\envs
6. Check with:
  conda env list
  The enviroment 'rippl_AI_env' should be included in the list
7. Activate the enviroment with 'conda activate rippl_AI_env', in case you want to launch the scripts from the command prompt. If you are using Visual Studio Code, you need to select the python interpreter 'conda activate rippl_AI_env'

If generating the enviroment from the .yml file fails, you can manually install the required packages, creating a new conda enviroment and following the next steps:
1. conda create -n rippl_AI_env python=3.9.15
1. conda install pip
2. pip install numpy
3. pip install tensorflow
4. pip install keras
5. pip install xgboost
6. pip install -U scikit-learn  (on ubuntu)
7. pip install imblearn
8. pip install matplotlib

If you want to download the lab data from figshare (not normalized, sampled with the original frequency of 30 000 Hz):
1. git clone https://github.com/cognoma/figshare.git
2. cd figshare
2. python setup.py

If you want to check how the data preprocessing works or implement your own:
1. pip install pandas
  
  https://www.cs.toronto.edu/~duvenaud/distill_bayes_net/public/