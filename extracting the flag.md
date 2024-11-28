# CTF Writeup: "Extracting The Flag"

## Challenge Overview
- **Category**: Miscellaneous (Misc)
- **Challenge Name**: Extracting The Flag
- **Points**: 500
- **Difficulty**: Medium
- **Creator**: SKRCTF
- **Description**:  
  The challenge required extracting a flag from a CAN log file.

  **Log File**:
  - Filename: `chal_candump.log`
  - Format: Each entry consists of a timestamp, interface identifier, and a payload in hexadecimal format.

## Objective
Extract the flag hidden in a CAN log file by parsing and analyzing the log entries, reconstructing binary data, and extracting the flag from an image.

## Tools Used
- **Python (pandas)**: For parsing and analyzing the log data.
- **Linux Commands**: For inspecting and extracting data from the log file.
- **Image Viewer**: For viewing the reconstructed image containing the flag.

## Solution Steps

### 1. Load and Inspect the CAN Log File

#### a. Initial Inspection of Log File
First, we examined the first 10 lines of the `chal_candump.log` file to understand its structure:

```bash
cat chal_candump.log | head -n 10
```

**Output**:
```
(timestamp) interface identifier#payload
```

- **Observation**: Each log entry is structured with a timestamp, interface identifier, and a payload in hexadecimal format, separated by `#`.

### 2. Parse the Log for Analysis
We used a Python script to parse the log file and extract relevant fields from the entries.

#### a. Python Script for Parsing

```python
import pandas as pd

# Load the log file
file_path = 'chal_candump.log'
with open(file_path, 'r') as f:
    data = [line.strip().split() for line in f if '#' in line]

# Convert the data into a DataFrame for easier analysis
df = pd.DataFrame(data, columns=['Timestamp', 'Interface', 'Raw'])

# Split the 'Raw' column into 'Identifier' and 'Payload'
df[['Identifier', 'Payload']] = df['Raw'].str.split('#', expand=True)
```

- **Explanation**:  
  - The script reads the log file line by line.
  - It splits each line into the timestamp, interface, and raw data (identifier and payload).
  - The raw data is further split into `Identifier` and `Payload` columns for easier analysis.

### 3. Identify Anomalous Identifier
We needed to identify any identifiers that appeared less frequently than others, which could indicate that they contain the flag.

#### a. Finding Rare Identifiers

```python
# Count occurrences of each identifier
identifier_counts = df['Identifier'].value_counts()

# Find identifiers that appear less frequently than average - 3 * std deviation
rare_identifiers = identifier_counts[identifier_counts < identifier_counts.mean() - 3 * identifier_counts.std()]

print(rare_identifiers)
```

**Output**:
```
800    10
```

- **Observation**: We found that identifier `800` had an unusually low frequency, suggesting it might contain relevant data (inconsistent payload lengths).

### 4. Extract and Reconstruct Data
We filtered the rows with identifier `800` and reconstructed the binary payload from the hexadecimal data.

#### a. Extracting the Payload and Reconstructing the Binary Data

```python
# Extract payload for identifier 800 and sort by timestamp
identifier_data = df[df['Identifier'] == '800'].sort_values(by='Timestamp')['Payload']

# Convert hex payload to binary
binary_data = b''.join(bytes.fromhex(payload) for payload in identifier_data)

# Save the binary data as an image
with open('reconstructed_image.png', 'wb') as f:
    f.write(binary_data)
```

- **Explanation**:  
  - We filtered the log entries with identifier `800`, sorted them by timestamp, and joined the payloads into one binary stream.
  - The binary data was then written to a PNG file (`reconstructed_image.png`).

### 5. Open the Reconstructed Image
We opened the reconstructed image to reveal the hidden flag.

```bash
xdg-open reconstructed_image.png
```

- **Observation**: The image contained the flag.

## Flag
The flag extracted from the image is:  
`SHCTF24{800_D035N7_3X157}`
![image](https://github.com/user-attachments/assets/cf55795f-1a76-4537-88c5-8d7eb017f166)

## Conclusion
By parsing the log file, identifying rare identifiers, and reconstructing the binary data, we successfully retrieved the flag embedded in an image. This challenge required data parsing, hexadecimal-to-binary conversion, and some basic image handling to extract the hidden flag.

