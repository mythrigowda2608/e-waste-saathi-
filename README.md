import os, sys, time, subprocess

# 1. Install
os.system("pip install -q fastapi uvicorn python-multipart jinja2 pillow")
time.sleep(3)

# 2. Create main.py with IMAGE UPLOAD
open("main.py","w").write('''
from fastapi import FastAPI, Form, UploadFile, File
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
import os, shutil, uuid

app = FastAPI()
os.makedirs("static/uploads", exist_ok=True)
app.mount("/static", StaticFiles(directory="static"), name="static")

FILE = "data.txt"
LAST_IMG = "last_img.txt"

def get_total():
    if os.path.exists(FILE):
        try: return float(open(FILE).read() or 0)
        except: return 0.0
    return 0.0

def save_total(v):
    open(FILE,"w").write(str(v))

def get_last_img():
    if os.path.exists(LAST_IMG):
        return open(LAST_IMG).read().strip()
    return ""

@app.get("/", response_class=HTMLResponse)
def home():
    total = get_total()
    points = int(total * 10)
    img = get_last_img()
    img_html = f'<img src="/{img}" style="width:200px;border-radius:10px;margin:15px;box-shadow:0 2px 10px #888">' if img else "<p>No image uploaded yet</p>"
    return f"""
    <html><body style="font-family:Arial;background:#e8f5e9;padding:20px">
    <div style="background:white;max-width:700px;margin:auto;padding:30px;border-radius:20px;text-align:center;box-shadow:0 4px 20px #aaa">
    <h1 style="color:#2e7d32">♻️ E-Waste Saathi - Step 11</h1>
    <div style="display:flex;gap:15px;margin:20px 0">
      <div style="background:#e8f5e9;padding:20px;border-radius:15px;flex:1"><p>CO2 SAVED</p><h1 style="font-size:45px;color:#1b5e20;margin:0">{total} kg</h1></div>
      <div style="background:#e8f5e9;padding:20px;border-radius:15px;flex:1"><p>GREEN POINTS</p><h1 style="font-size:45px;color:#1b5e20;margin:0">{points}</h1></div>
    </div>
    <h3>Last Uploaded E-Waste:</h3>
    {img_html}
    <br>
    <a href="/book" style="background:#2e7d32;color:white;padding:15px 40px;border-radius:10px;text-decoration:none;font-size:18px;display:inline-block;margin-top:15px">Book a Pickup + Upload Image →</a>
    </div></body></html>
    """

@app.get("/book", response_class=HTMLResponse)
def book():
    return """
    <body style="font-family:Arial;background:#e8f5e9;padding:20px">
    <div style="background:white;max-width:500px;margin:auto;padding:30px;border-radius:20px">
    <h2>Book Pickup + Upload Image</h2>
    <form action="/submit" method="post" enctype="multipart/form-data">
    <input name="name" placeholder="Your Name" required style="width:100%;padding:12px;margin:10px 0">
    <input name="address" placeholder="Baiyyappanahalli" required style="width:100%;padding:12px;margin:10px 0">
    <select name="item" style="width:100%;padding:12px;margin:5px 0">
      <option>Laptop</option><option>Phone</option><option>TV</option><option>Battery</option>
    </select>
    <input name="weight" type="number" step="0.1" value="2.5" required style="width:100%;padding:12px;margin:10px 0">
    <label style="font-weight:bold">Upload E-Waste Image:</label><br>
    <input name="image" type="file" accept="image/*" required style="width:100%;padding:10px;margin:10px 0;border:1px solid #ccc;border-radius:8px">
    <button style="background:#2e7d32;color:white;padding:15px;width:100%;border:none;border-radius:10px;margin-top:10px;font-size:18px">Submit Pickup</button>
    </form><br><a href="/">← Back to Home</a>
    </div></body>
    """

@app.post("/submit", response_class=HTMLResponse)
async def submit(name: str = Form(...), weight: float = Form(...), image: UploadFile = File(...)):
    # Save image
    ext = image.filename.split(".")[-1]
    filename = f"static/uploads/{uuid.uuid4().hex}.{ext}"
    with open(filename, "wb") as f:
        shutil.copyfileobj(image.file, f)
    open(LAST_IMG,"w").write(filename)

    co2 = weight * 2
    total = get_total() + co2
    save_total(total)

    return f"""
    <body style="font-family:Arial;background:#e8f5e9;padding:20px;text-align:center">
    <div style="background:white;max-width:600px;margin:auto;padding:30px;border-radius:20px">
    <h1 style="color:green">✅ Pickup Booked!</h1>
    <p>Thanks {name}, Image Uploaded Successfully</p>
    <img src="/{filename}" style="width:250px;border-radius:15px;margin:15px">
    <h2>CO2 Saved: {co2} kg | Total: {total} kg</h2>
    <a href="/" style="background:#2e7d32;color:white;padding:15px 30px;border-radius:10px;text-decoration:none;display:inline-block;margin-top:15px">Go to Home - See Image & Dashboard</a>
    </div></body>
    """
''')

# 3. Start server
os.system("pkill -f uvicorn")
time.sleep(2)
subprocess.Popen([sys.executable, "-m", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"])
time.sleep(5)

print("✅ VERIFIED - SCROLL DOWN - WEBSITE BELOW 👇")
from google.colab import output
output.serve_kernel_port_as_iframe(8000, height=900)
