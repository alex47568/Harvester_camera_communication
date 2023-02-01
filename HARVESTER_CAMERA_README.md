#Tutorial computer vision project 

this tutorial is meant to guide the researcher through the acquired knowledge and methods
used through out the project process.

##Camera communication with Harvester
It is recommended to follow along with anaconda as the main interpeter for python.
Further the error sheet is also being encouraged to use for bug solving.

### Anaconda interpeter
First step is to create a new anaconda environment(interpeter).
For those who didn't use anaconda before you can get it through this link:
Link to anaconda distribution package: https://www.anaconda.com/products/distribution/start-coding-immediately

First step is to create a new environment by using the create function on anaconda.
Give it a custom name and chose as package python 3.9.15 

When the environment is created the next packages are required for the camera to work
with Harvester. You can open a terminal of the new created environment by clicking on
the play button and then terminal. Make sure that you upgrade pip before installing the packages with

terminal input: pip install --upgrade pip

The following packages are required :

pip install harvesters <br>
pip install opencv-contrib-python  

Note: If you need a gui which is not needed for this project. 
you can follow the tutorial of harvester gui linked bellow.

link harvester gui: https://github.com/genicam/harvesters_gui

###adding interpreter to pycharm
The next step is to start a new python project on your pc and give it a custom name.
After that the anaconda interpeter needs to be added to the project. 
If you use pycharm the next steps are taken to add the interpeter.

- use: ctrl+alt+s to open settings menu
- Go to (your project name) tab and klick on "python interpeter"
- Then press on "show all"


![img.png](./img.png)

- the next step is to klick on the + sign right above to add a new interpeter. <br>
- next step is to go to Conda environment and klick on existing environment. <br>
- After that at interpreter klick on the three dots and chose the interpreter that 
was created and chose the python.exe file <br>


![img_2.png](./img_2.png)

###Harvester initialization 
After the interpeter is added and loaded in it is time to try some coding.

first we start with importing nessecary packages which are

```
# import necessary packages
from harvesters.core import Harvester
from harvesters.util.pfnc import bayer_location_formats
from harvesters.util.pfnc import mono_location_formats, \
    rgb_formats, bgr_formats, \
    rgba_formats, bgra_formats
import cv2 as cv 
```

When the import packages work it is nessecary to instal the gentl producer file for the camera <br>

For the matrix vision camera you can install the gentl.cti file from here. <br>
link to matrix vision .exe file: https://www.matrix-vision.com/en/downloads/drivers-software/mvbluecougar-gigabit-ethernet-dual-gigabit-ethernet-10gige-ethernet/windows-7-8-1-10

- chose the mvGenTL_Acquire-x86_64-2.48.0.exe download link

link to allied vision camera .exe file: https://www.alliedvision.com/en/support/software-downloads/

- chose the Vimba_v6.0_(operation system) file

After the download and following the setup we can come back to the python script and write the initialization phase.
<br>

```
# initializing harvester and name it h
h = Harvester()

# adding a CTI object to the harvester with the add_file function note: you can add as many as you want
h.add_file('C:/Program Files/MATRIX VISION/mvIMPACT Acquire/bin/x64/mvGenTLProducer.cti', True, True)
# h.add_file('C:/Program Files/Allied Vision/Vimba_6.0/VimbaCLConfigTL/Bin/Win64/VimbaCLConfigTL.cti')

# #.files method will let you know the loaded in CTI objects
print(h.files)

# #removes a specific CTI file
# h.remove_file('C:/Program Files/Allied Vision/Vimba_6.0/VimbaCLConfigTL/Bin/Win64/VimbaCLConfigTL.cti')

# this will update the list of added devices
h.update()
print(h)

# #here you can see the added devices and their info
print(h.device_info_list)

# this function will try to create a imageacquirer for the first candidate device
# note: the image requirer doesn't acquire/receive images but it transmits them
# if more camera are assigned h2.create(1) for second camera etc
# look with print(h.device_info_list) which device will be assigned
ia = h.create(0)

# Configure the target(images) device width and height;
ia.remote_device.node_map.Width.value = 1280
ia.remote_device.node_map.Height.value = 960

# this function will give the available property names
# print(DeviceInfo.search_keys)

# starting the image acquisition
ia.start()
```
this script can be copy/pasted in the file
First we start harvester and name it h as shown in the script

The second step is to add the cti file to harvester this function ```h.add.file()``` is the communication
between the camera and harvester. That why it is important that the path to this .cti file is correct.
The .cti file should be located in the program files directory unless the path was manual changed by the researcher.

Example path to matrix vision cti file: <br>
path code: ``` h.add_file('C:/Program Files/MATRIX VISION/mvIMPACT Acquire/bin/x64/mvGenTLProducer.cti', True, True)``` 

