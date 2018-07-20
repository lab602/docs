# 跟踪视频中的运动小球

GitHub:[Tracking Balls in Video](https://github.com/lab602/Tracking_Balls_with_Camshift)

* Feature_caculation.py

```python
import cv2
import numpy as np
try:
    import Cameshift_with_demo.GetRoi as GetRoi
except:
    import GetRoi as GetRoi


def HSV_frame_H_Cal(frame):
    """
    HSV_frame_H_Cal函数 提取通道H
    :param frame: 输入一帧图像，该帧图像是HSV图像
    :return:
    """
    return cv2.split(frame)[0]

def Hist_cal(frame, ix, iy, w, h):
    """
    Hist_cal 计算目标区域的直方图特征，仅提取其中的H通道进行直方图计算
    :param frame:
    :param ix: 初始找到的目标位置，左上角x坐标
    :param iy: 初始找到的目标位置，左上角y坐标
    :param w:  初始找到的目标的宽度
    :param h:  初始找到的目标的高度
    :return:
    """
    H = HSV_frame_H_Cal(frame)  # 提取H通道
    frame_hist = np.histogram(H[ix:ix+w, iy:iy+h], bins=12, range=[0, 180])   # 计算目标区域的H通道直方图特征

    return frame_hist, np.max(frame_hist[0]), ix, iy, w, h

def Back_projection_cal(frame_H, frame_hist, q_max):
    """
    Back_projection_cal 反投影计算函数，将图片像素反投影为特征图
    :param frame_H: 输入的一帧图像
    :param frame_hist: 目标区域的直方图特征
    :param q_max:  目标区域中直方图特征的最大值
    :return:
    """
    back_projection_img = np.zeros(frame_H.shape, dtype=np.uint8)
    for i in range(0, 180, 15):  # 利用numpy的特性高效计算反投影图，不用Python的for循环，提高效率
        back_projection_img[(frame_H >= i) & (frame_H < i + 15)] = frame_hist[0][(frame_hist[1] == i)[:12]] * 255 // q_max
    return back_projection_img


def Centroid_Mass_Cal(back_projection_img, ix, iy, w, h):
    """
    Centroid_Mass_Cal 目标质心，目标面积计算,
    :param back_projection_img: 输入一张特征图
    :param ix: 目标区域当前的左上角x坐标
    :param iy: 目标区域当前的右上角y坐标
    :param w: 目标区域当前的宽度
    :param h: 目标区域当前的高度
    :return:
    """
    M00 = np.sum(back_projection_img[ix:ix+w, iy:iy+h])  # 计算一阶矩
    xx = np.array([[x] for x in range(ix, ix + w)])
    M10 = np.sum(xx * back_projection_img[ix:ix + w, iy:iy + h])  # 利用numpy的广播操作计算二阶矩，提高效率
    yy = np.array([y for y in range(iy, iy + h)])
    M01 = np.sum(yy * back_projection_img[ix:ix + w, iy:iy + h])  # 利用numpy的广播操作计算二阶矩，提高效率
    xc = M10 // M00  # 新目标中心的x坐标
    yc = M01 // M00  # 新目标中心的y坐标
    S = 2 * np.sqrt(M00)  # 获取目标的面积大小

    return xc, yc, S


if __name__ == '__main__':
    frames, frames_gray, frames_color = GetRoi.GetFrame('../Ball.avi')
    H = HSV_frame_H_Cal(frames[10])
    frame_hist, q_max, ix, iy, w, h = Hist_cal(frames[0])
    back_projection_img = Back_projection_cal(H, frame_hist, q_max)
    cv2.imshow('int', frames_color[1][ix:ix + w, iy:iy + h])
    cv2.imshow('gg', frames[10])
    cv2.imshow('initint', back_projection_img)
    cv2.waitKey(0)

    print(Centroid_Mass_Cal(back_projection_img, ix, iy, w, h))
    pass
```

* GetRoi.py

```python
import cv2
import imutils


ix, iy = -1, -1
w, h = -1, -1
count = 0


def GetFrame(video):
    """
    GetFrame 做视频帧读取，同时转换视频帧为HSV图
    :param video: 视频文件路径
    :return:
    """
    frames_hsv = []
    frames_gray = []
    frames_color = []
    camera = cv2.VideoCapture(video)  # 读取视频
    count = 1
    while True:
        res, frame = camera.read()
        if not res:
            print(count)
            break
        frame = imutils.resize(frame, width=500)
        frames_color.append(frame)
        count = count + 1
        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)    # 转HSV图
        hsv[hsv[:, :, 1] > 150] = 0   # 重点小技巧，结合了图片的S通道特征，出去和中间小球颜色特征相似的跑道，从而是中间小球跟踪的关键步骤
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)  # 转GRAY图

        frames_hsv.append(hsv)
        frames_gray.append(gray)
    print('done_get_frame')
    camera.release()
    return frames_hsv, frames_gray, frames_color

def Get_ix_iy_w_h(event, x, y, flags, param):
    global ix, iy, w, h, count
    if event == cv2.EVENT_LBUTTONDOWN:
        if count == 0:
            ix, iy = x, y
            count = count + 1
        else:
            w = x - ix
            h = y - iy
            count = 0

def getRoi(frame):
    """
    getRoi 获取目标的位置，大小，使用方法是运行该程序，然后在目标的左上角和右下角各点击一下，然后键盘输入q，即在输出信息中获取target的位置信息
    :param frame: 输入一帧HSV图
    :return:
    """
    global ix, iy, w, h
    cv2.namedWindow('image')
    cv2.setMouseCallback('image', Get_ix_iy_w_h)
    while True:
        cv2.imshow('image', frame)
        k = cv2.waitKey(1) & 0xFF
        if k == ord('q'):
            break
    return iy, ix, h, w  # iy才是真正的ix，ix是iy，同时有h为实际的高，w为实际的宽

if __name__ == '__main__':
    frames, frames_gray, frames_color = GetFrame('../Ball.avi')
    getRoi(frames[0])
    print(frames[0].shape)

    print('target:', iy, ix, w, h)
    cv2.rectangle(frames[1], (ix, iy), (ix + w, iy + h), (255, 0, 0), 2)
    cv2.imshow('tt', frames[1])

    cv2.imshow('ttt', frames[1][iy:iy+h, ix:ix+w])
    cv2.waitKey(0)
```

* Meanshift.py

```python
import cv2
import numpy as np
try:
    import Cameshift_with_demo.Feature_caculation as FC
except:
    import Feature_caculation as FC


DISTANCE_THRESHOLD = 1  # 距离差距，如果移动距离小于1，则表明到达目标中心
ITERATION_TIMES = 30  # meanshift算法的迭代轮数，最多进行30轮迭代

def MeanShift(frame,frame_hist, q_max, ix, iy, w, h,erode_1, erode_2):
    """
    MeanShift 算法实现，核心函数
    :param frame: 输入的一帧图片，这帧图片已经被转化为HSV图片
    :param frame_hist: 对应的目标区域的统计直方图
    :param q_max: 目标区域中统计直方图中的最大值
    :param ix: 当前目标的位置，左上角点x坐标
    :param iy: 当前目标的位置，右上角点y坐标
    :param w: 目标的宽度，相对x坐标
    :param h: 目标的高度，相对y坐标
    :param erode_1: 腐蚀算子大小的选择
    :param erode_2: 腐蚀算子大小的选择
    :return:
    """
    # 转HSV，并取H通道
    frame_H = FC.HSV_frame_H_Cal(frame)
    # 计算反投影图，并做腐蚀膨胀并阈值化
    back_projection_img = FC.Back_projection_cal(frame_H, frame_hist, q_max)
    back_projection_img = cv2.threshold(back_projection_img, 230, 255, cv2.THRESH_BINARY)[1]
    back_projection_img = cv2.erode(back_projection_img,
                                    cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (erode_1, erode_2)),
                                    iterations=2)
    back_projection_img = cv2.dilate(back_projection_img, cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (8, 8)),
                                     iterations=2)
    # 保留之前算出的中心位置
    xc_ = int(ix + w // 2)
    yc_ = int(iy + h // 2)
    w_ = w
    h_ = h
    # 迭代寻找最优中心点
    for j in range(ITERATION_TIMES):
        while(True):
            try:
                xc, yc, S = FC.Centroid_Mass_Cal(back_projection_img, ix, iy, w, h)  # 计算出当前目标的中心位置，面积
                if xc > 0 and yc > 0:  # 一个防错策略，防止因目标移速过快，跳出了初始搜索框，因此判断是否在初始搜索框找到目标，如果没有就扩大搜索框
                    break
                else:  # 扩大搜索框
                    ix = ix -10
                    w = w + 40
                    h = h + 40
            except: # 另一个策略，如果当前无法找到目标，则保留原先的位置，等待下一轮进行搜索，这样可以减少中间丢失目标时程序出错跳出
                xc = xc_
                yc = yc_
                w = w_
                h = h_
                break
        if np.sqrt((ix + w / 2 - xc) ** 2 + (iy + h / 2 - yc) ** 2) < DISTANCE_THRESHOLD:  # 判断找到合适的匹配位置，是就退出
            break

        ix = int(xc - np.sqrt(S) // 2)  # 更新左上角x坐标
        iy = int(yc - np.sqrt(S) // 2)  # 更新左上角y坐标
        w = int(np.sqrt(S))  # 更新目标的宽度
        h = int(np.sqrt(S))  # 更新目标的高度

    return back_projection_img, ix, iy, w, h
```

* main.py

```python
import cv2
try:
    import Cameshift_with_demo.Feature_caculation as FC
    import Cameshift_with_demo.Meanshift as meanshift
    import Cameshift_with_demo.GetRoi as GetRoi
except:
    import Feature_caculation as FC
    import Meanshift as meanshift
    import GetRoi as GetRoi



frames_hsv, frames_gray, frames_color = GetRoi.GetFrame('../Ball.avi')
# 第1个目标的位置以及特征
frame_hist_1, q_max_1, ix_1, iy_1, w_1, h_1 = FC.Hist_cal(frames_hsv[0], 197, 56, 17, 14)
# 第2个目标的位置以及特征
frame_hist_2, q_max_2, ix_2, iy_2, w_2, h_2 = FC.Hist_cal(frames_hsv[0], 193, 78, 16, 13)
# 第3个目标的位置以及特征
frame_hist_3, q_max_3, ix_3, iy_3, w_3, h_3 = FC.Hist_cal(frames_hsv[0], 190, 96, 14, 13)


for i in range(1, len(frames_hsv)):
    back_projection_img_1, ix_1, iy_1, w_1, h_1 = meanshift.MeanShift(frames_hsv[i], frame_hist_1, q_max_1, ix_1, iy_1,
                                                                       w_1, h_1, 1, 3)
    back_projection_img_2, ix_2, iy_2, w_2, h_2 = meanshift.MeanShift(frames_hsv[i], frame_hist_2, q_max_2, ix_2, iy_2,
                                                                      w_2, h_2, 2, 2)
    back_projection_img_3, ix_3, iy_3, w_3, h_3 = meanshift.MeanShift(frames_hsv[i], frame_hist_3, q_max_3, ix_3, iy_3,
                                                                       w_3, h_3, 3, 3)

    try:
        cv2.rectangle(frames_color[i], (iy_1, ix_1), (iy_1 + h_1, ix_1 + w_1), (0, 255, 0), 2)
        cv2.rectangle(frames_color[i], (iy_2, ix_2), (iy_2 + h_2, ix_2 + w_2), (255, 0, 0), 2)
        cv2.rectangle(frames_color[i], (iy_3, ix_3), (iy_3 + h_3, ix_3 + w_3), (0, 0, 255), 2)
        cv2.imshow('img', frames_color[i])
        cv2.imshow('back_projection_img1', back_projection_img_1)
        cv2.imshow('back_projection_img2', back_projection_img_2)
        cv2.imshow('back_projection_img3', back_projection_img_3)
        if cv2.waitKey(110) & 0xff == 27:
            break
        # cv2.waitKey(0)
    except:
        cv2.imshow('back_projection_img1', back_projection_img_1)
        cv2.imshow('back_projection_img2', back_projection_img_2)
        cv2.imshow('back_projection_img2', back_projection_img_3)
        cv2.waitKey(0)
```