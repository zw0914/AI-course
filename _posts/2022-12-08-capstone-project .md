
Lecture
Virtual pen
08 Dec 2022 • 劉子維 謝凱亦
Virtual pen (虛擬筆)
期末專題描述

透過 Python OpenCV 圖像識別及視訊鏡頭偵測顏色及輪廓並使用各種顏色的筆來寫出彩色的字及線條

開發過程:

讀取鏡頭
偵測並過濾顏色範圍
偵測不同顏色筆的筆尖並選取輪廓
同樣的作法套用在其他顏色上
做出墨水出水點
執行環境

這次使用的環境在軟硬體上要求不高，全部在編譯器上即可完成，唯一的要求必須要有視訊鏡頭。

以下是使用版本供參考

python 3.11
opencv
numby
直接從編譯器安裝即可

編譯器
Pycharm community Edition 2022.3.1


基本色彩介紹
RGB Color system

RGB和CMYK的色彩模式差異

RGB 又稱三原色系統，一張彩色的圖片可以由三張單一顏色的值組合而成，分別是紅、藍及綠。



HSV Color system

HSL and HSV

HSV 同樣與 RGB 為顏色表達方式，但不同的是 HSV 分為三種，分別是 Hue(色調)、Saturation(飽和度)以及Value(亮度)，利用這三者的值來表達顏色。



HSV 相較於 RGB 更容易過濾出單一的顏色，所以在執行時必須由 RGB 轉為 HSV。

顏色過濾示意



程式說明
程式主要分為過濾顏色，建立輪廓及標示出水點

讀取視訊鏡頭

使用 VideoCapture 的函式來讀取

cap = cv2.VideoCapture(0)
while True:
    ret, frame = cap.read()
    if ret:
        cv2.imshow('video', frame)
    else:
        break
    if cv2.waitKey(1) == ord('q'):
        break


偵測並過濾顏色

讀取每一偵的畫面

ret, img = cap.read()
將每一偵畫面轉為 HSV

hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
建立一個動態的控制條來調整 HSV 的各項數值以過濾顏色

cv2.namedWindow('TrackBar')
cv2.resizeWindow('TrackBar', 640, 320)

cv2.createTrackbar('Hue Min', 'TrackBar', 0, 179, empty)
cv2.createTrackbar('Hue Max', 'TrackBar', 179, 179, empty)
cv2.createTrackbar('Sat Min', 'TrackBar', 0, 255, empty)
cv2.createTrackbar('Sat Max', 'TrackBar', 255, 255, empty)
cv2.createTrackbar('Val Min', 'TrackBar', 0, 255, empty)
cv2.createTrackbar('Val Max', 'TrackBar', 255, 255, empty)
即時取得改變後的數值

while True:
    h_min = cv2.getTrackbarPos('Hue Min', 'TrackBar')
    h_max = cv2.getTrackbarPos('Hue Max', 'TrackBar')
    s_min = cv2.getTrackbarPos('Sat Min', 'TrackBar')
    s_max = cv2.getTrackbarPos('Sat Max', 'TrackBar')
    v_min = cv2.getTrackbarPos('Val Min', 'TrackBar')
    v_max = cv2.getTrackbarPos('Val Max', 'TrackBar')
    print(h_min, h_max, s_min, s_max, v_min, v_max)
過濾後顯示即時的圖片

 lower = np.array([h_min, s_min, v_min])
    upper = np.array([h_max, s_max, v_max])

    mask = cv2.inRange(hsv, lower, upper)
    result = cv2.bitwise_and(img, img, mask=mask)

    cv2.imshow('img', img)
    #cv2.imshow('hsv', hsv)
    cv2.imshow('mask', mask)
    cv2.imshow('reslut', result)
    cv2.waitKey(1)


過濾後取得顏色的六項數值

偵測顏色輪廓

偵測輪廓

contours, hierarchy = cv2.findContours(img, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_NONE)
劃出輪廓

cv2.drawContours(imgContour, cnt, -1, (255, 0, 0), 4)
使用多邊形近似輪廓

peri = cv2.arcLength(cnt, True)
vertices = cv2.approxPolyDP(cnt, peri * 0.02, True)
得出這個多邊形的四項座標

x, y, w, h = cv2.boundingRect(vertices)


標出墨水出水點

出水點座標

return x+w//2, y


加入另外兩個顏色黃跟綠

黃與綠的六項數值

penColorHSV = [[103, 102, 104, 133, 255, 255],
               [70, 118, 76, 108, 255, 245],
               [25, 93, 134, 56, 255, 255]]


