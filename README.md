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

