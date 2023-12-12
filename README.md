# Introduction 
The goal of this project was to showcase my ability to identify, formulate, and solve image processing problems while performing a real-time demonstration of an image processing example. From this project, I have developed a simple graphic user interface (GUI) that can load an image, apply an image processing technique, and display the input/output result. Specifically, I have designed a combination of image processing techniques to have my application take any image and output it as an oil style drawing. 

# Objective 
The objective of this project is to derive an “oil painting” like output given an input image. Specifically, the output image should have a unique texture that appears a bit rough and thick to mimic that of a dynamic and impressionistic effect. In my project, I tried to emphasize the effects by focusing on brush size strokes, intensity, and quantization level. 

# How to run code
1. Save any photo(s) and esure the image file format is .PNG, .JEPG, or BMP
![graph](Images/ReadMe/SetUp_1.png)
2. Press “Upload Image” button
![graph](Images/ReadMe/SetUp_2.png)
3. A file directory will show up indicating which file you would like to upload, press "open" when you have found the image to upload
![graph](Images/ReadMe/SetUp_3.png)
4. If the file you have uploaded is not the desired file, press “Clear Image” and repeat steps 2-3
![graph](Images/ReadMe/SetUp_4.png)
5. Once the image is uploaded, use the scroll bars “Brush Size”, “Intensity”, and “Quant Levels” to manipulate the three categories
![graph](Images/ReadMe/SetUp_5.png)
6. After deciding values from the scroll bars, click “Apply Oil Painting” and watch the magic happen
![graph](Images/ReadMe/SetUp_6.png)
7. If you decide to upload a new file, restart and complete step 2-6

# Implementation Details - GUI
The function named get_file_path utilizes a file dialog to prompt the user to select a file and returns the selected file's path. 
```
def get_file_path():
    file_path = filedialog.askopenfilename(filetypes=[])
    return file_path
```

The function named upload_image that uses the get_file_path function to obtain a file path from the user. If a valid file path is obtained, it opens the image file, resizes it to 500x500 pixels, and displays it in a graphical user interface. The image was intentionally constrained to a 500x500 pixel resolution to optimize computational efficiency during image processing, resulting in quicker outputs. 
```
def upload_image():
    file_path = get_file_path()

    if file_path:
        original_image = Image.open(file_path)
        original_image = original_image.resize((500, 500))
        original_photo = ImageTk.PhotoImage(original_image)

        original_image_label.config(image=original_photo)
        original_image_label.image = original_photo
        original_image_label.file_path = file_path
        original_image_label.place(x=50, y=50)

        altered_image_label.config(image=None)
        altered_image_label.image = None
```

The clear_image function resets the display of both original_image_label and altered_image_label in a graphical user interface.
```
def clear_image():
    original_image_label.config(image=None)
    original_image_label.image = None
    original_image_label.file_path = None

    altered_image_label.config(image=None)
    altered_image_label.image = None

```

The code below disables the resizability while setting the window size to 1300 x 600. The goal of creating a restrictive limit was to optimize computational efficiency during image processing, resulting in quicker outputs. 
```
    window_width = 1300
    window_height = 600
    root.geometry(f"{window_width}x{window_height}")
    root.resizable(False, False)

```

The code below labels for displaying original and altered images, buttons for uploading and clearing images, sliders for adjusting brush size, intensity, and quantization levels, and a button for applying an oil painting effect. The layout is organized with appropriate padding to enhance the visual presentation in the root window.
```
    original_image_label = tk.Label(root)
    original_image_label.pack(pady=10)

    altered_image_label = tk.Label(root)
    altered_image_label.pack(pady=10)

    button_width = 15 
    upload_button = tk.Button(root, text="Upload Image", command=upload_image, width=button_width)
    upload_button.pack(pady=10)

    clear_button = tk.Button(root, text="Clear Image", command=clear_image, width=button_width)
    clear_button.pack(pady=10)

    brush_size_var = tk.IntVar()
    brush_size_slider = tk.Scale(root, from_=0, to=100, orient="horizontal", variable=brush_size_var, label="Brush Size")
    brush_size_slider.pack(pady=10)

    intensity_var = tk.IntVar()
    intensity_slider = tk.Scale(root, from_=0, to=100, orient="horizontal", variable=intensity_var, label="Intensity")
    intensity_slider.pack(pady=10)

    quantization_levels_var = tk.IntVar()
    quantization_levels_slider = tk.Scale(root, from_=0, to=100, orient="horizontal", variable=quantization_levels_var, label="Quant Levels")
    quantization_levels_slider.pack(pady=10)

    apply_button = tk.Button(root, text="Apply Oil Painting", command=apply_oil_painting, width=button_width)
    apply_button.pack(pady=10)
```

