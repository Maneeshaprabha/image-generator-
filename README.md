# Image Generator Application

🚀 I developed an **Image Generator** application using Python and the Unsplash API. This project, created for a client, combines the power of various libraries to provide a seamless experience.

## Key Features:
🔍 **Category-Based Image Search**: Users can select a category like Food, Animals, People, Music, Art, Vehicle, or Random to fetch a related image from Unsplash.

💾 **Download Images**: Users can easily download the displayed images to their local directories.

⚡ **Responsive UI**: Built with `tkinter` and `ttkbootstrap`, the application offers a modern and responsive user interface.

## Installation

1. Clone the repository:
    ```bash
    git clone [https://github.com/yourusername/image-generator.git](https://github.com/Maneeshaprabha/image-generator-.git)
    ```
2. Navigate to the project directory:
    ```bash
    cd image-generator
    ```
3. Install the required dependencies:
    ```bash
    pip install -r requirements.txt
    ```

## Usage

1. Run the application:
    ```bash
    python app.py
    ```
2. Select a category from the dropdown menu.
3. Click on "Generate Image" to fetch a random image from Unsplash.
4. Click on "Download Image" to save the displayed image to your local directory.

## Code Overview

```python
import requests
import io
import tkinter as tk
from tkinter import ttk, filedialog, messagebox
from PIL import Image, ImageTk
from ttkbootstrap import Style
import os

root = tk.Tk()
root.title("Image Generator")
root.geometry("700x500")  
root.config(bg="white")
root.resizable(False, False)

app_style = Style(theme="sandstone")

photo = None
current_image_data = None

def display_image(category):
    global photo, current_image_data

    url = f"https://api.unsplash.com/photos/random?query={category}&orientation=landscape&client_id="
    
    try:
        response = requests.get(url)
        response.raise_for_status()
        current_image_data = response.json()
        img_url = current_image_data["urls"]["regular"]
        img_data = requests.get(img_url).content

        photo = ImageTk.PhotoImage(Image.open(io.BytesIO(img_data)).resize((600, 400), resample=Image.LANCZOS))
        label.config(image=photo)
        label.image = photo
        download_button.config(state="normal")
    except requests.exceptions.RequestException as e:
        messagebox.showerror("Error", f"Failed to retrieve image: {e}")

def download_image():
    if current_image_data:
        photo_id = current_image_data["id"]
        track_url = f"https://api.unsplash.com/photos/{photo_id}/download?client_id="
        try:
            track_response = requests.get(track_url)
            track_response.raise_for_status()

            directory = filedialog.askdirectory()
            if directory:
                img_url = current_image_data["urls"]["full"]
                filename = f"{photo_id}.jpg"
                filepath = os.path.join(directory, filename)

                with open(filepath, "wb") as f:
                    f.write(requests.get(img_url).content)
                messagebox.showinfo("Download Complete", f"Image saved as {filename} in {directory}")
        except requests.exceptions.RequestException as e:
            messagebox.showerror("Error", f"Failed to track download: {e}")
    else:
        messagebox.showerror("Error", "No image to download. Generate an image first.")

def enable_button(*args):
    generate_button.config(state="normal" if category_var.get() != "Choose Category" else "disabled")

def create_gui():
    global category_var, generate_button, label, download_button

    category_var = tk.StringVar(value="Choose Category")
    category_options = ["Choose Category", "Food", "Animals", "People", "Music", "Art", "Vehicle", "Nature", "Random"]
    category_dropdown = ttk.OptionMenu(root, category_var, *category_options, command=enable_button)
    category_dropdown.grid(row=0, column=0, padx=10, pady=10, sticky="nsew")
    category_dropdown.config(width=14)

    generate_button = ttk.Button(root, text="Generate Image", state="disabled", command=lambda: display_image(category_var.get()))
    generate_button.grid(row=0, column=1, padx=10, pady=10, sticky="nsew")

    download_button = ttk.Button(root, text="Download Image", state="disabled", command=download_image)
    download_button.grid(row=0, column=2, padx=10, pady=10, sticky="nsew")

    label = tk.Label(root, background="white")
    label.grid(row=1, column=0, columnspan=3, padx=10, pady=10, sticky="nsew")

    root.columnconfigure([0, 1, 2], weight=1)
    root.rowconfigure(1, weight=1)

if __name__ == '__main__':
    create_gui()
    root.mainloop()