將筆尖出水點改為對應顏色，並將輪廓刪去較為明顯。

penColorBGR = [[255, 0, 0],
               [0, 255, 0],
               [0, 255, 255]]


最後使其畫出線條

建立一個新列表

drawPoints = []
將列表裡的值一一畫出

def draw(drawpoints):
    for point in drawpoints:
        cv2.circle(imgContour, (point[0], point[1]), 10, penColorBGR[point[2]], cv2.FILLED)
完整 Virtual pen 程式
顏色偵測

import cv2
import numpy as np

def empty(v):
    pass

#img = cv2.imread('XiWinnie.jpg')
#img = cv2.resize(img, (0, 0), fx=0.5, fy=0.5)
cap = cv2.VideoCapture(0)

cv2.namedWindow('TrackBar')
cv2.resizeWindow('TrackBar', 640, 320)

cv2.createTrackbar('Hue Min', 'TrackBar', 0, 179, empty)
cv2.createTrackbar('Hue Max', 'TrackBar', 179, 179, empty)
cv2.createTrackbar('Sat Min', 'TrackBar', 0, 255, empty)
cv2.createTrackbar('Sat Max', 'TrackBar', 255, 255, empty)
cv2.createTrackbar('Val Min', 'TrackBar', 0, 255, empty)
cv2.createTrackbar('Val Max', 'TrackBar', 255, 255, empty)



while True:
    h_min = cv2.getTrackbarPos('Hue Min', 'TrackBar')
    h_max = cv2.getTrackbarPos('Hue Max', 'TrackBar')
    s_min = cv2.getTrackbarPos('Sat Min', 'TrackBar')
    s_max = cv2.getTrackbarPos('Sat Max', 'TrackBar')
    v_min = cv2.getTrackbarPos('Val Min', 'TrackBar')
    v_max = cv2.getTrackbarPos('Val Max', 'TrackBar')
    print(h_min, h_max, s_min, s_max, v_min, v_max)

    ret, img = cap.read()
    hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

    lower = np.array([h_min, s_min, v_min])
    upper = np.array([h_max, s_max, v_max])

    mask = cv2.inRange(hsv, lower, upper)
    result = cv2.bitwise_and(img, img, mask=mask)

    cv2.imshow('img', img)
    #cv2.imshow('hsv', hsv)
    cv2.imshow('mask', mask)
    cv2.imshow('reslut', result)
    cv2.waitKey(1)
主要程式

import cv2
import numpy as np


cap = cv2.VideoCapture(0)
#BLUE  GREEN YELLOW
penColorHSV = [[103, 102, 104, 133, 255, 255],
               [70, 118, 76, 108, 255, 245],
               [25, 93, 134, 56, 255, 255]]
penColorBGR = [[255, 0, 0],
               [0, 255, 0],
               [0, 255, 255]]
drawPoints = []
def findPen(img):
    hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
    for i in range(len(penColorHSV)):
        lower = np.array(penColorHSV[i][:3])
        upper = np.array(penColorHSV[i][3:6])

        mask = cv2.inRange(hsv, lower, upper)
        result = cv2.bitwise_and(img, img, mask=mask)
        penx, peny = findContour(mask)
        cv2.circle(imgContour, (penx, peny), 10, penColorBGR[i], cv2.FILLED)
        if peny!=-1:
            drawPoints.append([penx, peny, i])
    #cv2.imshow('result', result)

def findContour(img):
    contours, hierarchy = cv2.findContours(img, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_NONE)
    x, y, w, h, = -1, -1, -1, -1
    for cnt in contours:
        #cv2.drawContours(imgContour, cnt, -1, (255, 0, 0), 4)
        area = cv2.contourArea(cnt)
        if area > 500:
            peri = cv2.arcLength(cnt, True)
            vertices = cv2.approxPolyDP(cnt, peri * 0.02, True)
            x, y, w, h = cv2.boundingRect(vertices)
    return x+w//2, y
def draw(drawpoints):
    for point in drawpoints:
        cv2.circle(imgContour, (point[0], point[1]), 10, penColorBGR[point[2]], cv2.FILLED)

while True:
    ret, frame = cap.read()
    if ret:
        imgContour = frame.copy()
        cv2.imshow('video', frame)
        findPen(frame)
        draw(drawPoints)
        cv2.imshow('contour', imgContour)
    else:
        break
    if cv2.waitKey(1) == ord('q'):
        break
成果說明
影片展示

成果影片
照片展示





參考資料
github
