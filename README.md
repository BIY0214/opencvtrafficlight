#include <opencv2/opencv.hpp>

using namespace cv;

const Scalar RED_LOWER(0, 100, 100);
const Scalar RED_UPPER(10, 255, 255);
const Scalar GREEN_LOWER(40, 50, 50);
const Scalar GREEN_UPPER(90, 255, 255);

string detectTrafficLight(Mat& frame, Mat& hsv) {
    Mat redMask, greenMask;
    inRange(hsv, RED_LOWER, RED_UPPER, redMask);
    inRange(hsv, GREEN_LOWER, GREEN_UPPER, greenMask);

    int redPixels = countNonZero(redMask);
    int greenPixels = countNonZero(greenMask);

    if (redPixels > greenPixels * 3) { // 调整阈值比例
        return "RED";
    } else if (greenPixels > redPixels * 3) {
        return "GREEN";
    } else {
        return "NOT";
    }
}

int main(int argc, char** argv) {
    string videoFilePath = "path_to_your_video.avi"; // 替换为您的视频文件路径
    VideoCapture cap(videoFilePath);

    if (!cap.isOpened()) {
        return -1;
    }

    Mat frame, hsv;
    string mode = "NOT DETECTED";

    while (true) {
        cap >> frame;
        if (frame.empty()) break;

        cvtColor(frame, hsv, COLOR_BGR2HSV);

        mode = detectTrafficLight(frame, hsv);

        putText(frame, mode, Point(10, 30), FONT_HERSHEY_SIMPLEX, 1, Scalar(255, 0, 0), 2);

        if (mode == "RED") {
            Mat redMask;
            inRange(hsv, RED_LOWER, RED_UPPER, redMask);
            vector<vector<Point>> contours;
            findContours(redMask, contours, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);
            for (size_t i = 0; i < contours.size(); i++) {
                Rect boundingRect = boundingRect(contours[i]);
                rectangle(frame, boundingRect, Scalar(0, 0, 255), 2);
            }
        }

        imshow("Traffic Light Detection", frame);

        if (waitKey(30) == 27) break; // 按 Esc 键退出
    }

    cap.release();
    destroyAllWindows();

    return 0;
}
