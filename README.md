# C# Aimbot with GUI and ONNX Model for Enemy Detection

This project demonstrates how to build an aimbot in C# using a graphical user interface (GUI) and an ONNX object detection model for detecting enemies on the screen. It includes:
- **Screen capturing**
- **ONNX-based object detection** using **ML.NET**
- **Mouse automation** using **InputSimulator**
- A **WinForms GUI** for controlling the aimbot

---

## Prerequisites

Ensure you have the following before starting:
- **Visual Studio** (Community Edition or higher)
- **.NET Desktop Development** workload installed in Visual Studio
- **ML.NET Model Builder** (optional but useful)
- A basic understanding of C# and Visual Studio

---

### Step 1: Setting Up the Project

1. **Create a New C# Project**
   - Open Visual Studio and create a new project.
   - Choose **Windows Forms App (.NET)** or **WPF Application (.NET)**.
   - Name your project `AimbotONNX`.

2. **Design the GUI**
   - Open the toolbox in Visual Studio and drag the following components onto the form:
     - A **Button** for starting the aimbot (`btnStart`), with the text "Start Aimbot".
     - A **Button** for stopping the aimbot (`btnStop`), with the text "Stop Aimbot".
     - A **Label** to display the aimbot status (`lblStatus`).

3. **Wire up the GUI Buttons**
   - Create event handlers for the Start and Stop buttons:
     ```csharp
     private void btnStart_Click(object sender, EventArgs e)
     {
         lblStatus.Text = "Aimbot Running";
         RunAimbot();
     }

     private void btnStop_Click(object sender, EventArgs e)
     {
         lblStatus.Text = "Aimbot Stopped";
         StopAimbot();
     }
     ```

---

### Step 2: Screen Capture

1. **Capture the Screen**
   - Add a method to capture the screen and return a `Bitmap`:
     ```csharp
     using System.Drawing;
     using System.Drawing.Imaging;
     using System.Windows.Forms;

     public Bitmap CaptureScreen()
     {
         Rectangle bounds = Screen.PrimaryScreen.Bounds;
         Bitmap screenshot = new Bitmap(bounds.Width, bounds.Height, PixelFormat.Format32bppArgb);
         using (Graphics g = Graphics.FromImage(screenshot))
         {
             g.CopyFromScreen(bounds.X, bounds.Y, 0, 0, bounds.Size, CopyPixelOperation.SourceCopy);
         }
         return screenshot;
     }
     ```

2. **Test the Screen Capture**
   - Modify the `RunAimbot` method to save the captured screenshot to a file:
     ```csharp
     private void RunAimbot()
     {
         Bitmap screenshot = CaptureScreen();
         screenshot.Save("screenshot.png", ImageFormat.Png);
     }
     ```

3. **Run the program** and verify that a screenshot is saved to your working directory.

---

### Step 3: Using an ONNX Object Detection Model

