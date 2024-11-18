<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Photo Frame Editor</title>
    <style>
        canvas {
            border: 1px solid black;
        }
        #canvasContainer {
            position: relative;
            margin-top: 20px;
        }
        #downloadLink {
            display: none;
        }
    </style>
</head>
<body>
    <h1 style="text-align: center;">Interactive Photo Frame Editor</h1>
    <div style="text-align: center;">
        <input type="file" id="imageUploader" accept="image/*">
    </div>
    <div id="canvasContainer">
        <canvas id="canvas" width="1080" height="1080"></canvas>
    </div>
    <br>
    <a id="downloadLink" style="display:none;" download="framed_image.png">
        <button>Download Image</button>
    </a>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const frame = new Image();
        const imageUploader = document.getElementById('imageUploader');
        let uploadedImage;
        let imgX = 0, imgY = 0, imgWidth, imgHeight, scale = 1, cropping = false;
        const frameWidth = canvas.width, frameHeight = canvas.height;

        // Load frame image
        frame.src = 'https://i.postimg.cc/pXZjQZ70/Tanzimul-Ummah-Alumni-Association.png';

        frame.onload = function() {
            drawFrame();
            // Load saved image if available
            loadSavedImage();
        };

        // Draw frame on the canvas
        function drawFrame() {
            ctx.clearRect(0, 0, canvas.width, canvas.height); // Clear canvas
            ctx.drawImage(frame, 0, 0, frameWidth, frameHeight); // Draw the frame
            if (uploadedImage) {
                ctx.drawImage(uploadedImage, imgX, imgY, imgWidth * scale, imgHeight * scale);
            }
        }

        // Handle image upload
        imageUploader.addEventListener('change', (e) => {
            const reader = new FileReader();
            reader.onload = function (event) {
                const img = new Image();
                img.onload = function () {
                    uploadedImage = img;
                    imgWidth = img.width;
                    imgHeight = img.height;
                    scale = Math.min(frameWidth / imgWidth, frameHeight / imgHeight); // Fit the image inside the frame
                    imgX = (frameWidth - imgWidth * scale) / 2;
                    imgY = (frameHeight - imgHeight * scale) / 2;
                    drawFrame();
                    enableCanvasInteraction();
                };
                img.src = event.target.result;
            };
            reader.readAsDataURL(e.target.files[0]);
        });

        // Enable image editing (dragging and resizing)
        function enableCanvasInteraction() {
            canvas.addEventListener('mousedown', (e) => {
                if (e.offsetX >= imgX && e.offsetX <= imgX + imgWidth * scale &&
                    e.offsetY >= imgY && e.offsetY <= imgY + imgHeight * scale) {
                    cropping = true;
                }
            });

            canvas.addEventListener('mousemove', (e) => {
                if (cropping) {
                    imgX = e.offsetX - imgWidth * scale / 2;
                    imgY = e.offsetY - imgHeight * scale / 2;
                    drawFrame();
                    saveCanvasImage();
                }
            });

            canvas.addEventListener('mouseup', () => {
                cropping = false;
            });

            // Zoom in/out (resize the image)
            canvas.addEventListener('wheel', (e) => {
                e.preventDefault();
                if (e.deltaY < 0) {
                    scale *= 1.1;
                } else {
                    scale /= 1.1;
                }
                drawFrame();
                saveCanvasImage();
            });

            // Show download button
            document.getElementById('downloadLink').style.display = 'inline';
        }

        // Save canvas image to localStorage
        function saveCanvasImage() {
            const imageData = canvas.toDataURL('image/png');
            localStorage.setItem('savedImage', imageData);
        }

        // Load saved image from localStorage
        function loadSavedImage() {
            const savedImage = localStorage.getItem('savedImage');
            if (savedImage) {
                const img = new Image();
                img.onload = function () {
                    uploadedImage = img;
                    const imgElement = new Image();
                    imgElement.src = savedImage;
                    uploadedImage = imgElement;
                    imgWidth = uploadedImage.width;
                    imgHeight = uploadedImage.height;
                    scale = Math.min(frameWidth / imgWidth, frameHeight / imgHeight); // Fit the image inside the frame
                    imgX = (frameWidth - imgWidth * scale) / 2;
                    imgY = (frameHeight - imgHeight * scale) / 2;
                    drawFrame();
                };
                img.src = savedImage;
            }
        }

        // Download the final image when the button is clicked
        document.getElementById('downloadLink').addEventListener('click', function() {
            const croppedCanvas = document.createElement('canvas');
            const croppedCtx = croppedCanvas.getContext('2d');
            croppedCanvas.width = frameWidth;
            croppedCanvas.height = frameHeight;
            croppedCtx.drawImage(frame, 0, 0, frameWidth, frameHeight); // Draw frame
            croppedCtx.drawImage(uploadedImage, imgX, imgY, imgWidth * scale, imgHeight * scale); // Draw cropped image
            const imageUrl = croppedCanvas.toDataURL('image/png');
            this.href = imageUrl;
        });
    </script>
</body>
</html>
