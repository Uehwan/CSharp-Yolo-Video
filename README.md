# CSharp-Yolo-Video
Although [the C# wrapper for Darknet](https://github.com/AlturosDestinations/Alturos.Yolo) exists, I went through a hard time figuring out how to apply the wrapper for videos. For later use for myself and saving others' time, I summarize how to apply the Yolo wrapper on videos.

## Getting Started
The following instructions will lead you to setting the environment for using the Yolo wrapper in your project.

0. System requriements
- .NET Framework 4.6.1
- [Microsoft Visual C++ Redistributable for Visual Studio 2017 x64](https://aka.ms/vs/16/release/vc_redist.x64.exe)

1. Install Alturos.Yolo using [NuGet](https://www.nuget.org/packages/Alturos.Yolo) as follows or you can use the NuGet GUI instead. If you're using NuGet GUI, search for "Alturos.Yolo".
```
PM> install-package Alturos.Yolo (C# wrapper and C++ dlls 28MB)
PM> install-package Alturos.YoloV2TinyVocData (YOLOv2-tiny Pre-Trained Dataset 56MB)
```

(Optional) For GPU support, install and download the followings.
1) Install the latest Nvidia driver for your graphic device.
2) [Install Nvidia CUDA Toolkit 10.1](https://developer.nvidia.com/cuda-downloads) (must be installed add a hardware driver for cuda support)
3) [Download Nvidia cuDNN v7.6.3 for CUDA 10.1](https://developer.nvidia.com/rdp/cudnn-download)
4) Copy the `cudnn64_7.dll` from the output directory of cdDNN v7.6.3. into the `x64` folder of your project.

2. Install [OpenCvSharp3-AnyCPU](https://github.com/shimat/opencvsharp) over NuGet as follows or search for "OpenCvSharp3-AnyCPU". Although the package name contains CvSharp3, it is actually an OpenCv 4.x wrapper.
```
PM> install-package OpenCvSharp3-AnyCPU
```

3. Download pretrained weights and place it in your project directory. For more information, visit [Alturos.Yolo](https://github.com/AlturosDestinations/Alturos.Yolo/blob/master/README.md#pre-trained-dataset)

Model | Processing Resolution | Cfg | Weights | Names |
--- | --- | --- | --- | --- |
YOLOv3 | 608x608 | [yolov3.cfg](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/yolov3.cfg) | [yolov3.weights](https://pjreddie.com/media/files/yolov3.weights) | [coco.names](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/coco.names) |
YOLOv3-tiny | 416x416 | [yolov3-tiny.cfg](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/yolov3-tiny.cfg) | [yolov3-tiny.weights](https://pjreddie.com/media/files/yolov3-tiny.weights) | [coco.names](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/coco.names) |
YOLOv2 | 608x608 | [yolov2.cfg](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/yolov2.cfg) | [yolov2.weights](https://pjreddie.com/media/files/yolov2.weights) | [coco.names](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/coco.names) |
YOLOv2-tiny | 416x416 | [yolov2-tiny.cfg](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/yolov2-tiny.cfg) | [yolov2-tiny.weights](https://pjreddie.com/media/files/yolov2-tiny.weights) | [voc.names](https://raw.githubusercontent.com/pjreddie/darknet/master/data/voc.names) |
yolo9000 | 448x448 | [yolo9000.cfg](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/yolo9000.cfg) | [yolo9000.weights](https://github.com/philipperemy/yolo-9000/tree/master/yolo9000-weights) | [9k.names](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/9k.names) |

## Write Codes for Video Object Recognition
The following is the minimum code for running the Yolo wrapper on a video file. For running the code, set the solution platform as "x64"!
```cs
using OpenCvSharp;
using OpenCvSharp.Extensions;

using Alturos.Yolo;

private void VideoObjectDetection()
{
  // YOLO setting
  int yoloWidth = 608, yoloHeight = 608;
  var configurationDetector = new ConfigurationDetector();
  var config = configurationDetector.Detect();
  YoloWrapper yoloWrapper = new YoloWrapper(config);
  
  // OpenCV & WPF setting
  VideoCapture videocapture;
  Mat image = new Mat();
  WriteableBitmap wb = new WriteableBitmap(yoloWidth, yoloHeight, 96, 96, PixelFormats.Bgr24, null);
  
  byte[] imageInBytes = new byte[(int)(yoloWidth * yoloHeight * image.Channels())];
  
  // Read a video file and run object detection over it!
  using (videocapture = new VideoCapture(address))
  {
    using(Mat imageOriginal = new Mat())
    {
      // read a single frame and convert the frame into a byte array
      videocapture.Read(imageOriginal);
      image = imageOriginal.Resize(new OpenCvSharp.Size(yoloWidth, yoloHeight));
      imageInBytes = image.ToBytes();
      
      // conduct object detection and display the result
      var items = yolowrapper.Detect(imageInBytes);
      foreach(var item in items)
      {
        var x = item.X;
        var y = item.Y;
        var width = item.Width;
        var height = item.Height;
        var type = item.Type;  // class name of the object
        
        // draw a bounding box for the detected object
        // you can set different colors for different classes
        Cv2.Rectangle(image, new OpenCvSharp.Rect(x, y, width, height), Scalar.Green, 3);
      }
      WriteableBitmapConverter.ToWriteableBitmap(image, wb);
      /* WPF component: videoViewer
      <Canvas Name="canvasYoloVideo" Height="608" Width="608">
        <Image Name="videoViewer" Height="608" Width="608" Stretch="Fill" />
      </Canvas>
      */
      videoViewer.Source = wb;
    }
  }
}
```