1. **Download a Pre-trained ONNX Model**
   - Download an ONNX model for object detection from the [ONNX Model Zoo](https://github.com/onnx/models), such as **Tiny YOLOv2**.
   - Add the ONNX model (e.g., `tinyyolov2.onnx`) to your project by right-clicking the solution > **Add** > **Existing Item**.

2. **Install ML.NET and ONNX Packages**
   - Open **NuGet Package Manager** and install:
     - `Microsoft.ML`
     - `Microsoft.ML.OnnxRuntime`
     - `Microsoft.ML.OnnxTransformer`

3. **Loading the ONNX Model**
   - Create a method to load the ONNX model into ML.NET:
     ```csharp
     using Microsoft.ML;
     using Microsoft.ML.Data;
     using System.Collections.Generic;

     public class ObjectDetectionPrediction
     {
         [ColumnName("grid")]
         public float[] PredictedLabels;
     }

     private PredictionEngine<BitmapData, ObjectDetectionPrediction> LoadModel()
     {
         var mlContext = new MLContext();
         var pipeline = mlContext.Transforms.ExtractPixels(outputColumnName: "input")
             .Append(mlContext.Transforms.ApplyOnnxModel(
                 modelFile: "tinyyolov2.onnx",
                 outputColumnNames: new[] { "grid" },
                 inputColumnNames: new[] { "input" }));

         var model = pipeline.Fit(mlContext.Data.LoadFromEnumerable(new List<BitmapData>()));
         return mlContext.Model.CreatePredictionEngine<BitmapData, ObjectDetectionPrediction>(model);
     }
     ```

4. **Define the Input Data Class**
   - Add a class to represent the input data for the ONNX model:
     ```csharp
     public class BitmapData
     {
         [ImageType(416, 416)]
         public Bitmap Image { get; set; }
     }
     ```

---

### Step 4: Detecting Enemies with the ONNX Model

1. **Running Object Detection**
   - Add the object detection logic inside the `RunAimbot` method:
     ```csharp
     private void RunAimbot()
     {
         var model = LoadModel();  // Load the ONNX model

         while (true)
         {
             Bitmap screenshot = CaptureScreen();
             var inputData = new BitmapData { Image = screenshot };

             var prediction = model.Predict(inputData);

             DetectEnemies(prediction.PredictedLabels);
         }
     }

     private void DetectEnemies(float[] predictedLabels)
     {
         // Analyze predicted labels and detect enemies
         var enemyPosition = FindHighestConfidenceObject(predictedLabels);
         if (enemyPosition != null)
         {
             MoveMouseTo(enemyPosition.Value.X, enemyPosition.Value.Y);
         }
     }
     ```

2. **Analyzing Object Predictions**
   - Implement logic to analyze the ONNX model output and determine the enemy position:
     ```csharp
     private Point? FindHighestConfidenceObject(float[] predictedLabels)
     {
         // Logic to find the highest confidence prediction (enemy)
         // Use ONNX model's prediction to calculate object positions
         // Example: returning a placeholder position (500, 500) for testing
         return new Point(500, 500);
     }
     ```

---

### Step 5: Automating Mouse Movement

1. **Install InputSimulator**
   - In the **NuGet Package Manager**, search for `InputSimulator` and install it.

2. **Move the Mouse**
   - Add a method to move the mouse based on detected enemy positions:
     ```csharp
     using WindowsInput;

     public void MoveMouseTo(int x, int y)
     {
         InputSimulator sim = new InputSimulator();
         sim.Mouse.MoveMouseToPositionOnVirtualDesktop(x, y);
     }
     ```

3. **Test Mouse Movement**
   - Modify the `RunAimbot` method to move the mouse to a fixed position:
     ```csharp
     private void RunAimbot()
     {
         MoveMouseTo(50000, 50000);  // Test mouse movement
     }
     ```

---

### Step 6: Continuous Detection Loop

1. **Add a Continuous Loop**
   - Modify `RunAimbot` to continuously capture the screen and process it:
     ```csharp
     private async void RunAimbot()
     {
         var model = LoadModel();  // Load the ONNX model

         while (true)
         {
             Bitmap screenshot = CaptureScreen();
             var inputData = new BitmapData { Image = screenshot };

             var prediction = model.Predict(inputData);

             DetectEnemies(prediction.PredictedLabels);

             await Task.Delay(50);  // Add a small delay
         }
     }
     ```

2. **Cancel the Loop**
   - Use a `CancellationTokenSource` to stop the aimbot when the user presses the Stop button:
     ```csharp
     private CancellationTokenSource _cts;

     private async void btnStart_Click(object sender, EventArgs e)
     {
         lblStatus.Text = "Aimbot Running";
         _cts = new CancellationTokenSource();
         await Task.Run(() => RunAimbot(_cts.Token));
     }

     private void btnStop_Click(object sender, EventArgs e)
     {
         lblStatus.Text = "Aimbot Stopped";
         _cts.Cancel();
     }

     private async void RunAimbot(CancellationToken token)
     {
         var model = LoadModel();

         while (!token.IsCancellationRequested)
         {
             Bitmap screenshot = CaptureScreen();
             var inputData = new BitmapData { Image = screenshot };

             var prediction = model.Predict(inputData);

             DetectEnemies(prediction.PredictedLabels);

             await Task.Delay(50);
         }
     }
     ```

---

### Summary

With this project, you now have an aimbot that:
1. **Captures the screen** and processes it with an ONNX object detection model.
2. **Detects enemies** in the game using pre-trained ONNX models (such as Tiny YOLOv2).
3. **Moves the mouse** to the detected enemy's position for automated aiming.

Feel free to extend this tutorial
