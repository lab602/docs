# 根据颜色识别物体

GitHub:[Object_Detection_with_Color](https://github.com/lab602/Object_Detection_with_Color)

```c++
#include "stdafx.h"
#include <opencv2/imgproc/imgproc.hpp>
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <iostream>

using namespace cv;
using namespace std;

int main(int argc, char **argv)
{
	VideoCapture cap(0);
	if (!cap.isOpened())
	{
		return -1;
	}	

	int iLowH = 11; int iHighH = 25; int iLowS = 43; int iHighS = 255; int iLowV = 46; int iHighV = 255;

	while (true)
	{
		Mat imgOriginal;
		bool bSuccess = cap.read(imgOriginal);
		if (!bSuccess)
		{
			break;
		}

		Mat imgHSV;
		vector<Mat> hsvSplit;
		cvtColor(imgOriginal, imgHSV, COLOR_BGR2HSV); //因为我们读取的是彩色图像，直方均衡化需要再HSV空间里做
		split(imgHSV, hsvSplit);
		equalizeHist(hsvSplit[2], hsvSplit[2]);
		merge(hsvSplit, imgHSV);
		Mat imgThresholded;
		inRange(imgHSV, Scalar(iLowH, iLowS, iLowV), Scalar(iHighH, iHighS, iHighV), imgThresholded); //开操作（去除噪点）
		Mat element = getStructuringElement(MORPH_RECT, Size(5, 5));
		morphologyEx(imgThresholded, imgThresholded, MORPH_OPEN, element); //闭操作（连接一些连通域）
		morphologyEx(imgThresholded, imgThresholded, MORPH_CLOSE, element);

		//对灰度图进行滤波
		GaussianBlur(imgThresholded, imgThresholded, Size(3, 3), 0, 0);
		imshow("【滤波后的图像】", imgThresholded);
		Mat cannyImage;
		Canny(imgThresholded, cannyImage, 128, 255, 3);

		vector<vector<Point>> contours;
		vector<Vec4i> hierarchy;
		findContours(cannyImage, contours, hierarchy, RETR_TREE, CHAIN_APPROX_SIMPLE, Point(0, 0));

		//绘制轮廓

		for (int i = 0; i < (int)contours.size(); i++)
		{
			drawContours(cannyImage, contours, i, Scalar(255), 1, 8);
		}
		imshow("【处理后的图像】", cannyImage);

		for (int i = 0; i < contours.size(); i++)
		{
			//每个轮廓
			vector<Point> points = contours[i];
			//对给定的 2D点集，寻找最小面积的包围矩形
			RotatedRect box = minAreaRect(Mat(points));
			Point2f vertex[4];
			box.points(vertex);
			//绘制出最小面积的包围圈
			line(imgOriginal, vertex[0], vertex[1], Scalar(100, 200, 211), 6, LINE_AA);
			line(imgOriginal, vertex[1], vertex[2], Scalar(100, 200, 211), 6, LINE_AA);
			line(imgOriginal, vertex[2], vertex[3], Scalar(100, 200, 211), 6, LINE_AA);
			line(imgOriginal, vertex[3], vertex[0], Scalar(100, 200, 211), 6, LINE_AA);

			//绘制中心的光标
			Point s1, l, r, u, d;
			s1.x = (vertex[0].x + vertex[2].x) / 2;
			s1.y = (vertex[0].y + vertex[2].y) / 2;
			l.x = s1.x - 10;
			l.y = s1.y;

			r.x = s1.x + 10;
			r.y = s1.y;

			u.x = s1.x;
			u.y = s1.y - 10;

			d.x = s1.x;
			d.y = s1.y + 10;

			line(imgOriginal, l, r, Scalar(100, 200, 211), 2, LINE_AA);
			line(imgOriginal, u, d, Scalar(100, 200, 211), 2, LINE_AA);

			cout << "Ball\n" << "(" << d.x<<" , " << d.y << ")" << endl;

		}
		namedWindow("result", WINDOW_NORMAL);

		imshow("result", imgOriginal);

		char key = (char)waitKey(30);
		if (key == 27) break;
	}


	return 0;
}
```