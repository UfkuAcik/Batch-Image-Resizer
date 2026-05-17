# 🖼️ Batch Image Resizer (OpenCV + Tkinter)

A modern desktop application for batch image resizing built with Python. It features a clean dark-themed Tkinter interface and uses OpenCV for fast image processing.

## ✨ Features

- 📁 Batch processing for images in a selected folder
- 📏 Percentage-based width and height scaling
- 🔗 Optional aspect-ratio lock for synchronized scaling
- ⚡ Fast resizing powered by OpenCV
- 🧠 Multiple interpolation methods:
  - Nearest Neighbor: fastest option
  - Linear / Bilinear: balanced default option
  - Cubic: higher-quality resizing
  - Lanczos4: highest-quality resizing
  - Area: best for downscaling
- 📊 Real-time processing statistics
- 📝 Detailed operation log with color-coded status messages
- 🧵 Multi-threaded processing so the UI remains responsive
- 🎨 Modern dark-themed Tkinter interface
- 📦 Automatic output folder generation with collision protection

## 🧠 Technologies Used

- Python 3.x
- OpenCV (`cv2`)
- Tkinter
- `threading`
- `glob` / `os`

## 🖥️ Interface Overview

The application uses a clean, structured layout:

- **Left panel:** Configuration settings
- **Right panel:** Processing statistics
- **Bottom panel:** Real-time logs and system feedback

## ⚙️ How It Works

1. Select a source folder containing images.
2. Set scaling percentages for width and height.
3. Choose an interpolation method.
4. Click **Start**.
5. Processed images are saved automatically to an output folder.

Output folders are generated using the selected interpolation method and scale values.

Example:

```text
<source_folder>/
  LINEAR_%40_%55/
  CUBIC_%80_%80/
```

If an output folder already exists, the application automatically creates a numbered folder such as:

```text
LINEAR_%90_%90_2
LINEAR_%90_%90_3
```

## 📦 Supported Image Formats

- PNG, JPG, JPEG
- BMP
- TIFF / TIF
- WEBP
- PPM / PGM / PBM
- EXR / HDR
- JP2
- SR / RAS

## 🚀 Installation

### 1. Clone the repository

```bash
git clone https://github.com/your-username/batch-image-resizer.git
cd batch-image-resizer
```

### 2. Install dependencies

```bash
pip install opencv-python
```

## ▶️ Run the Application

```bash
python resizer.py
```

## ⚠️ Notes

- The UI remains responsive thanks to threading.
- Existing output files are skipped automatically.
- Invalid or unreadable images are logged as warnings or errors.
- Scale values must be between 1% and 800%.

## 🤖 AI-Assisted Development

This project was developed with assistance from artificial intelligence tools.

- UI structure design, code organization, and optimization suggestions were supported by **Claude Sonnet 4.6**.
- AI was used to accelerate development, improve architecture clarity, and enhance the user experience design.

Despite AI assistance, the final implementation, integration, and decision-making were handled by the developer.

## 📸 Screenshots

<img width="1920" height="1015" alt="{E9CB8B37-FA14-4F10-9F7A-6F41D2176FA9}" src="https://github.com/user-attachments/assets/f955bbd2-7cb2-4ab4-ba14-3dce0039bf03" />

## 📌 Future Improvements

- Drag-and-drop folder support
- Image preview before and after resizing
- GPU acceleration with CUDA support
- Batch format conversion
- Estimated time remaining indicator

## 📄 License

This project is licensed under the MIT License.

## 💡 Developer Notes

This tool was built with performance and usability in mind. It uses a lightweight Tkinter interface and avoids heavy external dependencies to support cross-platform compatibility.

