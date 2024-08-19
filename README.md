## Hello GitHub

```Python

import sys
import cv2
import numpy as np
import threading
from PyQt5.QtWidgets import QApplication, QMainWindow, QLabel, QWidget, QGridLayout, \
    QVBoxLayout, QHBoxLayout, QComboBox, QStatusBar, QPushButton, QFileDialog
from PyQt5.QtGui import QPixmap, QImage
from PyQt5.QtCore import Qt, QTimer
import onnxruntime as ort
import concurrent.futures
import serial
import json
import time


###--- 应用窗口 ---###
class ImageRecognizerApp(QMainWindow):
    def __init__(self):
        super().__init__()

        # 固定窗口大小
        self.setFixedSize(1000, 600)
        self.setWindowTitle('Image Recognizer')

        # 创建主窗口部件
        self.central_widget = QWidget(self)
        self.setCentralWidget(self.central_widget)

        # 使用QGridLayout布局
        self.grid_layout = QGridLayout(self.central_widget)

        # 图片显示标签，固定大小
        self.image_label = QLabel(self)
        self.image_label.setFixedSize(400, 300)
        self.image_label.setAlignment(Qt.AlignCenter)
        self.grid_layout.addWidget(self.image_label, 0, 0, 2, 1)  # 占据两行

        # 识别结果显示标签的容器
        self.result_container = QWidget(self)
        self.result_layout = QVBoxLayout(self.result_container)
        self.result_layout.setAlignment(Qt.AlignLeft)  # 识别结果左对齐
        self.grid_layout.addWidget(self.result_container, 0, 1)

        # 识别结果显示标签
        self.result_label = QLabel('识别结果', self)
        self.result_label.setAlignment(Qt.AlignLeft | Qt.AlignTop)  # 左对齐
        self.result_layout.addWidget(self.result_label)

        # 用于显示识别结果的图像标签，固定大小
        self.result_image_label = QLabel(self)
        self.result_image_label.setFixedSize(200, 150)
        self.result_image_label.setAlignment(Qt.AlignCenter)
        self.result_layout.addWidget(self.result_image_label)

        # 开始识别按钮
        self.recognize_button = QPushButton('开始识别', self)
        self.recognize_button.clicked.connect(self.start_recognition)
        self.grid_layout.addWidget(self.recognize_button, 2, 1)

        # 选择图片按钮
        self.select_image_button = QPushButton('选择图片', self)
        self.select_image_button.clicked.connect(self.select_image)
        self.grid_layout.addWidget(self.select_image_button, 2, 0)

        # 识别时间间隔选择布局
        interval_layout = QHBoxLayout()
        interval_label = QLabel('设置识别时间间隔:')
        self.interval_combo_box = QComboBox(self)
        self.interval_combo_box.addItems([str(i) for i in range(5, 35, 5)])
        self.interval_combo_box.currentIndexChanged.connect(self.update_recognition_interval)
        interval_layout.addWidget(interval_label)
        interval_layout.addWidget(self.interval_combo_box)
        self.grid_layout.addLayout(interval_layout, 4, 0, 1, 2)

        # 状态栏
        self.status_bar = QStatusBar(self)
        self.setStatusBar(self.status_bar)

        # 初始化ONNX模型
        self.model_path = "best.onnx"
        self.onnx_model = ONNXImageRecognizer(self.model_path)

        # 识别定时器（5秒识别一次）
        self.recognition_timer = QTimer(self)
        self.recognition_timer.timeout.connect(self.recognize_frame)
        self.recognition_interval = 5000  # 默认识别时间间隔为5秒

        # 用于保存当前帧
        self.current_frame = None

        # 线程池（最多4个线程）
        self.executor = concurrent.futures.ThreadPoolExecutor(max_workers=4)

        # 启动摄像头线程
        self.cap = None
        self.camera_thread = threading.Thread(target=self.start_camera)
        self.camera_thread.start()

        # 串口相关参数
        self.serial_port = 'COM3'
        self.baud_rate = 9600

    def transPort(self, ret):
        ser = serial.Serial(self.serial_port, self.baud_rate, timeout=1)

        data = {
            "id": "2305468827",
            "params": {
                "led1": {
                    "value": ret
                }
            }
        }

        json_data = json.dumps(data)

        ser.write(b'AT\r')
        time.sleep(1)

        ser.write(json_data.encode("utf-8"))
        time.sleep(1)

        response = ser.read_all().decode("utf-8")
        print(f"{response}")

        ser.close()

    def start_recognition(self):
        if self.cap is not None and self.cap.isOpened():
            # 开始识别，设置定时器每隔指定的时间间隔触发一次
            self.recognition_timer.start(self.recognition_interval)
        else:
            self.result_label.setText("请先打开摄像头！")

    def start_camera(self):
        # 打开摄像头
        self.cap = cv2.VideoCapture(0)
        while True:
            ret, frame = self.cap.read()
            if ret:
                self.current_frame = frame
                self.display_frame(frame)
            else:
                self.result_label.setText("无法读取摄像头数据")
            cv2.waitKey(20)  # 每20毫秒捕获一帧

    def select_image(self):
        # 打开文件对话框选择图片
        file_name, _ = QFileDialog.getOpenFileName(self, "选择图片", "", "Images (*.png *.xpm *.jpg *.bmp);;All Files (*)")
        if file_name:
            img = cv2.imread(file_name)
            if img is not None:
                self.current_frame = img
                self.display_frame(img)
                self.process_frame(img)

    def recognize_frame(self):
        if self.current_frame is not None:
            self.executor.submit(self.process_frame, self.current_frame)

    def process_frame(self, img):
        # 使用线程池处理帧（用于视频流）
        det_boxes, scores, ids = self.onnx_model.infer_img(img)

        result_text = ""
        for box, score, id in zip(det_boxes, scores, ids):
            label = self.onnx_model.dic_labels[id]  # 仅保留标签
            result_text += f"{label}\n"
            self.onnx_model.plot_one_box(box.astype(int), img, color=(255, 0, 0), label=label, line_thickness=2)

        # 限制显示最多10行结果
        result_lines = result_text.split('\n')
        if len(result_lines) > 10:
            result_text = '\n'.join(result_lines[:10])

        # 更新识别结果显示标签
        self.result_label.setText(result_text)
        self.transPort(result_text)

        # 显示识别结果图像
        self.display_recognition_result(img)

    def display_frame(self, img):
        img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        q_img = QImage(img_rgb.data, img_rgb.shape[1], img_rgb.shape[0], img_rgb.strides[0], QImage.Format_RGB888)
        self.image_label.setPixmap(QPixmap.fromImage(q_img).scaled(self.image_label.size(), Qt.KeepAspectRatio))

    def display_recognition_result(self, img):
        # 在标签中显示识别结果图片
        img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        q_img = QImage(img_rgb.data, img_rgb.shape[1], img_rgb.shape[0], img_rgb.strides[0], QImage.Format_RGB888)
        self.result_image_label.setPixmap(
            QPixmap.fromImage(q_img).scaled(self.result_image_label.size(), Qt.KeepAspectRatio))

    def update_recognition_interval(self):
        selected_interval = int(self.interval_combo_box.currentText()) * 1000
        self.recognition_interval = selected_interval
        self.update_status_bar()

    def update_status_bar(self):
        if self.cap is not None and self.cap.isOpened():
            self.status_bar.showMessage(f"摄像头已打开 - 识别间隔: {self.recognition_interval / 1000} 秒")
        else:
            self.status_bar.showMessage("摄像头未打开")

    def closeEvent(self, event):
        # 关闭摄像头和定时器
        if self.cap is not None:
            self.cap.release()
        self.recognition_timer.stop()
        self.executor.shutdown()
        event.accept()
    ###--- 应用窗口 ---###

    ###--- yolo 处理部分 ---###


class ONNXImageRecognizer:
    def __init__(self, model_path):
        self.model_pb_path = model_path
        self.so = ort.SessionOptions()
        providers = ['CPUExecutionProvider']
        self.net = ort.InferenceSession(self.model_pb_path, self.so, providers=providers)

        self.dic_labels = {0: 'yellow_0',
                           1: 'white_0',
                           2: 'green_0',
                           3: 'green_1',
                           4: 'rad_0'}

        self.model_h = 640
        self.model_w = 640
        self.nl = 3
        self.na = 3
        self.stride = [8., 16., 32.]
        self.anchors = [[10, 13, 16, 30, 33, 23],
                        [30, 61, 62, 45, 59, 119],
                        [116, 90, 156, 198, 373, 326]]
        self.anchor_grid = np.asarray(self.anchors, dtype=np.float32).reshape(self.nl, -1, 2)

    def infer_img(self, img0):
        img = cv2.resize(img0, (self.model_w, self.model_h), interpolation=cv2.INTER_AREA)
        img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        img = img.astype(np.float32) / 255.0
        blob = np.expand_dims(np.transpose(img, (2, 0, 1)), axis=0)
        outs = self.net.run(None, {self.net.get_inputs()[0].name: blob})[0].squeeze(axis=0)
        outs = self.cal_outputs(outs)
        img_h, img_w, _ = np.shape(img0)
        return self.post_process_opencv(outs, img_h, img_w)

    def cal_outputs(self, outs):
        row_ind = 0
        grid = [np.zeros(1)] * self.nl
        for i in range(self.nl):
            h, w = int(self.model_w / self.stride[i]), int(self.model_h / self.stride[i])
            length = int(self.na * h * w)
            if grid[i].shape[2:4] != (h, w):
                grid[i] = self._make_grid(w, h)

            outs[row_ind:row_ind + length, 0:2] = (outs[row_ind:row_ind + length, 0:2] * 2. - 0.5 + np.tile(
                grid[i], (self.na, 1))) * int(self.stride[i])
            outs[row_ind:row_ind + length, 2:4] = (outs[row_ind:row_ind + length, 2:4] * 2) ** 2 * np.repeat(
                self.anchor_grid[i], h * w, axis=0)
            row_ind += length
        return outs

    def _make_grid(self, nx, ny):
        xv, yv = np.meshgrid(np.arange(ny), np.arange(nx))
        return np.stack((xv, yv), 2).reshape((-1, 2)).astype(np.float32)

    def post_process_opencv(self, outputs, img_h, img_w, thred_nms=0.8, thred_cond=0.8):
        conf = outputs[:, 4].tolist()
        c_x = outputs[:, 0] / self.model_w * img_w
        c_y = outputs[:, 1] / self.model_h * img_h
        w = outputs[:, 2] / self.model_w * img_w
        h = outputs[:, 3] / self.model_h * img_h
        p_cls = outputs[:, 5:]
        if len(p_cls.shape) == 1:
            p_cls = np.expand_dims(p_cls, 1)
        cls_id = np.argmax(p_cls, axis=1)

        p_x1 = np.expand_dims(c_x - w / 2, -1)
        p_y1 = np.expand_dims(c_y - h / 2, -1)
        p_x2 = np.expand_dims(c_x + w / 2, -1)
        p_y2 = np.expand_dims(c_x + h / 2, -1)
        areas = np.concatenate((p_x1, p_y1, p_x2, p_y2), axis=-1)

        areas = areas.tolist()
        ids = cv2.dnn.NMSBoxes(areas, conf, thred_cond, thred_nms)
        return np.array(areas)[ids], np.array(conf)[ids], cls_id[ids]

    def plot_one_box(self, x, img, color=None, label=None, line_thickness=None, scale_factor=1.0):
        box_w = x[2] - x[0]
        box_h = box_w  # 绘制正方形框

        new_w = box_w * scale_factor
        new_h = box_h * scale_factor

        c_x = x[0] + box_w / 2
        c_y = x[1] + box_h / 2
        new_x1 = int(c_x - new_w / 2)
        new_y1 = int(c_y - new_h / 2)
        new_x2 = int(c_x + new_w / 2)
        new_y2 = int(c_y + new_h / 2)

        tl = line_thickness or round(0.002 * (img.shape[0] + img.shape[1]) / 2) + 1
        color = color or [random.randint(0, 255) for _ in range(3)]
        c1, c2 = (new_x1, new_y1), (new_x2, new_y2)
        cv2.rectangle(img, c1, c2, color, thickness=tl, lineType=cv2.LINE_AA)

        if label:
            tf = max(tl - 1, 1)
            t_size = cv2.getTextSize(label, 0, fontScale=tl / 3, thickness=tf)[0]
            c2 = c1[0] + t_size[0], c1[1] - t_size[1] - 3
            cv2.rectangle(img, c1, c2, color, -1, cv2.LINE_AA)
            cv2.putText(
                img,
                label,
                (c1[0], c1[1] - 2),
                0,
                tl / 3,
                [225, 255, 255],
                thickness=tf,
                lineType=cv2.LINE_AA,
            )

    ###--- yolo 处理部分 ---###


if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = ImageRecognizerApp()
    window.show()
    sys.exit(app.exec_())

```

![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=2876225417&show_icons=true&theme=radical)

![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=2876225417&layout=compact&theme=dark)

![你的 GitHub wakatime stats](https://github-readme-stats.vercel.app/api/wakatime?username=ppQwQqq)




