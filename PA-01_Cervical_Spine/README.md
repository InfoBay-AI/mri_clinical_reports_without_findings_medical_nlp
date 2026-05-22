This repository contains sample Apache Parquet files of the Medical Hindi Image Dataset. The dataset includes sample medical images stored in compressed binary format for efficient machine learning workflows, computer vision research, dataset evaluation, and AI model development.

Convert Parquet Back to Images

# Requirements

Install dependencies:

```bash
pip install pandas pillow pyarrow pydicom numpy
```

Use the following Python script to extract images from the Parquet to DICOM IMAGES
```python
import os
import io
import numpy as np
import pandas as pd

from PIL import Image

import pydicom
from pydicom.dataset import Dataset, FileDataset
from pydicom.uid import generate_uid
from pydicom.uid import ExplicitVRLittleEndian

PARQUET_FILE = r"C:\Users\3\Downloads\PA-05_Right Shoulder_joint\train-00000-of-00001 (4).parquet"

OUTPUT_FOLDER = r"C:\Users\3\Downloads\DICOM_Output"

os.makedirs(OUTPUT_FOLDER, exist_ok=True)

print("Reading Parquet File...")

df = pd.read_parquet(PARQUET_FILE)

print("Total Records:", len(df))

for index, row in df.iterrows():

    try:

        patient_id = str(row["patient_id"])

        slice_index = str(row["slice_index"])

        modality = str(row["modality"])

        image_data = row["image"]

        # -----------------------------
        # Convert image to PIL
        # -----------------------------

        if isinstance(image_data, bytes):

            image = Image.open(
                io.BytesIO(image_data)
            )

        elif isinstance(image_data, dict):

            image = Image.open(
                io.BytesIO(image_data["bytes"])
            )

        elif isinstance(image_data, Image.Image):

            image = image_data

        else:

            print("Unsupported image type")
            continue

        image = image.convert("L")

        pixel_array = np.array(image)

        # -----------------------------
        # Create DICOM Dataset
        # -----------------------------

        file_meta = Dataset()

        file_meta.MediaStorageSOPClassUID = pydicom.uid.SecondaryCaptureImageStorage

        file_meta.MediaStorageSOPInstanceUID = generate_uid()

        file_meta.TransferSyntaxUID = ExplicitVRLittleEndian

        ds = FileDataset(
            None,
            {},
            file_meta=file_meta,
            preamble=b"\0" * 128
        )

        ds.PatientID = patient_id

        ds.Modality = modality

        ds.SeriesInstanceUID = generate_uid()

        ds.StudyInstanceUID = generate_uid()

        ds.SOPInstanceUID = generate_uid()

        ds.SOPClassUID = pydicom.uid.SecondaryCaptureImageStorage

        ds.Rows = pixel_array.shape[0]

        ds.Columns = pixel_array.shape[1]

        ds.SamplesPerPixel = 1

        ds.PhotometricInterpretation = "MONOCHROME2"

        ds.PixelRepresentation = 0

        ds.BitsStored = 8

        ds.BitsAllocated = 8

        ds.HighBit = 7

        ds.PixelData = pixel_array.tobytes()

        filename = f"{patient_id}_{slice_index}_{modality}.dcm"

        output_path = os.path.join(
            OUTPUT_FOLDER,
            filename
        )

        ds.save_as(output_path)

        print("Saved:", filename)

    except Exception as e:

        print("Error at row:", index)
        print(e)

print("\nCompleted")
print("DICOM files saved in:")
print(OUTPUT_FOLDER)
```


# Considerations

These Parquet files contain a sample of the complete dataset corpus and are provided for preview, evaluation, testing, and research purposes. The files are optimized in the Apache Parquet format for efficient storage and fast loading.

Please note that the uploaded files do not represent the full dataset collection. They include only a limited portion of the overall corpus intended to demonstrate the dataset structure, schema, and content quality.

For access to the full dataset, custom data delivery, commercial usage, or enterprise licensing options, please visit [InfoBay AI](https://infobay.ai/) or contact us directly for further information.

    -Ph: (91) 8303174762
    -Email: datareq@infobay.ai
