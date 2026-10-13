---
type: "practice"
difficulty: "Intermediate"
---

# How would you accept a CSV file upload in FastAPI, run predictions on each row, and return the results?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ File Uploads and Inference](<../../../../TOOLS/Python/FastAPI/File_Uploads_and_Inference.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To accept a CSV file upload in FastAPI, process it row by row using a machine learning model, and return the predicted results, we'll leverage FastAPI's capabilities for handling file uploads and processing data efficiently. The key steps involve defining an API endpoint that accepts a multipart/form-data request containing a CSV file, reading this file, applying predictions to each row, and then returning the results in a structured format.

**Code:**

```python
from fastapi import FastAPI, File, UploadFile
import pandas as pd
import numpy as np

app = FastAPI()

# Define the endpoint for uploading CSV files
@app.post("/predict/")
async def predict(file: UploadFile = File(...)):
    # Read the uploaded CSV file into a DataFrame
    contents = await file.read()
    df = pd.read_csv(io.BytesIO(contents))

    # Apply predictions to each row of the DataFrame using your model
    predictions = apply_predictions(df)

    # Convert the predictions back to a pandas Series or DataFrame for response formatting
    result = pd.Series(predictions, index=df.index)

    return {"predictions": result.tolist()}
```

**Example input / output:**

```

An example input and output can be as follows:

Input:

A CSV file named data.csv with the following content:
CODE
Copy
id,feature1,feature2
1,3.4,5.6
2,7.8,9.0
3,2.3,4.5

Output:

A JSON response containing the predictions for each row:
JSON
Copy
{
  "predictions": [0.9, 1.1, 0.6]
}
```

**Flow:**

1. Receive CSV Upload
2. Read CSV to DataFrame
3. Apply Predictions
4. Return Predicted Results asJSON

