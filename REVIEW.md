2. Same predictions as the notebook

The predictions of the web application were compared with the expected values in check_expected.json using check_image.jpg.

The self-test was run on the GitHub Pages website using `?selftest=1`. In `index.html` (lines 370–395), the test loads the image and expected values, runs the ONNX model and compares the probabilities.

The web predicted apple with a probability of 0.0001378767, cherry with 0.9997303, chestnut with 0.0001136179 and maple with 0.0000181881. These values were almost identical to those in `check_expected.json`.

The test passed with a maximum absolute difference of 2.826e-8, so all predictions agree to three decimal places.

The normal web page shows probabilities rounded to one decimal place as percentages, so I used the self-test to check the exact values.

No problems were found and no changes were needed.

3. Model preprocessing

The preprocessing in `index.html` (lines 214–235) was checked and compared with `model_info.json` and the original notebook.

In `index.html` (lines 221–225), the image is first cropped to the central square and resized to 96 × 96 pixels using `drawImage()`.

The RGB values are then divided by 255 and normalized using the mean and standard deviation from `model_info.json`. The mean values are [0.485, 0.456, 0.406] and the standard deviations are [0.229, 0.224, 0.225].

These values are the same as in the notebook, where `transforms.ToTensor()` and `transforms.Normalize(mean, std)` are used.

The channel order was also checked in `index.html` (lines 227–234). The red channel is stored first, followed by green and blue. The final tensor has the shape `(1, 3, 96, 96)`, as required by the model.

One difference is that the notebook does not explicitly crop or resize the images because they are already 96 × 96 pixels. The web page needs these extra steps because images from the phone camera can have different sizes.

The preprocessing matches the model requirements, and no changes were needed.

### 4. Nothing leaves the phone

The network communication was checked in `index.html` and using the Network tab in Firefox Developer Tools.

In `index.html` (lines 331–353), uploaded photos are loaded locally using `createImageBitmap()`. The classification is performed directly in the browser using `session.run()` (lines 246–252). Live camera frames are also processed locally (lines 288–291).

The Network tab showed that the page loads `model_info.json`, `leaf_model.onnx`, the ONNX Runtime JavaScript and WebAssembly files, and the test files `check_image.jpg` and `check_expected.json`. These files were loaded from GitHub Pages. The QR code is generated locally using `qrcode-generator.js`.

After uploading another photo, the predictions changed, but no new network requests appeared. No image upload requests were found.

One HTTP 404 error was shown for `favicon.ico`, but this is only the page icon and does not affect the classifier.

No issues with image privacy were found in the checked code and network activity. The photos are processed locally in the browser.