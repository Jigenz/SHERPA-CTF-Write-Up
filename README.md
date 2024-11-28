
# CTF Writeup: "Download The Flag"

## Challenge Overview
- **Category**: Miscellaneous (Misc)
- **Challenge Name**: Download The Flag
- **Points**: 500
- **Difficulty**: Medium
- **Creator**: SKRCTF
- **Description**:  
  "Just download the flag here... Easy right?"
  
  **Instance Info**:
  - IP Address: `52.148.100.67`
  - Port: `30632`
  - Lan Domain: `72-d5af8d6d-b3ea-4a35-b1b9-b0862e449658`
  - Remaining Time: ~59 minutes (~3576s)

## Objective
Retrieve the flag from the provided web server without downloading the entire 100 GB `flag.txt` file.

## Tools Used
- **Nmap**: For port scanning and service enumeration.
- **Curl**: For interacting with the HTTP server and performing range requests.
- **Basic Linux Commands**: `cat` for viewing file contents.

## Solution Steps

### 1. Initial Reconnaissance

#### a. Port Scanning with Nmap
The first step was to identify the services running on the target IP and port. Using Nmap to scan the target IP and port:

```bash
nmap -sV 52.148.100.67 -p 30632
```

**Output**:
```
PORT      STATE SERVICE VERSION
30632/tcp open  http    nginx 1.27.2
```

- **Conclusion**: An HTTP service is running on port `30632`, specifically **nginx 1.27.2**.

#### b. Accessing the Web Server
Using `curl` to interact with the web server and retrieve the index page:

```bash
curl -v http://52.148.100.67:30632
```

**Output**:
```html
<html>
<head><title>Index of /</title></head>
<body>
<h1>Index of /</h1><hr><pre><a href="../">../</a>
<a href="flag.txt">flag.txt</a>                                           23-Nov-2024 03:27        107374182450
</pre><hr></body>
</html>
```

- **Observation**: The index page lists a `flag.txt` file with an enormous size of approximately **100 GB** (107,374,182,450 bytes).

### 2. Strategy Development
Given the impracticality of downloading the entire 100 GB file, the approach was to:
1. **Partial Content Retrieval**: Utilize HTTP range requests to download specific portions of the file where the flag might reside.
2. **Pattern Searching**: Look for the flag pattern (`FLAG{...}` or similar) within the retrieved chunks.

### 3. Partial Content Retrieval Using Range Requests

#### a. Fetching the First 1,000 Bytes
We attempted to retrieve the first 1,000 bytes of the `flag.txt` file using the `-r` flag with `curl` for range requests:

```bash
curl -r 0-999 http://52.148.100.67:30632/flag.txt -o flag_start.txt
```

**Output**:
```bash
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
100  1000  100  1000    0     0     81      0  0:00:12  0:00:12 --:--:--    94
```

**Content of `flag_start.txt`**:
```
Here is the flag: https://r.mtdv.me/here-is-the-flag
Oops.. sorry wrong flag, this the real flag:
```

- **Analysis**: The initial part of the file contains a misleading URL, followed by a message indicating the real flag is elsewhere.

#### b. Fetching the Last 1,000 Bytes
Next, we calculated the byte range for the last 1,000 bytes of the file. The total file size was `107,374,182,450` bytes, so the last 1,000 bytes would be from byte `107374181450` to `107374182449`.

```bash
curl -r 107374181450-107374182449 http://52.148.100.67:30632/flag.txt -o flag_end.txt
```

**Output**:
```bash
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
100  1000  100  1000    0     0     78      0  0:00:12  0:00:12 --:--:--    11
```

**Content of `flag_end.txt`**:
```
SHCTF24{D0wnLO4d_7h3_fl4G_W1th0u7_thE_wh0L3_f1l3}
```

- **Analysis**: The real flag is found at the end of the file in the format `SHCTF24{...}`.

## Flag
The flag is:  
`SHCTF24{D0wnLO4d_7h3_fl4G_W1th0u7_thE_wh0L3_f1l3}`
![image](https://github.com/user-attachments/assets/0a075938-0f74-4258-a370-d3611cb0ff46)

## Conclusion
By leveraging HTTP range requests, we were able to download just the relevant portions of the `flag.txt` file and retrieve the flag without the need to download the entire 100 GB file. This approach saved both time and bandwidth while still achieving the challenge's objective.

## Notes
- The flag was embedded at the end of the file, but the misleading content at the beginning helped to validate our approach of partial content retrieval.
- This solution could be extended for challenges where large files are involved, and partial content requests are the key to success.


