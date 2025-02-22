from flask import Flask, render_template_string 
import getpass
from datetime import datetime
import subprocess
import pytz
import os

app = Flask(__name__)

def get_top_output():
    try:
        return subprocess.check_output(['top', '-b', '-n', 1]).decode('utf-8')
    except Exception as e:
        return f"Error: {str(e)}"

def get_full_name():
    try:
        
        username = getpass.getuser()
        with open('/etc/passwd', 'r') as passwd_file:
            for line in passwd_file:
                if line.startswith(username + ':'):
                    parts = line.split(':')
                    if len(parts) > 4:
                        full_name = parts[4].split(',')[0]  
                        return full_name
    except Exception:
        pass  

    return "Unknown User" 

@app.route('/htop')
def htop():
    template = """
    <html>
    <head>
        <title>System Status</title>
    </head>
    <body>
        <h1>System Information</h1>
        <p><b>Name:</b> {{ name }}</p>
        <p><b>Username:</b> {{ username }}</p>
        <p><b>Server Time (IST):</b> {{ server_time }}</p>
        <p><b>Top output:</b></p>
        <pre>{{ top_output }}</pre>
    </body>
    </html>
    """

    data = {
        'name': get_full_name(),
        'username': getpass.getuser(),
        'server_time': datetime.now(pytz.timezone('Asia/Kolkata')).strftime('%Y-%m-%d %H:%M:%S %Z'),
        'top_output': get_top_output()
    }
    return render_template_string(template, **data)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
