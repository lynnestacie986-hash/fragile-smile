from flask import Flask, render_template, request, redirect, url_for
from datetime import datetime
import os

app = Flask(__name__)
JOURNAL_FILE = "my_journal.txt"

def load_entries():
    if not os.path.exists(JOURNAL_FILE):
        return []
    
    with open(JOURNAL_FILE, "r", encoding="utf-8") as file:
        content = file.read()
    
    # Split entries by our divider line
    raw_entries = content.split("-" * 40 + "\n")
    entries = []
    
    for entry in raw_entries:
        if entry.strip():
            lines = entry.strip().split("\n")
            # Extract date if it exists
            date = lines[0].replace("Date: ", "") if lines[0].startswith("Date: ") else "Unknown Date"
            # Get the rest of the text
            text = "\n".join(lines[1:]) if len(lines) > 1 else lines[0]
            entries.append({"date": date, "text": text})
            
    return entries[::-1] # Show newest entries first

@app.route('/')
def index():
    entries = load_entries()
    return render_template('index.html', entries=entries)

@app.route('/add', methods=['POST'])
def add_entry():
    entry_text = request.form.get('content')
    if entry_text and entry_text.strip():
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M")
        with open(JOURNAL_FILE, "a", encoding="utf-8") as file:
            file.write(f"Date: {timestamp}\n")
            file.write(f"{entry_text.strip()}\n")
            file.write("-" * 40 + "\n")
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)