The ```True, True``` at the end of the function is for enabling errorlist for if something is bugged or not correct.

The rest of the code is documented with comments and recommended to scan through.
After harvester is started with ```ia.start()```. It is time to acquire images with the next following code. 

Before continuing lets test run this script.
connect the camera to your device through ethernet port and the power supply to the camera. <br>
when you run the script the first time windows firewall will pop up
and ask for permission. <br>
<br>
Note: when you run the script the first time it can be that you get an index error it can be that
the camera is not loaded in yet. simply wait a gew seconds and retry to run the script this should solve the issue.

<br>

when the script runs the following output should return
![img_3.png](./img_3.png)
<br>
if this is the output then the code works fine so far if not try the error sheet to resolve the issue.
next step is to start acquiring images

###image/video acquisition harvester
```# with while True: you can display images as video
while True:
    # fetching images from camera as buffer
    with ia.fetch() as buffer:
        # print(buffer)
        payload = buffer.payload
        component = payload.components[0]
        width = component.width
        height = component.height
        data_format = component.data_format

        # Reshape the image so that it can be drawn on the VisPy canvas:
        if data_format in mono_location_formats:
            content = component.data.reshape(height, width)
        
        else:
            # The image requires you to reshape it to draw it on the
            # canvas:
            if data_format in rgb_formats or \
                    data_format in rgba_formats or \
                    data_format in bgr_formats or \
                    data_format in bgra_formats:
                #
                content = component.data.reshape(
                    height, width,
                    int(component.num_components_per_pixel)  # Set of R, G, B, and Alpha
                )
                #
                if data_format in bgr_formats:
                    # Swap every R and B:
                    content = content[:, :, ::-1]
            
            # reshaping and changing bayer format to BGR added code to the original
            elif data_format in bayer_location_formats:
                content = component.data.reshape(height, width)
                content = cv.cvtColor(content, cv.COLOR_BAYER_RG2RGB)

            else:
                print("no aqcuired pixel format check for pixel format with print(buffer)")

```

The above line ```while True:``` makes it so it can turn the captured images into a playing video.
If you want simply to gain certain amount of images you can replace ```while True:```with: <br>
```
i = 0 
while i =< number of images to take:
```

This lines of code will take the appropriate format to display the images in. 
Note: This is not the original code from harvester but edited code so it can also convert bayer format to rgb format.
these lines of code should remain in the code for it to work since matrix vision camera gives bayer format back. 

After the images are acquired it is time to enhance the images/video and show it on the opencv function

````
# resizing the video
            scale_percent = 100  # percent of original size
            width = int(content.shape[1] * scale_percent / 100)
            height = int(content.shape[0] * scale_percent / 100)
            dim = (width, height)

            resized = cv.resize(content, dim, interpolation=cv.INTER_AREA)
            
            #converting BGR to RGB format
            bgr = cv.cvtColor(resized, cv.COLOR_BGR2RGB)
            
            # showing the captured images/video
            cv.imshow('Camera Windows', resized)

            # saving the images on a file location
            #cv.imwrite("C:\\Users\\alex4\\Documents\\computervision\\robotlab_computervision\\train_pictures_left_side\\picture" + str(img_number) + ".jpg", resized)

            ##for the researchers who wants to save images
            #img_number += 1
            
            ## for the researchers that wanted to acquire images instead of video
            #i += 1
            
            ## the amount of time taken between every picture taken. for researchers who wants to acquire images
            # time.sleep(1)

            # when f is pressed destroy all winodws, image aqcuisition stops and resets harvesters
            if  cv.waitKey(20) & 0xFF == ord('f'):
                cv.destroyAllWindows()
                # to stop the image acquisition
                ia.stop()

                # this function will disconnect the connecting devices from the image acquirer
                ia.destroy()

                # when you're done working with the harvester call this function
                h.reset()

````

Harvester gives as an output a bgr formatted image under the variable name ```content``` has it is shown in the code above.  
<br>
From there I added a resize function which will be handy for researchers who use it to acquire images.
Because when you get an image from harvester it might get to big or to small this way you can change the size to a desired format.

###Note for image acquired instead of video
change the line of code from ```if  cv.waitKey(20) & 0xFF == ord('f'):``` to <br>
``` if i = (your prefered amount of pictures taken) or cv.waitKey(20) & 0xFF == ord('f'): ```

Note: At last make sure whenever you stop with the script to stop the image acquisition with ```h.reset()```
if errors occur I recommend to use the error sheet at the attachment of this project.
More information about the harvester can be found on the following link.
Link harvester page: https://github.com/genicam/harvesters

When this code is run a window should show up with the acquired video. 
For researchers who acquire images they should be saved first in a map.
Use the saving images code to achieve that goal.

