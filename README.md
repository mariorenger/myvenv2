import torch
from transformers import BertTokenizer, BertForSequenceClassification
from flask import Flask, request, jsonify
https://dongnc-transfer-data.s3.ap-northeast-1.amazonaws.com/image_giangvl2.zip
app = Flask(__name__)

# Load the BERT model and tokenizer
model = BertForSequenceClassification.from_pretrained("bert-base-uncased")
tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")

# Define the prediction route
@app.route("/predict", methods=["POST"])
def predict():
    try:
        data = request.json
        input_text = data["data"]["text"]

        # Process input and make predictions
        inputs = tokenizer(input_text, return_tensors="pt", padding=True, truncation=True)
        outputs = model(**inputs)
        predictions = outputs.logits.tolist()

        response = {"predictions": predictions}
        return jsonify(response)
    except Exception as e:
        return jsonify({"error": str(e)})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

# Use an official Python runtime as a parent image
FROM python:3.8-slim

# Set the working directory within the container
WORKDIR /app

# Copy the requirements file into the container at /app
COPY requirements.txt /app/

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Copy the model files and script into the container at /app
COPY my_bert_model/ /app/my_bert_model/
COPY predict.py /app/

# Define an environment variable with the path to the model directory
ENV MODEL_PATH /app/my_bert_model

# Command to run when the container is started
CMD [ "python", "predict.py" ]

docker build -t my-bert-image:tag .

====================================================================================================

apiVersion: machinelearning.seldon.io/v1
kind: SeldonDeployment
metadata:
  name: bert-deployment
spec:
  name: bert-model
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - name: bert-container
          image: my-bert-image:tag  # Replace with your image name and tag
    graph:
      name: bert-model
      type: MODEL
    name: bert-predictor
    replicas: 1


kubectl apply -f seldon-deployment.yaml
kubectl get svc bert-deployment-bert-model

curl -X POST -H "Content-Type: application/json" -d '{"data": {"text": "your text here"}}' http://<endpoint-url>/seldon/bert-deployment/api/v1.0/predictions

import pandas as pd

# Read the Excel file into a DataFrame
excel_file = pd.ExcelFile('your_excel_file.xlsx')

# Group the DataFrame by the values in the first row (assuming it's the header)
grouped = excel_file.parse(excel_file.sheet_names[0], header=None, skiprows=1).groupby(0)

# Create a dictionary of DataFrames, one for each group
grouped_dataframes = {key: group for key, group in grouped}

# Save each group to a separate Excel file
for group_name, group_dataframe in grouped_dataframes.items():
    writer = pd.ExcelWriter(f'{group_name}.xlsx', engine='xlsxwriter')
    group_dataframe.to_excel(writer, sheet_name='Sheet1', index=False, header=False)
    writer.save()
    
import pandas as pd

# Read the Excel file into a DataFrame
excel_file = pd.ExcelFile('your_excel_file.xlsx')

# Create a writer to save the output to a new Excel file
output_excel_writer = pd.ExcelWriter('output_file.xlsx', engine='xlsxwriter')

# Loop through the sheets in the original Excel file
for sheet_name in excel_file.sheet_names:
    # Read the sheet into a DataFrame
    df = excel_file.parse(sheet_name)
    
    # Group the DataFrame by the values in the first row
    grouped = df.groupby(df.iloc[0])
    
    # Write each group to a separate sheet in the output Excel file
    for group_name, group_data in grouped:
        group_data.to_excel(output_excel_writer, sheet_name=group_name, index=False)

# Save the output Excel file
output_excel_writer.save()

