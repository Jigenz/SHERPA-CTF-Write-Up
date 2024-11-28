**Deepfake Detection Challenge Write-Up**

---

### **Challenge Overview**

- **Challenge Name:** Deepfake  
- **Difficulty:** Hard  
- **Creator:** X3 Security  
- **Description:**  
The Empire, under the command of Darth Vader, has developed an AI weapon capable of crafting flawless deepfake transmissions. The goal of this challenge is to extract a hidden secret embedded within the architecture of a Convolutional Neural Network (CNN) model that is used for detecting deepfake images. As a rogue codebreaker, you must investigate the model’s structure and uncover its hidden secret.

- **Given:**  
    - A Keras model file named `DeepFake_Detector_Sherpa.h5` (298 MB).
    - Password for the zip file containing the model: `S/6m3(@[`.
    - This model is self-trained and has an accuracy of 71% in detecting deepfake images.

---

### **Objective**

The goal is to extract the hidden secret (flag) embedded in the CNN model's architecture or data.

---

### **Solution Approach**

1. **Attempt to Load the Model:** Try loading the model using TensorFlow/Keras to inspect its layers and configurations.
2. **Handle Loading Errors:** Address any errors that occur due to custom layers or initializers.
3. **Inspect the Model File Directly:** Use `h5py` to read the HDF5 file and extract the model configuration and weights.
4. **Search for Suspicious Elements:** Look for unusual initializers or weights that may contain the hidden message.
5. **Extract and Interpret Hidden Data:** Convert any suspicious data into readable text to find the flag.

---

### **Step-by-Step Solution**

#### **Prerequisites:**

- **Python Version:** 3.6 or higher  
- **Libraries:**  
    - TensorFlow/Keras  
    - NumPy  
    - h5py  
    - JSON  
    - pprint  

Make sure the required libraries are installed by running the following command:
```bash
pip install tensorflow numpy h5py
```

#### **Step 1: Attempt to Load the Model**

We begin by trying to load the model using TensorFlow/Keras to examine its architecture and layers. The following code attempts to load the model:

```python
import tensorflow as tf

# Load the model
model = tf.keras.models.load_model('DeepFake_Detector_Sherpa.h5')
```

However, when attempting to load the model, we encounter the following error:

```
ValueError: Could not interpret initializer identifier: <lambda>
```

**Explanation:**
- The model uses a custom initializer (a lambda function) that Keras cannot deserialize, preventing the model from loading in the standard way.

---

#### **Step 2: Use `h5py` to Inspect the Model File**

Since we can't load the model conventionally, we use `h5py` to inspect the model’s HDF5 file directly.

```python
import h5py
import numpy as np
import json
import pprint

with h5py.File('DeepFake_Detector_Sherpa.h5', 'r') as f:
    # Part 1: Inspect model configuration
    model_config = f.attrs.get('model_config')
    if model_config is not None:
        if isinstance(model_config, bytes):
            model_config_str = model_config.decode('utf-8')
        else:
            model_config_str = model_config
        model_config_json = json.loads(model_config_str)
        print("Model Configuration:")
        pprint.pprint(model_config_json)
    
    # Part 2: Search for suspicious initializers
    print("Searching for suspicious initializers...\n")
    layers = model_config_json.get('config', {}).get('layers', [])
    for layer in layers:
        layer_name = layer.get('class_name', '')
        layer_config = layer.get('config', {})
        bias_initializer = layer_config.get('bias_initializer', {})
        if isinstance(bias_initializer, str):
            print(f"Layer: {layer_name}")
            print(f"Bias Initializer: {bias_initializer}")
            print("-" * 50)

    # Part 3: Examine weights for hidden messages
    print("Examining weights for hidden messages...\n")
    layer_names = f['model_weights'].attrs['layer_names']
    for layer_name in layer_names:
        if isinstance(layer_name, bytes):
            layer_name_str = layer_name.decode('utf-8')
        else:
            layer_name_str = layer_name
        layer_group = f['model_weights'][layer_name_str]
        weight_names = layer_group.attrs['weight_names']
        for weight_name in weight_names:
            if isinstance(weight_name, bytes):
                weight_name_str = weight_name.decode('utf-8')
            else:
                weight_name_str = weight_name
            weight_value = layer_group[weight_name_str][()]
            print(f"Layer: {layer_name_str}")
            print(f"Weight: {weight_name_str}")
            print(f"Shape: {weight_value.shape}")
            if 'bias' in weight_name_str:
                data = weight_value
                try:
                    text = ''.join([chr(int(round(x))) for x in data if 32 <= int(round(x)) <= 126])
                    if text:
                        print(f"Possible text in bias: {text}")
                except Exception as e:
                    pass
            print("-" * 50)
```

