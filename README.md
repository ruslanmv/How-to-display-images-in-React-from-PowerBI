# Display Power BI Images in a React Frontend Using CLI & REST API

In this tutorial, we will:
1. **Create a free Power BI account**.
2. **Upload a sample dataset (`dataframe.csv`)** using the **Power BI CLI**.
3. **Perform queries on the dataset to generate an image (`image.png`)** using **Power BI REST API**.
4. **Display the generated Power BI image dynamically in a React frontend**.

---

## **Step 1: Create a Free Power BI Account**
### **1. Sign Up for Power BI**
1. Go to **[Power BI Signup](https://powerbi.microsoft.com/)**.
2. Click on **Try Free**.
3. Sign up with a **work/school email** (Gmail/Hotmail won't work).
4. Verify your email and log in.

### **2. Install Power BI Desktop**
- Download **Power BI Desktop** from [Microsoft Store](https://www.microsoft.com/en-us/download/details.aspx?id=58494).
- Install and open it.

---

## **Step 2: Upload a Dataset (`dataframe.csv`) Using Power BI CLI**
We will create a simple CSV file and upload it to **Power BI Service** using **Power BI CLI**.

### **1. Create a Sample CSV File (`dataframe.csv`)**
Open Notepad, VS Code, or any text editor, and copy this data:

```csv
Category,Values
A,10
B,20
C,30
D,40
```
Save it as **`dataframe.csv`**.

### **2. Install Power BI CLI**
Power BI CLI allows you to manage datasets and reports via the command line.

#### **Install Power BI CLI**
Run this command in **PowerShell**:
```powershell
Install-Module -Name MicrosoftPowerBIMgmt -Scope CurrentUser
```

#### **Authenticate Power BI CLI**
```powershell
Login-PowerBI
```
- A browser will open, log in using your Power BI account.

---

### **3. Upload `dataframe.csv` to Power BI Using CLI**
First, create a **Power BI dataset**.

#### **Create a New Dataset**
```powershell
New-PowerBIReport -Name "Demo Report" -Workspace "My Workspace"
```

#### **Import the CSV File**
```powershell
Import-PowerBIReport -Path "C:\path\to\dataframe.csv" -Workspace "My Workspace"
```

Your dataset is now in Power BI.

---

## **Step 3: Query Data & Generate an Image Using Power BI REST API**
Now, we will **perform a query** on the dataset and **export a visual as an image**.

### **1. Get Report and Workspace IDs**
In Power BI Service:
1. Open **Power BI** → Go to **Workspaces**.
2. Click on the uploaded dataset.
3. Copy **Workspace ID** & **Report ID** from the URL:
   ```
   https://app.powerbi.com/groups/{workspaceId}/reports/{reportId}
   ```

### **2. Generate an Image Using Power BI REST API**
Use PowerShell to generate a **PNG image** from the Power BI report.

#### **PowerShell Script to Export an Image**
```powershell
$token = "YOUR_ACCESS_TOKEN"
$workspaceId = "YOUR_WORKSPACE_ID"
$reportId = "YOUR_REPORT_ID"

# Start Export Process
$exportUrl = "https://api.powerbi.com/v1.0/myorg/reports/$reportId/ExportTo"
$response = Invoke-RestMethod -Uri $exportUrl -Headers @{Authorization="Bearer $token"} -Method Post -ContentType "application/json"

# Wait for Export Completion
Start-Sleep -Seconds 10  

# Get Exported Image URL
$exportId = $response.id
$exportStatusUrl = "https://api.powerbi.com/v1.0/myorg/reports/$reportId/exports/$exportId"
$exportResponse = Invoke-RestMethod -Uri $exportStatusUrl -Headers @{Authorization="Bearer $token"} -Method Get

# Save Image
Invoke-WebRequest -Uri $exportResponse.resourceLocations[0] -OutFile "C:\PowerBI_Images\plot.png"
```

---

## **Step 4: Serve Image via Node.js Backend**
Since Power BI images need authentication, we will serve them through a **Node.js Express API**.

### **1. Install Node.js & Express**
```sh
npm install express cors axios
```

### **2. Create a Node.js Server (`server.js`)**
```javascript
const express = require("express");
const fs = require("fs");
const path = require("path");

const app = express();
const PORT = 5000;

app.use(express.json());
app.use(require("cors")());

// Serve the Power BI image
app.get("/powerbi-image", async (req, res) => {
    try {
        const imagePath = path.join(__dirname, "public", "plot.png");
        if (fs.existsSync(imagePath)) {
            res.sendFile(imagePath);
        } else {
            res.status(404).send("Image not found.");
        }
    } catch (error) {
        res.status(500).send("Error fetching image.");
    }
});

app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));
```

### **3. Run the Backend**
```sh
node server.js
```

---

## **Step 5: Fetch and Display Image in React Frontend**
Now, let’s create a simple **React component** to fetch and display the Power BI image.

### **1. Install Axios in React**
```sh
npm install axios
```

### **2. Create `PowerBIImage.js`**
```javascript
import React, { useState, useEffect } from "react";
import axios from "axios";

const PowerBIImage = () => {
  const [image, setImage] = useState("");

  useEffect(() => {
    const fetchImage = async () => {
      try {
        const response = await axios.get("http://localhost:5000/powerbi-image", {
          responseType: "blob",
        });
        const imageUrl = URL.createObjectURL(response.data);
        setImage(imageUrl);
      } catch (error) {
        console.error("Error fetching Power BI image:", error);
      }
    };

    fetchImage();
    const interval = setInterval(fetchImage, 10000); // Refresh every 10s
    return () => clearInterval(interval);
  }, []);

  return (
    <div>
      <h2>Live Power BI Report</h2>
      {image ? <img src={image} alt="Power BI Report" width="600px" /> : <p>Loading...</p>}
    </div>
  );
};

export default PowerBIImage;
```

### **3. Display Component in App.js**
```javascript
import React from "react";
import PowerBIImage from "./PowerBIImage";

function App() {
  return (
    <div>
      <h1>Power BI Real-Time Image Demo</h1>
      <PowerBIImage />
    </div>
  );
}

export default App;
```

### **4. Run the React App**
```sh
npm start
```

Now open `http://localhost:3000`, and the Power BI image should update every **10 seconds**.

---

 **Now, you have a live Power BI image updating in a React frontend!**
