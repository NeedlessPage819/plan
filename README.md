# C# Aimbot with GUI Tutorial

This repository contains a step-by-step tutorial for building an aimbot in C# with a graphical user interface (GUI). The project includes:
- **Screen capturing**
- **Image recognition** using **EmguCV** (OpenCV wrapper for .NET)
- **Mouse automation** using **InputSimulator**
- A **WinForms GUI** for controlling the aimbot

### Prerequisites

Before starting, ensure you have the following:
- **Visual Studio** (Community Edition is fine)
- **.NET Desktop Development** workload installed in Visual Studio
- Familiarity with basic C# programming and Visual Studio

### Step 1: Setting Up the Project

1. **Create a New C# Project**
   - Open Visual Studio.
   - Go to **File** > **New** > **Project**.
   - Select **Windows Forms App (.NET)** or **WPF Application (.NET)**.
   - Name your project `AimbotProject` and click **Create**.

2. **Design the GUI**
   - Open the toolbox on the left side and drag the following components onto the form:
     - A **Button** for starting the aimbot (`btnStart`), with the text "Start Aimbot".
     - A **Button** for stopping the aimbot (`btnStop`), with the text "Stop Aimbot".
     - A **Label** for displaying the aimbot status (`lblStatus`).

3. **Wire up the GUI**
   - Double-click the **Start Button** to create a click event handler and insert the following code:
     ```csharp
     private void btnStart_Click(object sender, EventArgs e)
     {
         lblStatus.Text = "Aimbot Running";
         RunAimbot();
     }
     ```
   - Do the same for the **Stop Button**:
     ```csharp
     private void btnStop_Click(object sender, EventArgs e)
     {
         lblStatus.Text = "Aimbot Stopped";
         StopAimbot();
     }
     ```

### Step 2: Screen Capture

1. **Capture the Screen**
   - Add a method to capture the screen as a `Bitmap`:
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

3. **Run the program** and ensure that the screenshot is saved.

### Step 3: Image Processing with EmguCV

1. **Install EmguCV**
   - Go to **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution**.
   - Search for `Emgu.CV` and install it.

2. **Basic Image Processing**
   - Convert the captured image to grayscale and detect edges:
     ```csharp
     using Emgu.CV;
     using Emgu.CV.Structure;

     private void RunAimbot()
     {
         Bitmap screenshot = CaptureScreen();
         Image<Bgr, byte> img = screenshot.ToImage<Bgr, byte>();

         // Convert to grayscale
         var grayImg = img.Convert<Gray, byte>();

         // Detect edges
         var edges = grayImg.Canny(100, 200);

         // Save the processed image
         edges.Bitmap.Save("edges.png");
     }
     ```

3. **Run the program** and check that an image with edges (`edges.png`) is saved.

### Step 4: Automating Mouse Movement

1. **Install InputSimulator**
   - In the **NuGet Package Manager**, search for `InputSimulator` and install it.

2. **Move the Mouse**
   - Add a method to move the mouse to a specific position:
     ```csharp
     using WindowsInput;

     public void MoveMouseTo(int x, int y)
     {
         InputSimulator sim = new InputSimulator();
         sim.Mouse.MoveMouseToPositionOnVirtualDesktop(x, y);
     }
     ```

3. **Test Mouse Movement**
   - Modify the `RunAimbot` method to move the mouse to an arbitrary position:
     ```csharp
     private void RunAimbot()
     {
         MoveMouseTo(50000, 50000);
     }
     ```

### Step 5: Continuous Detection Loop

1. **Add a Continuous Loop**
   - Modify `RunAimbot` to continuously capture the screen and process it:
     ```csharp
     private async void RunAimbot()
     {
         while (true)
         {
             Bitmap screenshot = CaptureScreen();
             DetectTargets(screenshot);
             await Task.Delay(50);  // Add a small delay
         }
     }
     ```

2. **Cancel the Loop**
   - Add a `CancellationTokenSource` to stop the aimbot:
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
         while (!token.IsCancellationRequested)
         {
             Bitmap screenshot = CaptureScreen();
             DetectTargets(screenshot);
             await Task.Delay(50);
         }
     }
     ```

### Step 6: Target Detection (Template Matching)

1. **Template Matching Example**
   - Add a method for detecting targets using template matching:
     ```csharp
     public void DetectTargets(Bitmap screenshot)
     {
         Image<Bgr, byte> img = screenshot.ToImage<Bgr, byte>();
         Image<Bgr, byte> template = new Image<Bgr, byte>("targetTemplate.png");

         using (var result = img.MatchTemplate(template, Emgu.CV.CvEnum.TemplateMatchingType.CcoeffNormed))
         {
             double minVal = 0.0, maxVal = 0.0;
             Point minLoc = new Point(), maxLoc = new Point();
             result.MinMax(out minVal, out maxVal, out minLoc, out maxLoc);

             if (maxVal > 0.8)  // Threshold for target detection
             {
                 MoveMouseTo(maxLoc.X, maxLoc.Y);
             }
         }
     }
     ```

2. **Run the aimbot** and verify that it moves the mouse when it detects a match.

### Summary

This project demonstrates how to:
1. Capture the screen using `System.Drawing`.
2. Process images using **EmguCV**.
3. Control the mouse using **InputSimulator**.
4. Create a simple GUI using **WinForms**.

Feel free to extend this tutorial by adding more advanced image recognition methods or optimizing the performance for real-time applications.
