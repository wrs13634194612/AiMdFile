说明：
python生成和打开读取DICOM文件

功能包括：

1.生成带有模拟PHI的灰度图像。
2.将图像转换为DICOM格式，包含必要的元数据和假的患者信息。
3.保存生成的DICOM文件。
4.读取DICOM文件并显示其中的图像，用于验证。

运行结果，生成的文件 在项目py文件的根目录
C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\hello.py
C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\fake_phi.dcm



效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f0251bfc99614bb69a74553e511eddd4.png#pic_center)

step1:生成简单DICOM文件 C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\hello.py

```python
from PIL import Image, ImageDraw, ImageFont
import numpy as np
import pydicom
from pydicom.dataset import Dataset, FileDataset
from pydicom.uid import generate_uid, SecondaryCaptureImageStorage, ExplicitVRLittleEndian
import datetime

# 1. Make an image with fake PHI burned in

img_size = (512, 512)  # Image size (width x height)
img = Image.new("L", img_size, color=0)  # New grayscale image with black background
draw = ImageDraw.Draw(img)  # Set up drawing context

# Fake PHI text to embed in the image
phi_text = [
    "FAKEMAN, JOHN",
    "COMPLETELYREALPERSON, JOHN",
    "111222333",
    "DOB: 12/34/5678"
]

# Load font (fallback to default if needed)
try:
    font = ImageFont.truetype("DejaVuSans.ttf", size=20)
except IOError:
    font = ImageFont.load_default()

# Draw each line of PHI text on the image
for i, line in enumerate(phi_text):
    draw.text((10, 10 + i * 24), line, fill=255, font=font)  # White text

# Convert image to raw pixel bytes for DICOM
pixel_array = np.array(img, dtype=np.uint8)
pixel_bytes = pixel_array.tobytes()

# 2. Build the DICOM file

filename = "fake_phi.dcm"

# Set up DICOM file metadata
file_meta = Dataset()
file_meta.MediaStorageSOPClassUID = SecondaryCaptureImageStorage
file_meta.MediaStorageSOPInstanceUID = generate_uid()
file_meta.ImplementationClassUID = generate_uid()
file_meta.TransferSyntaxUID = ExplicitVRLittleEndian

# Create the DICOM dataset
ds = FileDataset(filename, {}, file_meta=file_meta, preamble=b"\0" * 128)

# Set basic DICOM fields
dt = datetime.datetime.now()
ds.PatientName = "FAKELAST^FAKEFIRST"
ds.PatientID = "FAKE123"
ds.StudyInstanceUID = generate_uid()
ds.SeriesInstanceUID = generate_uid()
ds.SOPInstanceUID = file_meta.MediaStorageSOPInstanceUID
ds.Modality = "MR"
ds.SeriesDescription = "MR TEST"
ds.StudyDate = dt.strftime('%Y%m%d')
ds.StudyTime = dt.strftime('%H%M%S')

# Image data settings
ds.Rows, ds.Columns = pixel_array.shape
ds.BitsAllocated = 8
ds.BitsStored = 8
ds.HighBit = 7
ds.PixelRepresentation = 0
ds.SamplesPerPixel = 1
ds.PhotometricInterpretation = "MONOCHROME2"
ds.PixelData = pixel_bytes

# Save the final DICOM file
ds.save_as(filename)
print(f"Saved DICOM with burned-in PHI: {filename}")


```

step2:生成复杂DICOM文件，包括图像处理