**Explanation:**
- **Part 1:** Extracts and prints the model configuration to understand the structure.
- **Part 2:** Searches for suspicious initializers such as `<lambda>`, which may be involved in hiding data.
- **Part 3:** Examines the weights of the layers, particularly looking for biases, which may contain hidden messages.

---

#### **Step 3: Run the Script and Analyze the Output**

Upon running the script:

```bash
python inspect_model.py
```

**Sample Output:**

```
Model Configuration:
{'class_name': 'Sequential',
 'config': {'layers': [...], 'name': 'sequential'}}

Searching for suspicious initializers...

Layer: Dense
Bias Initializer: <lambda>
--------------------------------------------------

Examining weights for hidden messages...

Layer: dense
Weight: dense/bias:0
Shape: (512,)
Possible text in bias: SHCTF24{Hidd3n_in_Th3_Lay3rs}
--------------------------------------------------
```

**Explanation:**
- **Part 1:** The model configuration is successfully extracted and printed.
- **Part 2:** We identify that the Dense layer uses a custom bias initializer (`<lambda>`), which is suspicious.
- **Part 3:** The script examines the weights of the layers and finds readable text in the bias of the Dense layer. The text is `"SHCTF24{Hidd3n_in_Th3_Lay3rs}"`, which is the hidden flag.

---

#### **Step 4: Extract the Hidden Message**

From the output, we obtain the hidden flag embedded in the model's bias:

**Flag:**  
![image](https://github.com/user-attachments/assets/28889900-d2cd-4a43-94e6-2f390095af12)

`SHCTF24{Hidd3n_in_Th3_Lay3rs}`

---

### **Conclusion**

By directly inspecting the model's HDF5 file using `h5py`, we bypassed issues related to custom initializers and successfully uncovered the hidden message embedded in the model. The hidden flag was stored in the bias vector of the Dense layer, and by interpreting the bias values as ASCII characters, we successfully extracted the flag.

---

### **Understanding the Code**

- **Part 1:** Inspecting Model Configuration  
  - Extract the model configuration stored in the HDF5 file.
  - Convert the configuration string to JSON format for easier inspection.

- **Part 2:** Searching for Suspicious Initializers  
  - Inspect each layer's configuration for unusual initializers (e.g., `<lambda>`).
  - Flag layers with suspicious initializers for further investigation.

- **Part 3:** Examining Weights for Hidden Messages  
  - Retrieve the weights and biases for each layer and inspect them.
  - Check if the weights or biases contain hidden text (i.e., ASCII characters).
  
---

### **Final Notes**

- **Error Handling:** The script includes `try-except` blocks to handle potential issues during data conversion.
- **Data Types:** We pay attention to the different data types (bytes vs. strings) to prevent errors.
- **Methodology:** By examining the HDF5 file directly, we were able to bypass Keras model loading issues and uncover the hidden message.

---

### **Takeaways**

- **Model Security:** This challenge demonstrates how machine learning models can hide data within their architecture and weights, emphasizing the importance of inspecting models for hidden secrets.
- **Data Inspection Tools:** Libraries like `h5py` are valuable for inspecting binary files and troubleshooting model loading issues.
- **Error Handling:** It's important to handle potential errors gracefully when dealing with complex data formats like HDF5 files.

---

**Flag:**  
`SHCTF24{Hidd3n_in_Th3_Lay3rs}`

**Challenge Solved Successfully!**
