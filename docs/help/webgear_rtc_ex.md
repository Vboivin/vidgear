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

# WebGear_RTC_RTC Examples

&nbsp;

## Using WebGear_RTC with RaspberryPi Camera Module

Because of WebGear_RTC API's flexible internal wapper around VideoGear, it can easily access any parameter of CamGear and PiGear videocapture APIs.

!!! info "Following usage examples are just an idea of what can be done with WebGear_RTC API, you can try various [VideoGear](../../gears/videogear/params/), [CamGear](../../gears/camgear/params/) and [PiGear](../../gears/pigear/params/) parameters directly in WebGear_RTC API in the similar manner."
 
Here's a bare-minimum example of using WebGear_RTC API with the Raspberry Pi camera module while tweaking its various properties in just one-liner:

```python
# import libs
import uvicorn
from vidgear.gears.asyncio import WebGear_RTC

# various webgear_rtc performance and Raspberry Pi camera tweaks
options = {
    "frame_size_reduction": 25,
    "hflip": True,
    "exposure_mode": "auto",
    "iso": 800,
    "exposure_compensation": 15,
    "awb_mode": "horizon",
    "sensor_mode": 0,
}

# initialize WebGear_RTC app
web = WebGear_RTC(
    enablePiCamera=True, resolution=(640, 480), framerate=60, logging=True, **options
)

# run this app on Uvicorn server at address http://localhost:8000/
uvicorn.run(web(), host="localhost", port=8000)

# close app safely
web.shutdown()
```

&nbsp;

## Using WebGear_RTC with real-time Video Stabilization enabled
 
Here's an example of using WebGear_RTC API with real-time Video Stabilization enabled:

```python
# import libs
import uvicorn
from vidgear.gears.asyncio import WebGear_RTC

# various webgear_rtc performance tweaks
options = {
    "frame_size_reduction": 25,
}

# initialize WebGear_RTC app  with a raw source and enable video stabilization(`stabilize=True`)
web = WebGear_RTC(source="foo.mp4", stabilize=True, logging=True, **options)

# run this app on Uvicorn server at address http://localhost:8000/
uvicorn.run(web(), host="localhost", port=8000)

# close app safely
web.shutdown()
```

&nbsp;

## Display Two Sources Simultaneously in WebGear_RTC

In this example, we'll be displaying two video feeds side-by-side simultaneously on browser using WebGear_RTC API by simply concatenating frames in real-time: 

!!! new "New in v0.2.2" 
    This example was added in `v0.2.2`.

```python
# import necessary libs
import uvicorn, asyncio, cv2
import numpy as np
from av import VideoFrame
from aiortc import VideoStreamTrack
from vidgear.gears.asyncio import WebGear_RTC
from vidgear.gears.asyncio.helper import reducer

# initialize WebGear_RTC app without any source
web = WebGear_RTC(logging=True)

# frame concatenator
def get_conc_frame(frame1, frame2):
    h1, w1 = frame1.shape[:2]
    h2, w2 = frame2.shape[:2]

    # create empty matrix
    vis = np.zeros((max(h1, h2), w1 + w2, 3), np.uint8)

    # combine 2 frames
    vis[:h1, :w1, :3] = frame1
    vis[:h2, w1 : w1 + w2, :3] = frame2

    return vis


# create your own Bare-Minimum Custom Media Server
class Custom_RTCServer(VideoStreamTrack):
    """
    Custom Media Server using OpenCV, an inherit-class
    to aiortc's VideoStreamTrack.
    """

    def __init__(self, source1=None, source2=None):

        # don't forget this line!
        super().__init__()

        # check is source are provided
        if source1 is None or source2 is None:
            raise ValueError("Provide both source")

        # initialize global params
        # define both source here
        self.stream1 = cv2.VideoCapture(source1)
        self.stream2 = cv2.VideoCapture(source2)

    async def recv(self):
        """
        A coroutine function that yields `av.frame.Frame`.
        """
        # don't forget this function!!!

        # get next timestamp
        pts, time_base = await self.next_timestamp()

        # read video frame
        (grabbed1, frame1) = self.stream1.read()
        (grabbed2, frame2) = self.stream2.read()

        # if NoneType
        if not grabbed1 or not grabbed2:
            return None
        else:
            print("Got frames")

        print(frame1.shape)
        print(frame2.shape)

        # concatenate frame
        frame = get_conc_frame(frame1, frame2)

        print(frame.shape)

        # reducer frames size if you want more performance otherwise comment this line
        # frame = await reducer(frame, percentage=30)  # reduce frame by 30%

        # contruct `av.frame.Frame` from `numpy.nd.array`
        av_frame = VideoFrame.from_ndarray(frame, format="bgr24")
        av_frame.pts = pts
        av_frame.time_base = time_base

        # return `av.frame.Frame`
        return av_frame

    def terminate(self):
        """
        Gracefully terminates VideoGear stream
        """
        # don't forget this function!!!

        # terminate
        if not (self.stream1 is None):
            self.stream1.release()
            self.stream1 = None

        if not (self.stream2 is None):
            self.stream2.release()
            self.stream2 = None


# assign your custom media server to config with both adequate sources (for e.g. foo1.mp4 and foo2.mp4)
web.config["server"] = Custom_RTCServer(
    source1="dance_videos/foo1.mp4", source2="dance_videos/foo2.mp4"
)

# run this app on Uvicorn server at address http://localhost:8000/
uvicorn.run(web(), host="localhost", port=8000)

# close app safely
web.shutdown()
``` 

!!! success "On successfully running this code, the output stream will be displayed at address http://localhost:8000/ in Browser."


&nbsp;