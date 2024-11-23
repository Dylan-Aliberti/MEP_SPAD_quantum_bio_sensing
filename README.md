# MEP_SPAD_quantum_bio_sensing
All code developed for my master thesis project at the QIT / Ishihara lab in the TU Delft, titled Characterization of Single Photon Avalanche Diodes and Integration with Diamond for Quantum Bio-sensing.
This code is intended to be viewed after having read the corresponding thesis, and/or as a lab member who needs to use this. The report itself will be included in this repository soon.

## Overview
The code in this repository consists of mostly two parts.
- Python code for operating the confocal setup for 2D scans as well as a z-scan. The main files for this are "ODMR_2D.py" for the data acquisition and "ODMR_2D_process.py" for the processing of the data into plots and graphs. For z-scans, a different Python file is used called "z_scan.py". When setting up a measurement, the dictionary at the top of the file ODMR_2D.py needs to be changed with the right parameters. Alternatively, this script can read the settings from a json file when the filepath is provided, which has to be set within the code. When the measurement is done, the file will be saved as a Numpy (.npy) file as well as a json file containing the parameters and other meta-information. To process and view the data, the command "python3 ODMR_2D_process.py <path>" can be typed in the terminal, where <path> refers to the file path of the .npy file. The data acquisition and processing scripts are dependent on some other modules, namely point_in_triangle.py and double_dip_fitter.py.

- Code for operating the SPAD array. This consists of FPGA code, Arduino code, and Python code. Most of the SPAD array readout files also have a single SPAD version, which is meant to be used when connecting a single SPAD to the FPGA rather than the digital chip with serial connections. The Arduino code also has a test program that generates frames instead of acquiring them from the FPGA, so that the Python part of the whole system can be tested independently from the FPGA and SPAD chip. The code for SPAD array readout / single SPAD readout is very specific to the hardware used and may need changes when new hardware has to be tested. If you are interested in using this code for testing new SPAD hardware, feel free to contact me at dylan.aliberti@gmail.com.

## How to use the code 
This how-to-use section will only explain how to use part (a) of the files mention above, as part (b) only has a very specific use case and needs more detailed attention if the time to use them actually comes (in which case, you can contact me at dylan.aliberti@gmail.com, like mentioned above).

### General procedure for setting up a 2D scan and viewing the data
_This procedure assumes the diamond is already mounted on the piezo stage. Also, code from TNO is used for the alignemnt part, which is not included in this repository. The terminal commands that start with "rtcs" are from TNO. Also note that this procedure is dependent on both hardware and software. This procedure works with the setup the way it was at the end of the thesis (november 2024). When using this in the future, be wary of any changes._

After mounting, you make sure the piezo is put a bit backwards (in the -z direction) and firmly press the course stage close. Once you have that:
1. Type "rtcs gui"
2. Autozero all axes of the piezo stack.
3. Flip the pellico beamsplitter down, turn on camera and LED, and make sure the LED is set at 12 milli amps.
4. Find the focus by slowly moving closer (so in the +z direction). Typically the piezo stack is mounted such that the focus will be a few millimeters.
5. You will see the LED form a double square shape. This happens when you are approximately at the surface, or at the back surface.

#### Determining tilt
Do movements across x and y direction (not at the same time) by a few millimeters (or less if the diamond is not that big). and see how much you have to change z in order to get the LED square back. By dividing that by the amount you moved in x or y, you can calculate the tilt. Do this for both x and y. Later in the program you have to input these two values in units mm per mm.

#### Determining the exact surface focus
1. Turn the LED off and laser on.
2. Now focus the laser spot using the camera image and making fine (micron sized) adjustments in the z direction. Should differ about 12 micron from the double square LED shape.
3. (Optional) You can do a z-scan in the region where you think the diamond is, as long as you are careful not to accidentally press the piezo stage against the microwave antenna + objective. A good z-scan will show two maxima which are roughly at the top and back surface of the diamond, after which the count rate steeply drops to near zero. Sometimes those maxima are more spreadout, in which case the abovementioned laser focus using the camera will often disagree with the maxima found in this plot. The cause of this disagreement is not fully understood, so investigate this more if you want precise alignment. And if you find a better way, please let me know at dylan.aliberti@gmail.com.
4. Now you have the focus. Note down the full three coordinates of this point, as you will need to feed that to the scanning script.
5. Turn off the imager and raise the pellico beamsplitter.
 
#### Checking ODMR
You can check whether you actually get ODMR by typing "rtcs esr-scan --show --num-sweeps 1". You can change the parameters if needed. For the help page use "rtcs esr-scan --help"
 
#### Preparing the script
Now it's time to open file ODMR_2D.py. At the top of the file you will see a dictionary defined with a lot of parameters. The five numbers from the piezo scoping (2 tilts and 3 focus coordinates) need to be written in "ax, ay, x0, y0 and z0" respectively. Also take a look at all the other parameters such as number of steps, number of measurements per sweep, etc etc, and make sure "measurement_type" is set to the type of measurement you want to do, such as "ODMR".
 
#### Invoking the measurement
Once everything is prepared, go in the terminal to my folder, /home/dl-lab-pc3/Dylan, and type "python3 ODMR_2D.py".
To go to my folder you can type "cd /home/dl-lab-pc3/Dylan/", assuming you start the terminal in the home directory.
When the program starts, you should see a time estimate, and numbers indicating PL will appear after each sweep is done.

#### Viewing the acquired data
After the scanning script is done, a filepath is logged to which the data is stored, which leads to a Numpy (.npy) file. Open this file by typing "python3 ODMR_2D_process.py <path>" in the terminal, where <path> is the path to the data file. This script will first show you ten randomly selected pixels from the scan (if applicable), allowing you to judge whether the ODMR graphs and their fitting generally looks okay. After that, the plots of the 2D scans will be shown one by one. Everytime the program shows you a plot, you have to close the plot window in order for the next one to appear. Every plot is automatically saved when shown, containing the original file name as a prefix in its own file name. Congratulations, you now have plots of your (processed) data! You can upload them to OneNote, put them in presentations, or send them in Teams to your supervisor.

#### Changing plot parameters
If you want to change plotting parameters (such as title, font size, unit, etc.), please take a look inside the script of ODMR_2D_process.py. Look for the region where the plot_map function is called repeatedly (the place in the script is different depending on which type of measurement is loaded). This function accepts a lot of parameters, allowing you to change things. Commonly changed are clipping thresholds and the unit. When changing clipping, please make sure to edit the title entry correspondingly. And when changing the unit, please make sure to also edit the corresponding conversion rate, as the function does not automatically recognize the unit.
