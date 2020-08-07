from flask import Flask, render_template, jsonify, request
import socket
import re
import subprocess

app = Flask(__name__)


def run_command(command):
    p = subprocess.Popen(command,
                         stdout=subprocess.PIPE,
                         stderr=subprocess.STDOUT)
    return iter(p.stdout.readline, b'')


@app.route('/')
def home():
    return render_template('home.html')


@app.route('/port')
def port():
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        res = s.connect_ex((request.args.get('host'), int(request.args.get('port'))))
        if res == 0:
            d = {"success": 1, "up": 1}
            return jsonify(d)
        else:
            d = {"success": 1, "up": 0}
            return jsonify(d)

    except Exception as e:
        print(e)
        return jsonify({"success": 0, "up": 0})


@app.route('/svc')
def service():
    server = request.args.get('server')
    svc = request.args.get('service')
    res = run_command('sc \\\\{} query \"{}\"'.format(server, svc))
    for line in res:
        theline = re.findall("        STATE              : 4  RUNNING", line.decode())
        if theline:
            return jsonify({"success": 1, "up": 1})
            break
        else:
            running = 0
            continue

    return jsonify({"success": 0, "up": 0})

if __name__ == "__main__":
    app.run(debug=True)
