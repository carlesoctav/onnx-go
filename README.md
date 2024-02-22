# A Performance Comparison Between Go ONNX Runtime, Python ultralytics and python ONNX Runtime for Object Detection inference using YOLOv8

Personal Motivation: I want learn more about Go, as it's a compiled language with garbage collection.

Other Motivation: It remove the need of PyArmor (to obfuscate) since it is already in binary format, making it a simple delivery format if we can somehow port yolov8-inference repo to Go or C or C++. It's definitely possible to rewrite yolov8-inference with C or C++ (as OpenCV and ONNX runtime are written in C++ and C). I also think it is possible to rewrite it in go,  as someone has already create a C-Go binding on ONNX runtime (https://github.com/yalue/onnxruntime_go) package and OpenCV (https://gocv.io/). 

Until now, i have had no success to statically compiling a Go program. (by statically, I mean compiling the program with the required third-party dependencies, like OpenCV and ONNX runtime (which are C files). But if you have an ONNX runtime and OpenCV shared library, the compiled binary can run flawlessly in any other computer (tested with Fahmi's computer which already has ONNX runtime shared library).

Here some comparisons between a simple image inference using Go, python ONNX Runtime, Python Ultralytics:

Here are some details about the experiment:
1. For the Go inference, I have created a repo (binary, photo, and ONNX model included). The repo can be visited here: https://github.com/carlesoctav/onnx-go. You can't just run the binary; you need to install the C ONNX runtime shared library on your system, and after that, everything is fine. I'll provide some scripts here (requires super privilege).
```bash
cd \

wget https://github.com/microsoft/onnxruntime/releases/download/v1.16.1/onnxruntime-linux-x64-1.16.1.tgz
sudo tar -xf onnxruntime-linux-x64-1.16.1.tgz
sudo rm onnxruntime-linux-x64-1.16.1.tgz
sudo mv onnxruntime-osx-arm64-1.15.1/lib/* /usr/local/lib/
sudo mv onnxruntime-osx-arm64-1.15.1/include/* /usr/local/
sudo rm -rf  onnxruntime-linux-x64-1.16.1

```

2. On the other hand, I created a simple script to profile Python Ultralytics and Python ONNX Runtime.

```python
from ultralytics import YOLO

# Load the YOLOv8 model
model = YOLO('yolov8n.pt')
# model.export(format = "onnx") # uncomment this line if you hasnt import the model to onnx
onnx_model = YOLO('yolov8n.onnx', verbose= True)
onnx_time = 0
yolo_time = 0

for i in  range(100):
    results_1 = onnx_model('../../learn/onnxruntime/car.png')
    for v in results_1[0].speed.values():
        onnx_time+=v

for i in range(100):
    results_2 = model.predict('../../learn/onnxruntime/car.png')
    for v in results_2[0].speed.values():
        yolo_time+=v

print(f"DEBUGPRINT[2]: simple.py:19: yolo_time={yolo_time/100} ms")
print(f"DEBUGPRINT[1]: simple.py:13: onnx_time={onnx_time/100} ms")

```

Here are the results:

```
go:
Min Time: 87.112408ms, Max Time: 216.287957ms, Avg Time: 109.424594ms, Count: 100
50th: 101.97106ms, 90th: 132.742448ms, 99th: 188.766604ms

python ultralytics (only avg):
DEBUGPRINT[2]: simple.py:19: yolo_time=120.58785200119019 ms

python onnx-runtime (only avg):
DEBUGPRINT[1]: simple.py:13: onnx_time=113.30247402191162 ms
```

Some noticeable things:
1. Python needs more time to run the program (besides the inference time) as it's an interpreted language, while Go runs instantly. The compile time is also very fast.
2. Python performs even worse if I run the inference while doing something else (like compiling OpenCV in this case).

```
golang:
Min Time: 96.012379ms, Max Time: 249.6243ms, Avg Time: 164.771568ms, Count: 100
50th: 166.642018ms, 90th: 199.643473ms, 99th: 223.501989ms

python (only avg):
DEBUGPRINT[2]: simple.py:19: yolo_time=211.84597253799438 ms
DEBUGPRINT[1]: simple.py:13: onnx_time=183.39422702789307 ms
```

