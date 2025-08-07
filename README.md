# Computer Vision Training Pipeline

A complete workflow for object detection using Label Studio for annotation and Azure Custom Vision for model training and inference.

## Overview

This pipeline provides an end-to-end solution for:
- Image annotation using Label Studio
- Training object detection models with Azure Custom Vision
- Automated prediction and re-annotation workflows
- Format conversion between different annotation systems

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Workflow](#workflow)
  - [1. Image Annotation with Label Studio](#1-image-annotation-with-label-studio)
  - [2. Upload to Azure Custom Vision](#2-upload-to-azure-custom-vision)
  - [3. Model Training](#3-model-training)
  - [4. Automated Prediction](#4-automated-prediction)
  - [5. Format Conversion](#5-format-conversion)
- [File Structure](#file-structure)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Prerequisites

- Python 3.8+
- Azure Custom Vision account and project
- Label Studio access
- Images for training/annotation

## Installation

Install required packages:

```bash
# Core packages
pip install label-studio
pip install label-studio-converter
pip install azure-cognitiveservices-vision-customvision
pip install azure-storage-blob
pip install coco2customvision
pip install pylabel
pip install requests
pip install python-dotenv
pip install Pillow
```

## Configuration

Create a `.env` file in your project root with your Azure credentials:

```env
ENDPOINT=https://your-region.api.cognitive.microsoft.com/
TRAINING_KEY=your-training-key
PREDICTION_KEY=your-prediction-key
PROJECT_ID=your-project-id
ITERATION_NAME=your-iteration-name
```

## Workflow

### 1. Image Annotation with Label Studio

#### 1.1 Start Label Studio

Set up environment variables and start Label Studio:

```bash
# Windows
set LOCAL_FILES_SERVING_ENABLED=true
set LABEL_STUDIO_LOCAL_FILES_DOCUMENT_ROOT="C:\path\to\exported_data"

# Unix-like systems
export LOCAL_FILES_SERVING_ENABLED=true
export LABEL_STUDIO_LOCAL_FILES_DOCUMENT_ROOT="/path/to/exported_data"

# Start Label Studio
label-studio
```

**Important:** Always verify `LABEL_STUDIO_LOCAL_FILES_DOCUMENT_ROOT` points to the correct path before starting Label Studio.

#### 1.2 Project Setup

1. Navigate to http://localhost:8080
2. Create a new project
3. Configure cloud storage:
   - Settings → Cloud Storage → Add Source Storage → Local files
   - Set absolute local path to: `/path/to/exported_data/images`
4. Set up labeling interface with your class labels:

```xml
<View>
  <Image name="image" value="$image"/>
  <RectangleLabels name="label" toName="image">
    <Label value="YourClass1" background="red"/>
    <Label value="YourClass2" background="blue"/>
    <!-- Add your actual class names here -->
  </RectangleLabels>
</View>
```

#### 1.3 Data Import and Export

- **Import**: Use Data Manager → Import to load existing annotations (JSON format)
- **Export**: Export annotations in COCO format for Azure Custom Vision compatibility

### 2. Upload to Azure Custom Vision

The pipeline automatically converts COCO format annotations to Azure Custom Vision format and uploads both images and bounding box annotations.

**Key Features:**
- Automatic format conversion from COCO to Azure Custom Vision
- Normalized coordinate handling
- Batch upload with error handling
- Mapping file generation for tracking uploads

**Required Configuration:**
```python
COCO_ANNOTATION_FILE = r"path/to/coco/annotations.json"
IMAGES_DIR = r"path/to/images"
MAPPING_FILE = r"path/to/mapping.json"
```

### 3. Model Training

Train and evaluate your model directly in the Azure Custom Vision portal:

1. Navigate to your Azure Custom Vision project
2. Click "Train" to start training
3. Evaluate model performance
4. Publish iteration when satisfied with results

### 4. Automated Prediction

Use your trained model to automatically predict bounding boxes on new images:

**Configuration:**
```python
ROOT = Path(r"path/to/new/images")
CONFIDENCE_THRESHOLD = 0.82
ITERATION_NAME = "your-published-iteration"
```

**Output:** JSON files for each predicted image

### 5. Format Conversion

#### Azure Predictions → Label Studio Format

Convert Azure Custom Vision predictions back to Label Studio format for review and refinement:

```python
def convert_json_directory_to_labelstudio(
    json_dir: str,
    output_file: str,
    image_root_url="/data/local-files/?d=images",
    copy_images_to: str | None = None
)
```

**Features:**
- Automatic image copying
- Label Studio task generation
- XML configuration file creation
- Batch processing support

## Example File Structure

```
project/
├── .env                              # Azure credentials
├── exported_data/                    # Label Studio data
│   ├── images/                      # Training images
│   └── annotations.json             # COCO format annotations
├── azure_cv_annotations.json/       # Azure predictions
├── label_studio_data/               # Converted data for Label Studio
│   ├── images/                      # Copied images
│   ├── label_studio_tasks.json     # Label Studio tasks
│   └── label_studio_tasks_config.xml # Interface configuration
├── mapping.json                     # Upload tracking
└── custom_vision_export.json       # Azure export data
```

## Troubleshooting

### Common Issues

1. **Label Studio not serving local files**
   - Verify `LOCAL_FILES_SERVING_ENABLED=true`
   - Check `LABEL_STUDIO_LOCAL_FILES_DOCUMENT_ROOT` path
   - Ensure images are in the correct subdirectory

2. **Azure upload failures**
   - Verify Azure credentials in `.env` file
   - Check that Custom Vision tags exist in project
   - Validate image file paths

3. **Coordinate conversion issues**
   - COCO format uses absolute coordinates: `[x_min, y_min, width, height]`
   - Azure Custom Vision uses normalized coordinates: `[0-1]`
   - Label Studio uses percentage coordinates: `[0-100]`

### File Path Issues

- Use raw strings (`r""`) for Windows paths
- Ensure consistent path separators
- Verify file existence before processing

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues and questions:
- Check the troubleshooting section
- Review Azure Custom Vision documentation
- Consult Label Studio documentation