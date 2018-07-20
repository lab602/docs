# 识别小球并绘制追踪轨迹

GitHub:[Tracking_Ball](https://github.com/lab602/Tracking_Ball_with_Color)

```python
from collections import deque
from imutils.video import VideoStream
import numpy as np
import argparse
import cv2
import imutils
import time


ap = argparse.ArgumentParser()
ap.add_argument("-v", "--video",
	help="path to the (optional) video file")
ap.add_argument("-b", "--buffer", type=int, default=32,
	help="max buffer size")
args = vars(ap.parse_args())

greenLower = (11, 43, 46)
greenUpper = (25, 255, 255)

pts = deque(maxlen=args["buffer"])
counter = 0
(dX, dY) = (0, 0)
direction = ""

if not args.get("video", False):
	vs = VideoStream(src=0).start()


else:
	vs = cv2.VideoCapture(args["video"])

time.sleep(2.0)

while True:

	frame = vs.read()


	frame = frame[1] if args.get("video", False) else frame


	if frame is None:
		break


	frame = imutils.resize(frame, width=600)
	blurred = cv2.GaussianBlur(frame, (11, 11), 0)
	hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)


	mask = cv2.inRange(hsv, greenLower, greenUpper)
	mask = cv2.erode(mask, None, iterations=2)
	mask = cv2.dilate(mask, None, iterations=2)


	cnts = cv2.findContours(mask.copy(), cv2.RETR_EXTERNAL,
		cv2.CHAIN_APPROX_SIMPLE)
	cnts = cnts[0] if imutils.is_cv2() else cnts[1]
	center = None

	if len(cnts) > 0:

		c = max(cnts, key=cv2.contourArea)
		((x, y), radius) = cv2.minEnclosingCircle(c)
		M = cv2.moments(c)
		center = (int(M["m10"] / M["m00"]), int(M["m01"] / M["m00"]))


		if radius > 0:

			cv2.circle(frame, (int(x), int(y)), int(radius),
				(0, 255, 255), 2)
			cv2.circle(frame, center, 5, (0, 0, 255), -1)
			pts.appendleft(center)

	for i in np.arange(1, len(pts)):

		if pts[i - 1] is None or pts[i] is None:
			continue

		thickness = int(np.sqrt(args["buffer"] / float(i + 1)) * 2.5)
		cv2.line(frame, pts[i - 1], pts[i], (0, 0, 255), thickness)

	cv2.putText(frame, direction, (10, 30), cv2.FONT_HERSHEY_SIMPLEX,
		0.65, (0, 0, 255), 3)
	cv2.putText(frame, "dx: {}, dy: {}".format(dX, dY),
		(10, frame.shape[0] - 10), cv2.FONT_HERSHEY_SIMPLEX,
		0.35, (0, 0, 255), 1)


	cv2.imshow("Frame", frame)
	key = cv2.waitKey(1) & 0xFF
	counter += 1

	if key == ord("q"):
		break

if not args.get("video", False):
	vs.stop()

else:
	vs.release()

cv2.destroyAllWindows()
```
我们先导入必要的包。我们将使用deque，一种类似列表的数据结构。以记录我们在视频流中过去小球的位置坐标，这样我们就能在跟踪时绘制球的“轨迹”。

- buffer是deque的最大大小 ，它保留了我们正在跟踪的球的前一个（x，y）坐标列表 。这个队列   允许我们绘制球的“轨迹”，详细描述其过去的位置。较小的队列将导致较短的尾部，而较大的队列将产生较长的尾部（因为正在跟踪更多的点。

之后定义了HSV颜色空间中颜色的下边界和上边界。这里使用使用imutils库中的范围检测器脚本确定。这些颜色边界将允许我们检测视频文件中的小球。

我们对视频的每一帧进行了一些预处理 。首先，我们调整框架的大小，使其宽度为600px。缩小框架尺寸使我们能够更快地处理框架，从而导致FPS 增加（因为我们处理的图像数据较少）。然后我们将模糊框架以减少高频噪声，这样我们就可以更好关注于框架内的对象小球。最后，我们将帧转换   为HSV颜色空间。 

使用cv2.inRange确定来确定小球的位置。我们首先为颜色提供较低的HSV颜色边界，然后是较高的HSV边界。

计算图像中对象的轮廓，使用cv2.findContour函数。

进行检查以确保发现至少一个轮廓 。如果找到至少一个轮廓，我们在的cnts列表中找到最大的轮廓，计算最小包围圆，然后计算中心（x，y） -坐标（即“质心”）。

进行快速检查以确保最小封闭圆的半径足够大。如果半径通过测试，我们绘制两个圆：一个围绕球本身，另一个围绕球的质心。

依照质心的变化坐标连线绘制出运动轨迹。