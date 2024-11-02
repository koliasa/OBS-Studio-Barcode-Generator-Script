# OBS Studio Barcode Generator Script

## Description

This script for OBS Studio displays a barcode on your scene. You can configure the barcode data, its width, and height directly through the OBS Studio settings interface.

## Requirements

- **OBS Studio** (version with Python scripting support)
- **Python 3** installed on your system
- **Pillow** library for Python

## Installation

### 1. Install Python and Required Libraries

Ensure you have Python 3 installed. If not, install it using your package manager. For example, on Debian/Ubuntu:

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

### 2. Create a Virtual Environment

It's recommended to create a virtual environment to isolate packages.

```bash
python3 -m venv ~/obs_barcode_env
```

### 3. Activate the Virtual Environment

```bash
source ~/obs_barcode_env/bin/activate
```

### 4. Install the Pillow Library

```bash
pip install Pillow
```

## Setting up the Script in OBS Studio

### 1. Load the Script

Copy the following code into a file named `barcode_generator.py`:

```python
import obspython as obs
from PIL import Image, ImageDraw

# Parameters (these values will be changed in OBS)
barcode_data = ""
barcode_width = 300
barcode_height = 100

# Function to update the barcode
def update_barcode():
    global barcode_data, barcode_width, barcode_height

    # Check if data is provided
    if not barcode_data:
        return

    # Create barcode image
    img = Image.new("1", (barcode_width, barcode_height), "white")
    draw = ImageDraw.Draw(img)

    # Generate simple barcode with lines
    x = 10
    for index, char in enumerate(barcode_data):
        if char.isdigit() and int(char) % 2 == 0:  # each digit generates a black or white stripe
            draw.rectangle([x, 0, x + 4, barcode_height], fill="black")
        x += 6  # distance between stripes

    # Save barcode to file
    img.save("/tmp/barcode.png")

    # Update image in OBS
    source = obs.obs_get_source_by_name("Barcode Image")
    if source:
        settings = obs.obs_data_create()
        obs.obs_data_set_string(settings, "file", "/tmp/barcode.png")
        obs.obs_source_update(source, settings)
        obs.obs_data_release(settings)
        obs.obs_source_release(source)

# Function to update parameters from OBS settings
def script_update(settings):
    global barcode_data, barcode_width, barcode_height
    barcode_data = obs.obs_data_get_string(settings, "barcode_data")
    barcode_width = obs.obs_data_get_int(settings, "barcode_width")
    barcode_height = obs.obs_data_get_int(settings, "barcode_height")
    update_barcode()

# Function to create OBS settings interface
def script_properties():
    props = obs.obs_properties_create()

    # Field to input barcode data
    obs.obs_properties_add_text(props, "barcode_data", "Barcode Data", obs.OBS_TEXT_DEFAULT)
    
    # Width and height settings for the barcode
    obs.obs_properties_add_int(props, "barcode_width", "Barcode Width", 100, 1000, 10)
    obs.obs_properties_add_int(props, "barcode_height", "Barcode Height", 50, 500, 10)

    return props

# Initialize default values
def script_defaults(settings):
    obs.obs_data_set_default_string(settings, "barcode_data", "1234567890")
    obs.obs_data_set_default_int(settings, "barcode_width", 300)
    obs.obs_data_set_default_int(settings, "barcode_height", 100)
```

### 2. Add the Script in OBS Studio

1. **Open OBS Studio**.
2. Go to **Tools** → **Scripts**.
3. Click **+** to add a script, and select the `barcode_generator.py` file.

### 3. Add Image Source

1. In your scene, click **+** in the **Sources** section and select **Image Source**.
2. Name it **Barcode Image**.
3. In the source settings, set the file path to `/tmp/barcode.png` (the script will automatically create this file).

### 4. Configure Script Settings

1. In the **Scripts** window, find the loaded `barcode_generator.py`.
2. Enter the desired barcode data in the **Barcode Data** field.
3. Set the desired **Width** and **Height** of the barcode.
4. The script will automatically update the barcode image in your scene.

## Usage

1. **Open OBS Studio** and load the script if you haven't done so already.
2. **Configure barcode settings** through the script interface in OBS.
3. **Ensure** the **Barcode Image** source is added to your scene.
4. **Use the scene** with the displayed barcode in your live stream or recordings.

## Troubleshooting

### Error `ModuleNotFoundError: No module named 'PIL'`

Ensure you installed the Pillow library in your virtual environment:

```bash
pip install Pillow
```

### Creating the Virtual Environment

If you haven't created the virtual environment, follow the steps in the **Installation** section.

### Ensure OBS is Using the Correct Python Interpreter

1. Open OBS Studio.
2. Go to **Tools** → **Settings** → **Scripts**.
3. Verify that the path to the Python interpreter is correctly set (path to your virtual environment).

## License

This project is licensed under the [MIT License](LICENSE).
