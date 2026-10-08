# Bid fare recommender — local app (fuel-aware)

Enter trip type, car type, seats, pickup and dropoff; get a suggested driver bid
with a low/high range, a confidence score, and the fuel effect applied.

## Fresh setup (Windows, VS Code)

### 1. Export from Colab
Run the notebook to the end. The export cell writes two files:

* `bid_model_bundle.pkl`
* `requirements.txt` — exact library versions, **Python version on line 2**

It also prints e.g. `your local venv must use Python 3.12`.

### 2. Put the files together
```
bid_app/
├── app.py
├── model_service.py
├── bd_places.py
├── bid_model_bundle.pkl      <- from Colab (exact name)
├── requirements.txt          <- from Colab (replace the placeholder)
├── static/index.html
└── .vscode/launch.json
```

### 3. Create the venv with the SAME Python as Colab
```powershell
py -3.12 -m venv .venv                    # use the version the export printed
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python app.py
```
Open <http://127.0.0.1:8000>.

If `py -3.12` is not found, install that version from python.org (it sits
alongside other versions). If activation is blocked:
`Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

In VS Code: **Ctrl+Shift+P → Python: Select Interpreter → .venv**, then F5.

### 4. Check the startup lines
```
  bundle : bid_model_bundle.pkl
  built  : Python 3.12.x, scikit-learn 1.x.x
  fuel   : Tk 162.5/L (petrol_octane) in force since 2026-09-21
  one_way   : ...
```
If Python doesn't match, the app stops with the exact command to fix it.
This check exists because a mismatch otherwise loads fine and then crashes
mid-quote with `SystemError: no locals when deleting 'dict'`.

## Fuel price

The model predicts a fuel-neutral fare, then scales it by the pump price in
force on the booking date (Tk/L, average of octane and petrol). The fuel table
travels inside the bundle.

* **Prices changed?** Add a row to `FUEL_TABLE` in Stage B, re-run the export.
  No retraining needed if the session is still alive (just re-run Stage B's
  code cell, the predictor cells, and the export — or retrain fully later).
* **Before re-exporting**, type the new price into the app's *Fuel price*
  field. Blank = table price; a number = that price. Also useful for
  "what if fuel goes to Tk 180?".
* A warning appears when the price is above anything the model was built and
  tested on. Beyond that point the effect comes from the cost-share formula,
  not observed fares — compare with the first real bids at the new price.

## Updating the model later
Replace `bid_model_bundle.pkl` (and `requirements.txt` if Colab's versions
changed), stop the server with Ctrl+C, and start it again. No code changes.

## What the app refuses to do
* `hourly` / `airport_rental` trips: never in training → HTTP 400.
* Round trips without a return time → HTTP 400.
* Car types not seen in training → HTTP 400.
