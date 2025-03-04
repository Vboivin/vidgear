<!--
===============================================
vidgear library source-code is deployed under the Apache 2.0 License:

Copyright (c) 2019 Abhishek Thakur(@abhiTronix) <abhi.una12@gmail.com>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
===============================================
-->

# WriteGear API Usage Examples: Non-Compression Mode


!!! warning "Important Information"

    * **DO NOT** feed frames to WriteGear with different dimensions or channels, or-else WriteGear API will exit with `ValueError`.

    * ==In case WriteGear API fails to detect valid FFmpeg executables on your system, it will auto-switches to this(Non-Compression) Mode.==

    * Always use `writer.close()` at the very end of the main code. **NEVER USE IT INBETWEEN CODE** to avoid undesired behavior.


&thinsp;


## Bare-Minimum Usage

Following is the bare-minimum code you need to get started with WriteGear API in Non-Compression Mode:

```python
# import required libraries
from vidgear.gears import CamGear
from vidgear.gears import WriteGear
import cv2

# open any valid video stream(for e.g `myvideo.avi` file)
stream = CamGear(source="myvideo.avi").start()

# Define writer with Non-compression mode and suitable output filename for e.g. `Output.mp4`
writer = WriteGear(output_filename="Output.mp4", compression_mode=False)

# loop over
while True:

    # read frames from stream
    frame = stream.read()

    # check for frame if Nonetype
    if frame is None:
        break

    # {do something with the frame here}

    # write frame to writer
    writer.write(frame)

    # Show output window
    cv2.imshow("Output Frame", frame)

    # check for 'q' key if pressed
    key = cv2.waitKey(1) & 0xFF
    if key == ord("q"):
        break

# close output window
cv2.destroyAllWindows()

# safely close video stream
stream.stop()

# safely close writer
writer.close()
```

&nbsp; 

## Using Non-Compression Mode with VideoCapture Gears


In Non-Compression mode, WriteGear API provides flexible control over [**OpenCV's VideoWriter API**](https://docs.opencv.org/master/dd/d9e/classcv_1_1VideoWriter.html#ad59c61d8881ba2b2da22cff5487465b5) parameters through its `output_param` dictionary parameter by formating them as dictionary attributes. Also, WriteGear API can be used in conjunction with any other Gear effortlessly. The complete usage example is as follows:

```python
# import required libraries
from vidgear.gears import VideoGear
from vidgear.gears import WriteGear
import cv2

# define suitable tweak parameters for writer
output_params = {"-fourcc": "MJPG", "-fps": 30}

# open live video stream on webcam at first index(i.e. 0) device
stream = VideoGear(source=0, logging=True).start()

# Define writer with defined parameters and suitable output filename for e.g. `Output.mp4`
writer = WriteGear(
    output_filename="Output.mp4", compression_mode=False, logging=True, **output_params
)


# loop over
while True:

    # read frames from stream
    frame = stream.read()

    # check for frame if Nonetype
    if frame is None:
        break

    # {do something with the frame here}
    # lets convert frame to gray for this example
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    # write gray frame to writer
    writer.write(gray)

    # Show output window
    cv2.imshow("Output Gray Frame", gray)

    # check for 'q' key if pressed
    key = cv2.waitKey(1) & 0xFF
    if key == ord("q"):
        break

# close output window
cv2.destroyAllWindows()

# safely close video stream
stream.stop()

# safely close writer
writer.close()
```


&nbsp;

## Using Non-Compression Mode with OpenCV

You can easily use WriterGear API directly with any Video Processing library(_For e.g OpenCV itself_) in Non-Compression Mode. The complete usage example is as follows:

```python
# import required libraries
from vidgear.gears import WriteGear
import cv2

# define suitable tweak parameters for writer
output_params = {"-fourcc": "MJPG", "-fps": 30}

# Open suitable video stream, such as webcam on first index(i.e. 0)
stream = cv2.VideoCapture(0)

# Define writer with defined parameters and suitable output filename for e.g. `Output.mp4`
writer = WriteGear(
    output_filename="Output.mp4", compression_mode=False, logging=True, **output_params
)

# loop over
while True:

    # read frames from stream
    (grabbed, frame) = stream.read()

    # check for frame if not grabbed
    if not grabbed:
        break

    # {do something with the frame here}
    # lets convert frame to gray for this example
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    # write gray frame to writer
    writer.write(gray)

    # Show output window
    cv2.imshow("Output Gray Frame", gray)

    # check for 'q' key if pressed
    key = cv2.waitKey(1) & 0xFF
    if key == ord("q"):
        break

# close output window
cv2.destroyAllWindows()

# safely close video stream
stream.release()

# safely close writer
writer.close()
```

&nbsp; 