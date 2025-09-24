import requests
from flask import Flask, request, render_template_string

app = Flask(__name__)
API_KEY = "your_openweathermap_api_key"  # replace with your own API key

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Weather Dashboard</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body { background: linear-gradient(to right, #4facfe, #00f2fe); color: #fff; min-height: 100vh; }
        .container { max-width: 600px; margin-top: 80px; }
        .card { border-radius: 15px; overflow: hidden; }
        input { border-radius: 10px; }
        button { border-radius: 10px; }
    </style>
</head>
<body>
    <div class="container">
        <div class="card shadow">
            <div class="card-body text-center bg-dark">
                <h2 class="mb-4">🌦 Weather Dashboard</h2>
                <form method="post" class="d-flex mb-4">
                    <input type="text" class="form-control me-2" name="city" placeholder="Enter city" required>
                    <button type="submit" class="btn btn-primary">Search</button>
                </form>
                {% if weather %}
                    {% if weather.city != "Not Found" %}
                        <h3>{{ weather.city }}</h3>
                        <h1>{{ weather.temp }}°C</h1>
                        <p class="lead">{{ weather.description }}</p>
                        <p><strong>Humidity:</strong> {{ weather.humidity }}%</p>
                        <p><strong>Wind:</strong> {{ weather.wind }} m/s</p>
                    {% else %}
                        <div class="alert alert-danger">City not found!</div>
                    {% endif %}
                {% endif %}
            </div>
        </div>
    </div>
</body>
</html>
"""

@app.route("/", methods=["GET", "POST"])
def home():
    weather = None
    if request.method == "POST":
        city = request.form.get("city")
        url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric"
        try:
            response = requests.get(url, timeout=5)
            data = response.json()
            if data.get("cod") == 200:
                weather = {
                    "city": data["name"],
                    "temp": data["main"]["temp"],
                    "description": data["weather"][0]["description"].title(),
                    "humidity": data["main"]["humidity"],
                    "wind": data["wind"]["speed"]
                }
            else:
                weather = {"city": "Not Found"}
        except Exception:
            weather = {"city": "Not Found"}
    return render_template_string(HTML_TEMPLATE, weather=weather)

if __name__ == "__main__":
    app.run(debug=True)
