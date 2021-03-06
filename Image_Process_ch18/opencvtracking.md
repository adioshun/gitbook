# Object Tracking using OpenCV (C++/Python)

OpenCV3.4에서 지원 하는 Tracking 알고리즘 (ref. pyimagesearch)




1. BOOSTING Tracker: Based on the same algorithm used to power the machine learning behind Haar cascades \(AdaBoost\), but like Haar cascades, is over a decade old. This tracker is slow and doesn’t work very well. Interesting only for legacy reasons and comparing other algorithms. \(minimum OpenCV 3.0.0\)

2. MIL Tracker: Better accuracy than BOOSTING tracker but does a poor job of reporting failure. \(minimum OpenCV 3.0.0\)

3. KCF Tracker: Kernelized Correlation Filters. Faster than BOOSTING and MIL. Similar to MIL and KCF, does not handle full occlusion well. \(minimum OpenCV 3.1.0\)

4. CSRT Tracker: Discriminative Correlation Filter \(with Channel and Spatial Reliability\). Tends to be more accurate than KCF but slightly slower. \(minimum OpenCV 3.4.2\)

5. MedianFlow Tracker: Does a nice job reporting failures; however, if there is too large of a jump in motion, such as fast moving objects, or objects that change quickly in their appearance, the model will fail. \(minimum OpenCV 3.0.0\)

6. TLD Tracker: I’m not sure if there is a problem with the OpenCV implementation of the TLD tracker or the actual algorithm itself, but the TLD tracker was incredibly prone to false-positives. I do not recommend using this OpenCV object tracker. \(minimum OpenCV 3.0.0\)

7. MOSSE Tracker: Very, very fast. Not as accurate as CSRT or KCF but a good choice if you need pure speed. \(minimum OpenCV 3.4.1\)

8. GOTURN Tracker: The only deep learning-based object detector included in OpenCV. It requires additional model files to run \(will not be covered in this post\). My initial experiments showed it was a bit of a pain to use even though it reportedly handles viewing changes well \(my initial experiments didn’t confirm this though\). I’ll try to cover it in a future post, but in the meantime, take a look at Satya’s writeup. \(minimum OpenCV 3.2.0\)

My personal suggestion is to:

* Use CSRT when you need higher object tracking accuracy and can tolerate slower FPS throughput

* Use KCF when you need faster FPS throughput but can handle slightly lower object tracking accuracy

* Use MOSSE when you need pure speed

