# Blockchain-Based-image-uploding-web-application
from flask import Flask, render_template, request, redirect, url_for, send_from_directory
import os
import hashlib
import datetime

from PIL import Image

app = Flask(__name__)
UPLOAD_FOLDER = './uploads'
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
blockchain = []


def allowed_file(filename):
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS


@app.route('/', methods=['GET'])
def home():
    return render_template('index.html', blockchain=blockchain)


@app.route('/upload', methods=['POST'])
def upload_image():
    file = request.files['file']
    if file and allowed_file(file.filename):
        filename = file.filename
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
        image_hash = calculate_hash(filename)
        add_block(image_hash, filename)
    return redirect(url_for('home'))


@app.route('/image/<string:image_hash>', methods=['GET'])
def get_image(image_hash):
    for block in blockchain:
        if block['hash'] == image_hash:
            return send_from_directory(app.config['UPLOAD_FOLDER'], block['filename'])
    return "Image not found."


def calculate_hash(filename):
    hash_object = hashlib.sha256()
    with open(os.path.join(app.config['UPLOAD_FOLDER'], filename), "rb") as f:
        for chunk in iter(lambda: f.read(4096), b""):
            hash_object.update(chunk)
    return hash_object.hexdigest()


def add_block(image_hash, filename):
    timestamp = str(datetime.datetime.now())
    previous_hash = "0" if len(blockchain) == 0 else blockchain[-1]['hash']
    block = {
        'index': len(blockchain) + 1,
        'timestamp': timestamp,
        'data': filename,
        'previous_hash': previous_hash,
        'hash': image_hash,
        'filename': filename
    }
    blockchain.append(block)


if __name__ == '__main__':
    app.run(debug=True)