```python
from PIL import Image, ImageDraw, ImageFont, ImageFilter
import numpy as np
import pydicom
from pydicom.dataset import Dataset, FileDataset
from pydicom.uid import generate_uid, MRImageStorage, ExplicitVRLittleEndian
import datetime
import random
from pydicom import config

# 禁用pydicom的严格模式以允许实验性写入
config.enforce_valid_values = False


def generate_mri_like_image(size=(512, 512)):
    """生成类似MRI的合成图像"""
    # 创建基础组织结构
    x, y = np.meshgrid(np.linspace(-1, 1, size[0]), np.linspace(-1, 1, size[1]))
    d = np.sqrt(x * x + y * y)
    img = np.sin(10 * d).astype(np.float32)

    # 添加随机解剖结构
    structures = [
        (0.3, 0.5, 0.2, 180),  # 灰质
        (0.6, -0.4, 0.15, 220),  # 白质
        (-0.5, -0.5, 0.3, 100)  # CSF
    ]

    for sx, sy, sr, val in structures:
        dist = np.sqrt((x - sx) ** 2 + (y - sy) ** 2)
    img += val * np.exp(-dist / (2 * sr ** 2))

    # 添加噪声
    img += np.random.normal(0, 15, size)

    # 归一化到0-255
    img = (img - img.min()) / (img.max() - img.min()) * 255
    return img.astype(np.uint8)


def add_phi_overlay(image):
    """添加复杂的PHI覆盖层"""
    draw = ImageDraw.Draw(image)

    # 使用不同字体和样式
    fonts = []
    try:
        fonts.append(ImageFont.truetype("Arial.ttf", 24))
        fonts.append(ImageFont.truetype("Times.ttf", 20))
    except:
        fonts.append(ImageFont.load_default())

    phi_content = [
        ("PATIENT: DOE^JANE", (10, 10), 0, (255, 255, 255), 0.8),
        ("ID: 987654321", (400, 10), -5, (200, 200, 200), 0.7),
        ("DOB: 01/01/1985", (10, 450), 0, (255, 255, 255), 0.9),
        ("ACQ: 01-JAN-2023", (300, 470), 3, (220, 220, 220), 0.6),
        ("INST: MRI-01", (200, 300), 45, (180, 180, 180), 0.5)
    ]

    for text, pos, angle, color, alpha in phi_content:
        # 创建透明层
        overlay = Image.new("RGBA", image.size, (0, 0, 0, 0))
        overlay_draw = ImageDraw.Draw(overlay)

        # 旋转处理
        if angle != 0:
            text_layer = Image.new("RGBA", (200, 100), (0, 0, 0, 0))
            text_draw = ImageDraw.Draw(text_layer)
            text_draw.text((0, 0), text, fill=(*color, int(255 * alpha)), font=fonts[0])
            text_layer = text_layer.rotate(angle, expand=True)
            overlay.paste(text_layer, pos)
        else:
            overlay_draw.text(pos, text, fill=(*color, int(255 * alpha)), font=fonts[0])

        # 合并图层
        image = Image.alpha_composite(image.convert("RGBA"), overlay)

    return image.convert("L")


def create_dicom_dataset(pixel_data, study_time):
    """创建完整的DICOM数据集"""
    filename = f"mri_study_{study_time}.dcm"

    # 文件元信息
    file_meta = Dataset()
    file_meta.MediaStorageSOPClassUID = MRImageStorage
    file_meta.MediaStorageSOPInstanceUID = generate_uid()
    file_meta.ImplementationClassUID = generate_uid()
    file_meta.TransferSyntaxUID = ExplicitVRLittleEndian

    # 主数据集
    ds = FileDataset(filename, {}, file_meta=file_meta, preamble=b"\0" * 128)
    dt = datetime.datetime.now()
    study_date = dt.strftime('%Y%m%d')
    study_time = dt.strftime('%H%M%S')

    # 患者信息
    ds.PatientName = "Doe^Jane"
    ds.PatientID = "987654321"
    ds.PatientBirthDate = "19850101"
    ds.PatientSex = "F"
    ds.PatientAge = "038Y"

    # 检查信息
    ds.StudyDate = study_date
    ds.StudyTime = study_time
    ds.StudyID = "STUDY_001"
    ds.StudyDescription = "Brain MRI Research Study"

    # 系列信息
    ds.SeriesDate = study_date
    ds.SeriesTime = study_time
    ds.SeriesNumber = "3"
    ds.SeriesDescription = "T2 Weighted Axial"
    ds.Modality = "MR"
    ds.SeriesInstanceUID = generate_uid()

    # 设备信息
    ds.Manufacturer = "ACME Imaging"
    ds.ManufacturerModelName = "MRI-3T X500"
    ds.DeviceSerialNumber = "SN-123456"
    ds.SoftwareVersions = ["v3.2.1"]

    # 图像参数
    ds.ScanningSequence = "SE"
    ds.SequenceVariant = "SK"
    ds.ScanOptions = "FAST_GATING"
    ds.MRAcquisitionType = "3D"
    ds.RepetitionTime = "500"
    ds.EchoTime = "30"
    ds.MagneticFieldStrength = 3
    ds.SpacingBetweenSlices = 1.5
    ds.SliceThickness = 1.0

    # 图像几何参数
    ds.Rows, ds.Columns = pixel_data.shape
    ds.PixelSpacing = [0.5, 0.5]
    ds.SliceLocation = 25.3
    ds.ImageOrientationPatient = [1, 0, 0, 0, 1, 0]
    ds.ImagePositionPatient = [-128, -128, 25.3]

    # 图像数据设置
    ds.BitsAllocated = 8
    ds.BitsStored = 8
    ds.HighBit = 7
    ds.PixelRepresentation = 0
    ds.SamplesPerPixel = 1
    ds.PhotometricInterpretation = "MONOCHROME2"
    ds.PixelData = pixel_data.tobytes()

    return ds


def main():
    # 生成合成MRI图像
    mri_img = generate_mri_like_image()

    # 转换为PIL图像并添加PHI
    pil_img = Image.fromarray(mri_img).convert("L")
    pil_img = add_phi_overlay(pil_img.convert("RGBA")).convert("L")

    # 添加医学图像常见的伪影
    pil_img = pil_img.filter(ImageFilter.GaussianBlur(0.8))

    # 转换为numpy数组
    pixel_array = np.array(pil_img, dtype=np.uint8)

    # 创建DICOM数据集
    study_time = datetime.datetime.now().strftime("%H%M%S")
    ds = create_dicom_dataset(pixel_array, study_time)

    # 添加唯一标识符
    ds.StudyInstanceUID = generate_uid()
    ds.SOPInstanceUID = generate_uid()

    # 验证并保存
    ds.is_little_endian = True
    ds.is_implicit_VR = False

    # 保存文件
    ds.save_as("complex_mri.dcm", write_like_original=False)
    print("Generated complex DICOM with realistic features and PHI")


if __name__ == "__main__":
    main()
```

step3:读取文件 和在线DICOM查看器网站 对比，看看读取结果是否一致

```python
import pydicom


old_file = r'C:\Users\wangrusheng\Downloads\fake_phi.dcm'
ds = pydicom.dcmread(old_file)  # 读取DICOM文件‌:ml-citation{ref="6" data="citationList"}
# 显示图像
import matplotlib.pyplot as plt
plt.imshow(ds.pixel_array, cmap=plt.cm.gray)
plt.show()
```


end