# Implementation Details - Image Processing 
The main function used to preform image processing is function oil_painting_effect(image_path, brush_size, intensity, quantization_levels)


The code below ensures that the brush size used for the median blur operation is an odd number. This is important because the median blur operation typically works best with an odd-sized kernel. The median filter operates by taking the median value of the pixel intensities within a neighborhood defined by the kernel size. When the kernel size is odd, there is a well-defined center pixel in the neighborhood. However, when the kernel size is even, there is no exact center, and it might not behave as expected.
```
    if brush_size % 2 == 0:
        brush_size += 1
```

Applying a median filter helps to reduce noise in the image by replacing each pixel value with the median value in its neighborhood. This is particularly useful for images with salt-and-pepper noise, as it helps in preserving edge information while smoothing out unwanted variations.
```
median_filtered = cv2.medianBlur(original_image, brush_size)
```
```
def median_filter(image, kernel_size):
    pad_size = kernel_size // 2
    padded_image = cv2.copyMakeBorder(image, pad_size, pad_size, pad_size, pad_size, cv2.BORDER_REPLICATE)
    result = np.zeros_like(image)

    for i in range(image.shape[0]):
        for j in range(image.shape[1]):
            window = padded_image[i:i+kernel_size, j:j+kernel_size].flatten()
            result[i, j] = np.median(window)

    return result
```

Converting the image to grayscale simplifies the processing by reducing the image to a single channel representing intensity. This is often beneficial when applying subsequent operations that don't rely on color information, making the algorithm more efficient and avoiding unnecessary complexity.
```
gray_image = cv2.cvtColor(median_filtered, cv2.COLOR_BGR2GRAY)
```
```
def rgb_to_grayscale(image):
    if len(image.shape) == 3 and image.shape[2] == 3:
        grayscale_image = 0.299 * image[:, :, 0] + 0.587 * image[:, :, 1] + 0.114 * image[:, :, 2]

        # NOTE: convert to 8 bit
        grayscale_image = grayscale_image.astype(np.uint8)

        return grayscale_image
    else:
        raise ValueError("Input image should be in RGB format (3 channels).")
```

Color quantization is employed to reduce the number of distinct colors in the image. This not only reduces memory requirements but also serves to simplify subsequent analysis.
```
    quantized_image = cv2.cvtColor(median_filtered, cv2.COLOR_BGR2RGB)
    Z = quantized_image.reshape((-1, 3))
```

The criteria for the k-means clustering algorithm, such as the number of clusters (k), are set to control the granularity of color quantization. The image is then converted back to uint8 format to ensure compatibility with subsequent processing steps. This step essentially defines how many dominant colors or clusters the final image will have.
```
def kmeans(X, k, max_iters=100, tol=1e-4):
    centroids = X[np.random.choice(len(X), k, replace=False)]

    for _ in range(max_iters):
        labels = np.argmin(np.linalg.norm(X[:, np.newaxis] - centroids, axis=2), axis=1)
        new_centroids = np.array([X[labels == i].mean(axis=0) for i in range(k)])
        
        if np.linalg.norm(new_centroids - centroids) < tol:
            break

        centroids = new_centroids
    return labels, centroids
```

Reconstructing the image after color quantization involves assigning each pixel to its representative color cluster. This step is crucial for visually preserving the structure of the original image while introducing the stylized effect created by the color quantization.
```
    quantized_image = centers[labels.flatten()]
    quantized_image = quantized_image.reshape(original_image.shape)
```

Applying intensity to the quantized image involves enhancing or modifying the intensity values of the pixels. This step is  included to further emphasize certain features, textures, or contours in the image, enhancing the overall oil-painting-like effect and making the final output visually appealing.
```
    intensity_matrix = np.ones(original_image.shape, dtype="uint8") * intensity
    oil_painting = cv2.add(quantized_image, intensity_matrix)    
```

# Output Images
![graph](Images/Output/cat1.png)
![graph](Images/Output/cat2.png)
![graph](Images/Output/cat3.png)
![graph](Images/Output/cat4.png)
![graph](Images/Output/cat5.png)

![graph](Images/Output/seattle1.png)
![graph](Images/Output/seattle2.png)
![graph](Images/Output/seattle3.png)
![graph](Images/Output/seattle4.png)
![graph](Images/Output/seattle5.png)