> [OpenCV Object Tracking](https://www.pyimagesearch.com/2018/07/30/opencv-object-tracking/), [\[code\_SingleOT\]](https://gist.github.com/adioshun/779738c3e28151ffbb9dc7d2b13c2c0a), [\[code\_MOT\]](https://gist.github.com/adioshun/72106c82674fd6cd7b06fe9105c2ab86)


> 각 알고리즘에 대한 상세 설명은 [[여기]](https://www.learnopencv.com/object-tracking-using-opencv-cpp-python/)참고 

절차 
1. first set up the tracker by choosing a tracker type — BOOSTING, MIL, KCF, TLD, MEDIANFLOW, GOTURN, or MOSSE. 
2. We then open a video and grab a frame. 
3. We define a bounding box containing the object for the first frame and initialize the tracker with the first frame and the bounding box. 
4. Finally, we read frames from the video and just update the tracker in a loop to obtain a new bounding box for the current frame. 


```python
import cv2
import sys
 
(major_ver, minor_ver, subminor_ver) = (cv2.__version__).split('.')￼
 
if __name__ == '__main__' :
 
    # Set up tracker.
    # Instead of MIL, you can also use
 
    tracker_types = ['BOOSTING', 'MIL','KCF', 'TLD', 'MEDIANFLOW', 'GOTURN', 'MOSSE']
    tracker_type = tracker_types[2]
 
    if int(minor_ver) < 3:
        tracker = cv2.Tracker_create(tracker_type)
    else:
        if tracker_type == 'BOOSTING':
            tracker = cv2.TrackerBoosting_create()
        if tracker_type == 'MIL':
            tracker = cv2.TrackerMIL_create()
        if tracker_type == 'KCF':
            tracker = cv2.TrackerKCF_create()
        if tracker_type == 'TLD':
            tracker = cv2.TrackerTLD_create()
        if tracker_type == 'MEDIANFLOW':
            tracker = cv2.TrackerMedianFlow_create()
        if tracker_type == 'GOTURN':
            tracker = cv2.TrackerGOTURN_create()
        if tracker_type == 'MOSSE':
            tracker = cv2.TrackerMOSSE_create()
 
    # Read video
    video = cv2.VideoCapture("videos/chaplin.mp4")
 
    # Exit if video not opened.
    if not video.isOpened():
        print "Could not open video"
        sys.exit()
 
    # Read first frame.
    ok, frame = video.read()
    if not ok:
        print 'Cannot read video file'
        sys.exit()
     
    # Define an initial bounding box
    bbox = (287, 23, 86, 320)
 
    # Uncomment the line below to select a different bounding box
    bbox = cv2.selectROI(frame, False)
 
    # Initialize tracker with first frame and bounding box
    ok = tracker.init(frame, bbox)
 
    while True:
        # Read a new frame
        ok, frame = video.read()
        if not ok:
            break
         
        # Start timer
        timer = cv2.getTickCount()
 
        # Update tracker
        ok, bbox = tracker.update(frame)
 
        # Calculate Frames per second (FPS)
        fps = cv2.getTickFrequency() / (cv2.getTickCount() - timer);
 
        # Draw bounding box
        if ok:
            # Tracking success
            p1 = (int(bbox[0]), int(bbox[1]))
            p2 = (int(bbox[0] + bbox[2]), int(bbox[1] + bbox[3]))
            cv2.rectangle(frame, p1, p2, (255,0,0), 2, 1)
        else :
            # Tracking failure
            cv2.putText(frame, "Tracking failure detected", (100,80), cv2.FONT_HERSHEY_SIMPLEX, 0.75,(0,0,255),2)
 
        # Display tracker type on frame
        cv2.putText(frame, tracker_type + " Tracker", (100,20), cv2.FONT_HERSHEY_SIMPLEX, 0.75, (50,170,50),2);
     
        # Display FPS on frame
        cv2.putText(frame, "FPS : " + str(int(fps)), (100,50), cv2.FONT_HERSHEY_SIMPLEX, 0.75, (50,170,50), 2);
 
        # Display result
        cv2.imshow("Tracking", frame)
 
        # Exit if ESC pressed
        k = cv2.waitKey(1) & 0xff
        if k == 27 : break
```



```cpp
#include <opencv2/opencv.hpp>
#include <opencv2/tracking.hpp>
#include <opencv2/core/ocl.hpp>
 
using namespace cv;
using namespace std;
 
// Convert to string
#define SSTR( x ) static_cast< std::ostringstream & >( \
( std::ostringstream() << std::dec << x ) ).str()
 
int main(int argc, char **argv)
{
    // List of tracker types in OpenCV 3.4.1
    string trackerTypes[7] = {"BOOSTING", "MIL", "KCF", "TLD","MEDIANFLOW", "GOTURN", "MOSSE"};
    // vector <string> trackerTypes(types, std::end(types));
 
    // Create a tracker
    string trackerType = trackerTypes[2];
 
    Ptr<Tracker> tracker;
 
    #if (CV_MINOR_VERSION < 3)
    {
        tracker = Tracker::create(trackerType);
    }
    #else
    {
        if (trackerType == "BOOSTING")
            tracker = TrackerBoosting::create();
        if (trackerType == "MIL")
            tracker = TrackerMIL::create();
        if (trackerType == "KCF")
            tracker = TrackerKCF::create();
        if (trackerType == "TLD")
            tracker = TrackerTLD::create();
        if (trackerType == "MEDIANFLOW")
            tracker = TrackerMedianFlow::create();
        if (trackerType == "GOTURN")
            tracker = TrackerGOTURN::create();
        if (trackerType == "MOSSE")
            tracker = TrackerMOSSE::create();
    }
    #endif
    // Read video
    VideoCapture video("videos/chaplin.mp4");
     
    // Exit if video is not opened
    if(!video.isOpened())
    {
        cout << "Could not read video file" << endl; 
        return 1; 
    } 
 
    // Read first frame 
    Mat frame; 
    bool ok = video.read(frame); 
 
    // Define initial bounding box 
    Rect2d bbox(287, 23, 86, 320); 
 
    // Uncomment the line below to select a different bounding box 
    // bbox = selectROI(frame, false); 
    // Display bounding box. 
    rectangle(frame, bbox, Scalar( 255, 0, 0 ), 2, 1 ); 
 
    imshow("Tracking", frame); 
    tracker->init(frame, bbox);
     
    while(video.read(frame))
    {     
        // Start timer
        double timer = (double)getTickCount();
         
        // Update the tracking result
        bool ok = tracker->update(frame, bbox);
         
        // Calculate Frames per second (FPS)
        float fps = getTickFrequency() / ((double)getTickCount() - timer);
         
        if (ok)
        {
            // Tracking success : Draw the tracked object
            rectangle(frame, bbox, Scalar( 255, 0, 0 ), 2, 1 );
        }
        else
        {
            // Tracking failure detected.
            putText(frame, "Tracking failure detected", Point(100,80), FONT_HERSHEY_SIMPLEX, 0.75, Scalar(0,0,255),2);
        }
         
        // Display tracker type on frame
        putText(frame, trackerType + " Tracker", Point(100,20), FONT_HERSHEY_SIMPLEX, 0.75, Scalar(50,170,50),2);
         
        // Display FPS on frame
        putText(frame, "FPS : " + SSTR(int(fps)), Point(100,50), FONT_HERSHEY_SIMPLEX, 0.75, Scalar(50,170,50), 2);
 
        // Display frame.
        imshow("Tracking", frame);
         
        // Exit if ESC pressed.
        int k = waitKey(1);
        if(k == 27)
        {
            break;
        }
 
    }
}
```