---
layout: post
title: capstone-project
author: [ZW0914]
category: [Lecture]
tags: [jekyll, ai]
---

期末專題實作 試穿衣服Virty try on
###
系統測試成果展示
<iframe width="684" height="385" src="https://www.youtube.com/embed/DtzN5vtEgOk" title="RL-Robocar" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


---
### OpenPose
**Kaggle:** [rkuo2000/openpose-pytorch](https://github.com/rkuo2000/openpose-pytorch)<br>
![](https://github.com/rkuo2000/AI-course/blob/gh-pages/images/OpenPose_pytorch_racers.png?raw=true)
![](https://github.com/rkuo2000/AI-course/blob/gh-pages/images/OpenPose_pytorch_fall1.png?raw=true)

---
### Pose Recognition (姿態辨識)


##Virty try on(試穿衣服)


**專題實作步驟:**
1. 建立身體動作之姿態照片資料集 (例如：5 poses , take 20 pictures of each pose)<br>
2. 始用**MMPose** 辨識出照片中的各姿勢之身體關鍵點 (use MMPose convert 16 keypoints (x,y) of each pose)<br>
3. 產生姿態關鍵點資料集 x_train.append(pose_keypoints) ( x_train.shape = (20x5, 16, 2), y_train.shape= (20x5, 1) )<br>
4. 建立DNN模型並訓練模型, 然後下載模型檔`pose_dnn.h5`至PC <br>
5. 於PC建立帶camera輸入之服務器程式, 載入模型`pose_dnn.h5`進行姿態動作辨識 <br>

**模型建構與訓練之程式樣本** (PC or Kaggle)<br>

```
input_shape=(16,2)
num_classes=5

inputs = layers.Input(shape=input_shape)
x = layers.Dense(128)(inputs)
outputs = layers.Dense(num_classes, activation="softmax")(x)
model = models.Model(inputs=inputs, outputs=outputs)

models.compile(loss = 'categorical_crossentropy', optimizer = 'adam' , metrics = ['accuracy'])

history = model.fit(x_train, y_train, batch_size=1, epochs=20, validation_data=(x_test, y_test))
models.save_model(model, 'pose_dnn.h5')
```

**姿態辨識服務器之程式樣本** (PC with Camera)<br>

```
model = models.load_model('models/pose_dnn.h5')
labels = ['stand', 'raise-right-arm', 'raise-left-arm', 'cross arms','both-arms-left']

cap = cv2.VideoCapture(0)

while(cap.isOpened()):
    ret, frame = cap.read()
    image = cv2.cvtColor(frame,cv2.COLOR_BGR2RGB)
    
    mmdet_results = inference_detector(det_model, image) # 人物偵測產生BBox
    person_results = process_mmdet_results(mmdet_results, args.det_cat_id) # 記住人物之BBox  
    pose_results, returned_outputs = inference_top_down_pose_model(...) # 感測姿態產生pose keypoints
    
    x_test = np.array(preson_results).reshape(1,16,2) # 將Keypoints List 轉成 numpy Array
    preds = model.fit(x_test) # 辨識姿態動作
    maxindex = int(np.argmax(preds))
    txt = labels[maxindex]
    print(txt)
```

---
### Head Pose Estimation
**Kaggle:** [rkuo2000/head-pose-estimation](https://kaggle.com/rkuo2000/head-pose-estimation)<br>
<iframe width="652" height="489" src="https://www.youtube.com/embed/BHwHmCUHRyQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---
### VTuber-Unity 
**Head-Pose-Estimation + Face-Alignment + GazeTracking**<br>

<u>Build-up Steps</u>:
1. Create a character: **[VRoid Studio](https://vroid.com/studio)**
2. Synchronize the face: **[VTuber_Unity](https://github.com/kwea123/VTuber_Unity)**
3. Take video: **[OBS Studio](https://obsproject.com/download)**
4. Post-processing:
 - Auto-subtitle: **[Autosub](https://github.com/kwea123/autosub)**
 - Auto-subtitle in live stream: **[Unity_live_caption](https://github.com/kwea123/Unity_live_caption)**
 - Encode the subtitle into video: **[小丸工具箱](https://maruko.appinn.me/)**
5. Upload: YouTube
6. [Optional] Install CUDA & CuDNN to enable GPU acceleration
7. To Run <br>
`$git clone https://github.com/kwea123/VTuber_Unity` <br>
`$python demo.py --debug --cpu` <br>

<p align="center"><img src="https://github.com/kwea123/VTuber_Unity/blob/master/images/debug_gpu.gif?raw=true"></p>

---
### OpenVtuber
<u>Build-up Steps</u>:
* Repro [Github](https://github.com/1996scarlet/OpenVtuber)<br>
`$git clone https://github.com/1996scarlet/OpenVtuber`<br>
`$cd OpenVtuber`<br>
`$pip3 install –r requirements.txt`<br>
* Install node.js for Windows <br>
* run Socket-IO Server <br>
`$cd NodeServer` <br>
`$npm install express socket.io` <br>
`$node. index.js` <br>
* Open a browser at  http://127.0.0.1:6789/kizuna <br>
* PythonClient with Webcam <br>
`$cd ../PythonClient` <br>
`$python3 vtuber_link_start.py` <br>

<p align="center"><img src="https://camo.githubusercontent.com/83ad3e28fa8a9b51d5e30cdf745324b09ac97650aea38742c8e4806f9526bc91/68747470733a2f2f73332e617831782e636f6d2f323032302f31322f31322f72564f33464f2e676966"></p>

---
### Hand Pose
![](https://github.com/facebookresearch/InterHand2.6M/blob/main/assets/teaser.gif?raw=true)
**Dataset:** [InterHand2.6M](https://github.com/facebookresearch/InterHand2.6M)<br>

1. Download pre-trained InterNet from [here](https://drive.google.com/drive/folders/1BET1f5p2-1OBOz6aNLuPBAVs_9NLz5Jo?usp=sharing)
2. Put the model at `demo` folder
3. Go to `demo` folder and edit `bbox` in [here](https://github.com/facebookresearch/InterHand2.6M/blob/5de679e614151ccfd140f0f20cc08a5f94d4b147/demo/demo.py#L74)
4. run `python demo.py --gpu 0 --test_epoch 20`
5. You can see `result_2D.jpg` and 3D viewer.

**Camera positios visualization demo**
1. `cd tool/camera_visualize`
2. Run `python camera_visualize.py`


<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*

